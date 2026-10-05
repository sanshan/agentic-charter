# Delivery and synchronization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Tech Stack:** Node.js 22/24, TypeScript strict + ESM, pnpm, node:test, JSON schemas and Markdown. Exact dependencies are locked in #3.

**Spec:** [Approved specification](../specs/2026-10-05-agentic-charter-design.md), originally approved at commit `8721c98357d08e66f03bd6c5df6a872b0b3ca7ad` by the owner on 2026-10-05.

## Global Constraints

- One package: `@sanshan/agentic-charter`; bin: `agentic-charter`; commands: `init`, `sync`, `check`.
- CLI Node.js major 22 and 24; schema_version: 1; UTF-8, LF; no generation timestamps.
- No Nx in this single-package repo; preserve consumer Nx-managed blocks.
- Ordinary tests: no network, LLM, database or credentials; no hidden install-hook.
- Package/lockfile own version; generated manifest is provenance, not an editable version pin.
- Any sync conflict means zero target-file writes; no destructive force option.
- Base-branch policy governs its own weakening PR; exceptions require ID, scope and reason.
- Codex GitHub is a PR-review path, not a task runner; no parser-derived machine clean.
- Manual curator, owner acceptance of every catalog PR, no automerge.
- Task branches originate from `epic/1-agentic-charter` and PRs target it.
- Before each implementation reread epic #1 and all subtasks #2–#17.
- CI for exact head precedes independent review; relevant new head invalidates previous review.
- Plan status: proposed for owner review; implementation waits for plan approval and method selection.
- [Index](2026-10-05-agentic-charter.md) owns shared commands and acceptance procedure.


**Goal:** Implement #11–#12: standalone CLI delivery, zero-write conflict handling and trustworthy drift checks.

**Architecture:** A pure renderer produces desired bytes. A filesystem planner compares previous/current/desired state before a journaled writer changes owned files.

## Review Focus

1. Two CLI processes race — #12 must lock and recheck bytes before writing.
2. Unmanaged content shares an AGENTS file — #11 preserves exact surrounding bytes, including CRLF.
3. Manifest was edited to hide drift — #12 regenerates desired bytes independently.
4. Interrupted multi-file write — #12 blocks new work and recovers from durable original bytes.
5. Profile removal encounters a modified obsolete file — #12 aborts all writes and preserves the file.

## File Structure and Interfaces

- `src/generation/types.ts`: OwnedOutput = {path, kind: 'file'|'block', bytes: Uint8Array, block_id?: string}; DesiredOutput = {outputs, manifest}; GeneratedManifest follows spec §5.
- `src/generation/render.ts`: `renderBundle(catalog: ResolvedCatalog, config: ConsumerConfig, identity: PackageIdentity): DesiredOutput`; PackageIdentity = {name, version, generator_schema_version}.
- `src/generation/agents-block.ts`: `mergeAgentsBlock(current: Uint8Array | null, replacement: Uint8Array): Uint8Array`.
- `src/config/lockfile.ts`: `readLockedPackage(root: string, packageName: string): Promise<LockedPackage>`; LockedPackage = {manager: 'pnpm'|'npm', importer, name, version, integrity?: string}.
- `src/sync/types.ts`: Snapshot = {files: ReadonlyMap<string, Uint8Array>, manifest: GeneratedManifest | null}; Plan = {changes, conflicts, baselineDigests}; Change = {path, before: Uint8Array|null, after: Uint8Array|null}; Conflict = {path, reason}.
- `src/sync/plan.ts`: `planSync(snapshot: Snapshot, desired: DesiredOutput): Plan`.
- `src/sync/snapshot.ts`: `readSnapshot(root: string, desiredPaths: readonly string[]): Promise<Snapshot>`.
- `src/sync/apply.ts`: `applyPlan(root: string, plan: Plan): Promise<ApplyResult>`; result = {changedPaths}; conflict or IO/recovery throws typed CharterError.
- `src/sync/check.ts`: `checkBundle(root: string, desired: DesiredOutput): Promise<CheckReport>`; CheckReport = {ok, diagnostics}.
- `src/cli/{init,sync,check}.ts`: command handlers; `src/cli/main.ts` handles args/errors only.

CharterError = {code: 'config'|'conflict'|'io', diagnostics}; CLI maps config/schema/compatibility to 2, conflict to 3, IO/recovery to 4, check drift to 1. Expected errors are typed; no stack leak by default.

### Task 9 — #11: pure bundle and init

**Files:** create generation/config/CLI modules above as needed, `templates/agents.md`, `templates/setup.md`, `tests/generation.test.ts`, `tests/init.test.ts`, `tests/lockfile.test.ts`, `tests/support/consumer.ts`; modify bin and smoke test.

**Consumes:** Part 1 Catalog/ResolvedCatalog/ConsumerConfig and validation APIs.
**Produces:** renderBundle, mergeAgentsBlock, readLockedPackage, initial planSync/applyPlan used by init. Import their shared types from src/sync/types.ts even before full sync exists. #12 extends the same transaction machinery; do not create a second writer.

- [ ] Define fixture helper `createConsumer(files: Record<string,string>): Promise<ConsumerFixture>` with root/readAll/dispose, plus `invokeCli(root,args)` returning status/stdout/stderr. Fixtures contain package/lockfile metadata and locally installed packed Charter.
- [ ] Write rendering/init tests: repeated rendering has identical bytes; non-interactive missing profiles returns 2; unknown schema returns 2; init twice is idempotent. AGENTS test asserts unmanaged prefix/suffix bytes remain identical, including Nx block and CRLF.
- [ ] Add ownership collision test: conflicting workflow path plus absent config leaves ALL files unchanged, including no new config. Add invalid/duplicated marker tests.
- [ ] Run focused tests; expect missing handlers/rendering failures.
- [ ] Implement generation of selected rules/sources/effective policy/manifest and routing. Hash actual outputs excluding self-referential manifest hash; serialize manifest deterministically. Preserve local paths and rewrite only generated source references.
- [ ] Implement readLockedPackage for pnpm/npm importer scope, installed package version consistency and ambiguity rejection; unsupported manager yields diagnostic without replacing lockfile.
- [ ] Implement CLI --profiles, --non-interactive and --dry-run; TTY only for prompts. Add profile selection to newly created config; existing config remains consumer-owned. Dry-run returns same preflight errors but never writes.
- [ ] Implement initial preflight and durable writer needed by init with tests before use; #12 adds update-specific cases. Never ship init with partial collision writes.
- [ ] Update package smoke: pending-command assertion from #3 becomes a successful packaged init test. Run `pnpm verify`, then delete fixture node_modules and verify generated sources remain readable.
- [ ] Commit `feat: initialize consumer standards without overwriting local content`.

At #11, rendering accepts owned template outputs, but actual workflow content remains #13. Setup instructions explicitly mark that integration as pending until #13, rather than pretending review works.

### Task 10 — #12: sync planning, check and recovery

**Files:** create/extend sync modules above, `src/sync/{journal,lock,recover}.ts`, `tests/sync-plan.test.ts`, `tests/check.test.ts`, `tests/sync-recovery.test.ts`, `tests/sync-concurrency.test.ts`, `docs/consumer/update-and-recovery.md`.

**Interfaces:** existing Plan/DesiredOutput; `recoverTransaction(root: string): Promise<RecoveryReport>` for internal startup recovery; RecoveryReport = {state: 'none'|'recovered'|'manual-required', paths}. No extra public CLI command required.

- [ ] Write table-driven plan tests for current=prior, current=desired, missing prior, missing owned output, modified obsolete output, unmanaged collision and profile add/remove. Every conflict asserts `plan.conflicts.length > 0` and application performs zero target writes.
- [ ] Write forged-manifest test: modify owned rule and its recorded digest, then assert `(await checkBundle(root, desired)).ok === false`. Mutating desired package/config is a policy change, not proof of trust.
- [ ] Run focused tests and confirm update/check assertions fail.
- [ ] Implement three-way byte comparison, owned block extraction, sorted deterministic diff and read-only check. Corrupt/unsupported prior manifest prevents writes; first install never adopts arbitrary existing files.
- [ ] Add failure-injection tests for every rename/write boundary and process termination after first write. Assert original target bytes restored or durable journal remains with explicit manual-required state.
- [ ] Add two-process lock test and TOCTOU test; byte change after planning aborts without overwriting concurrent changes. Recovery never clobbers a newer user edit; such a path becomes manual-required.
- [ ] Run recovery/concurrency tests to observe failure before implementation.
- [ ] Implement exclusive lock, durable journal/backups, staging and original-byte recheck. Cleanly rollback ordinary IO failures. On next invocation, unresolved journal blocks mutation until safe recovery; report affected paths and exact recovery instructions.
- [ ] Add upgrade/compatible-downgrade/unsupported-schema tests and whole-Git-snapshot rollback walkthrough. Run `pnpm verify`; verify no stale transaction artifacts are packed.
- [ ] Commit `feat: synchronize owned outputs with drift and recovery checks`.

## Completion

init/sync/check operate from packed CLI outside source tree. No target-file writes on conflict/dry-run/check; local files survive updates; documented failure recovery distinguishes normal error from interrupted multi-file transaction.
