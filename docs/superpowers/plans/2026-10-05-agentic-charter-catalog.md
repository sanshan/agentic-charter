# Catalog and standards Implementation Plan

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


**Goal:** Implement #3–#10: a tested package foundation and a traceable engineering catalog.

**Architecture:** Schemas and profile selection are deterministic. Sources own requirements; rules own decision boundaries and require separate calibration.

## Review Focus

1. Reused retired ID — #4 rejects active/retired collisions.
2. Source symlink escapes the repo — #4 rejects realpath escape.
3. Monorepo packages use different framework versions — #7 verifies each target.
4. Next.js project uses JavaScript without TypeScript — #9 does not require TypeScript.
5. Inherited calibration is mistaken for new evidence — #6 keeps migrated rules advisory.

## File Structure and Interfaces

- `src/catalog/types.ts`: Rule, Profile, Catalog, Scope, Diagnostic, ValidationResult, ResolvedCatalog.
- `src/config/types.ts`: ConsumerConfig, Target, Exception, ResolvedVersion.
- `schemas/{rule,profile,config,manifest,result,evidence}.schema.json`: schema_version 1, closed objects and enums from spec §5.
- `src/catalog/validate.ts`: `validateCatalog(root: string): Promise<ValidationResult>`.
- `src/catalog/load.ts`: `loadCatalog(root: string): Promise<Catalog>`.
- `src/catalog/resolve.ts`: `resolveProfiles(catalog: Catalog, ids: readonly string[]): ResolvedCatalog`.
- `src/catalog/paths.ts`: `assertSafePath(root: string, path: string): Promise<string>`.
- `src/catalog/digest.ts`: `semanticDigest(rule: Rule, sourceBytes: Uint8Array): string`.
- `src/config/validate.ts`: `validateConfig(input: unknown, catalog: Catalog): ConsumerConfig`.
- `src/config/compatibility.ts`: `validateTargets(config: ConsumerConfig, catalog: ResolvedCatalog, versions: readonly ResolvedVersion[]): readonly Diagnostic[]`.
- `catalog/rules/`, `catalog/profiles/`, `catalog/retired.json`: released data.
- `standards/`: canonical sources; `fixtures/calibration/`: cases; `docs/provenance/`: source records.

Diagnostic = {code, path, message}; ValidationResult = {ok, diagnostics}; ResolvedVersion = {target_id, package_name, version, source_path}; ResolvedCatalog contains sorted profileIds, rules, sources. CatalogError codes: invalid-schema, duplicate-id, retired-id, missing-source, unsafe-path, unknown-profile, profile-cycle, unsupported-version. Schema shapes follow spec §5; exported types mirror them.

### Task 1 — #3: toolchain and honest CI

**Files:** create `package.json`, `pnpm-lock.yaml`, `tsconfig.json`, `tsconfig.build.json`, `tsconfig.test.json`, `eslint.config.mjs`, `.gitignore`, `src/cli/main.ts`, `tests/package-smoke.test.ts`, `scripts/package-smoke.mjs`, `.github/workflows/ci.yml`, `AGENTS.md`, `docs/development.md`; modify `README.md`.

**Interfaces:** `main(argv: readonly string[]): Promise<number>`; initially --help/--version only, pending init/sync/check return not-implemented nonzero. Executable guard prevents execution on import.

- [ ] Resolve compatible TypeScript, ESLint and pnpm from primary package metadata; commit exact versions and lockfile. Add only currently used dependencies.
- [ ] Write package smoke around isolated tarball: `assert.equal(runBin('--version').stdout.trim(), manifest.version)`; `assert.notEqual(runBin('init').status,0)`. Define runBin locally using spawnSync.
- [ ] Run `pnpm test`; expect missing-bin/implementation failure, not registry failure.
- [ ] Implement minimal bin, compiler/scripts from index, ESM/strict, engines `^22.0.0 || ^24.0.0`, files allowlist and no install-hook.
- [ ] Add Node 22/24 CI matrix, frozen lockfile, read-only permissions and no provider secret. Document instructions, commands and license inventory.
- [ ] Run `pnpm verify`; expect exit 0 and no false command support claim.
- [ ] Commit `chore: establish package toolchain and deterministic CI`; open draft PR and follow index acceptance.

### Task 2 — #4: schemas, resolver and provenance

**Files:** create modules/schemas from File Structure, `tests/catalog.test.ts`, `tests/config.test.ts`, `tests/digest.test.ts`, `tests/support/catalog-fixture.ts`, `scripts/validate-catalog.mjs`.

**Interfaces:** catalog/config functions above. Fixture helper creates/disposes real directories. Validation must not infer semantic rule violations.

- [ ] Port template validator negative cases by meaning. Add tests for unknown fields/schema, duplicate/retired IDs, filenames, missing source, traversal/symlink, profile cycles and invalid exception overlaps.
- [ ] Add assertions `assert.deepEqual(resolveProfiles(catalog,['typescript']).profileIds,['general','javascript','typescript'])`; title-only change preserves digest; violation/source change changes it.
- [ ] Run focused tests; expect absent exports/unmet assertions.
- [ ] Implement closed JSON schemas with Ajv, normalized paths plus realpath ancestry, sorted graph traversal and semantic hashing. Use semver for ranges; add each dependency only where used.
- [ ] Implement targets/local sources/exceptions; reject unknown IDs/actions and ambiguous overlaps. No hidden precedence by array order.
- [ ] Run focused tests and `pnpm catalog:validate`; unreleased empty skeleton is explicit, release eligibility rejects advertised empty profiles.
- [ ] Commit `feat: validate catalog profiles and consumer configuration`.

### Task 3 — #5: lifecycle and base-policy authority

**Files:** create `docs/review/contract.md`, `docs/review/lifecycle.md`, `docs/review/policy-changes.md`, `src/review/types.ts`, `src/review/policy-context.ts`, `tests/policy-context.test.ts`; modify AGENTS routing.

**Interfaces:** `selectReviewPolicy(base: PolicySnapshot, proposed: PolicySnapshot): PolicyContext`; PolicySnapshot = {revision, effective_policy_digest, config, rules}; PolicyContext = {governing, proposed, ownerApprovalRequired}.

- [ ] Write weakening test: base rule blocking, head exception disables it; `assert.equal(ctx.governing.rules[0].severity,'blocking')`; `assert.equal(ctx.ownerApprovalRequired,true)`.
- [ ] Run focused test and observe missing authority selection.
- [ ] Implement immutable base selection and single-owner contract for POLICY/CORRECTNESS/OPINION, semantic changes, retirement, evidence, independent review and bootstrap.
- [ ] Document add/clarify/split/merge/retire/downgrade/deterministic-promotion/unavailable-reviewer procedures with actor/input/evidence/output. No automerge.
- [ ] Run focused tests; manually inspect no-self-weakening examples and links. Avoid documentation-text-matching tests.
- [ ] Commit `docs: define rule lifecycle and base-policy authority`.

### Task 4 — #6: migrate general standards

**Files:** create `docs/provenance/workspace-template.md`, `standards/general/change-discipline.md`, `standards/general/requirement-ownership.md`, `catalog/profiles/general.json`, relevant `catalog/rules/PRR-NNN.json`, matching `fixtures/calibration/PRR-NNN.json`, `tests/catalog-admission.test.ts`; update retired ledger.

**Interfaces:** existing validator/resolver; data-only change, no provider API.

- [ ] Inventory PRR-001–007 at template revision da3aa6759c206485752c05f6643099188afe7f5a; record migrate/adapt/defer-to-profile/retired, source and license. PRR-006 stays retired.
- [ ] Write admission assertions: migrated rule with only historical evidence stays advisory; PRR-006 is absent from active rules and present in ledger.
- [ ] Run admission tests and confirm incomplete migration fails.
- [ ] Transfer verified general semantics and minimal sources. Preserve ID only for unchanged responsibility; framework/runtime rules remain owned by their later tasks.
- [ ] Add violation/boundary fixtures without product files. Do not invent rules solely to fill profiles.
- [ ] Run catalog validation/admission tests; inspect every source mapping.
- [ ] Commit `feat: migrate traceable general engineering standards`.

### Task 5 — #7: JS, TS and Node profiles

**Files:** create `catalog/profiles/{javascript,typescript,node}.json`, `standards/{javascript,typescript,node}/contracts.md`, `docs/provenance/runtime-languages.md`, `tests/runtime-profiles.test.ts`; extend rules/fixtures.

**Interfaces:** resolveProfiles and validateTargets; no new semantic inference engine.

- [ ] Verify primary sources and technology ranges; record URLs, versions and revisions.
- [ ] Add mixed-runtime/version tests: distinct target versions retain separate diagnostics; browser target excludes server-only candidates. Candidate filtering is not a final applies_when decision.
- [ ] Run focused tests; confirm missing profiles/compatibility fail.
- [ ] Implement profile content and compatibility; keep lint/typecheck/testing ownership distinct from AI decisions.
- [ ] Add explicit violation/boundary fixture pairs. Semantic classifications await #14.
- [ ] Run catalog validation and focused tests; pure Node requires neither Nest nor Nx.
- [ ] Commit `feat: add version-bounded language and Node profiles`.

### Task 6 — #8: Nx and NestJS profiles

**Files:** create `catalog/profiles/{nx,nestjs}.json`, `standards/{nx,nestjs}/contracts.md`, `docs/provenance/nx-nestjs.md`, `tests/nx-nest-profiles.test.ts`; extend rules/fixtures.

**Interfaces:** existing catalog/profile APIs.

- [ ] Verify official sources for discovery/generators/DI/config/lifecycle and record ranges.
- [ ] Add HTTP/context cases: Nest includes Node/TS but not Nx unless selected; context-only project does not require HTTP/DB adapter.
- [ ] Run focused tests and verify missing profile failures.
- [ ] Author atomic rules, sources and fixture pairs. Do not copy AccounterBro names or duplicate deterministic import checks.
- [ ] Run validation/tests; inspect Nx/no-Nx and HTTP/context applicability.
- [ ] Commit `feat: add scoped Nx and NestJS standards`.

### Task 7 — #9: Next.js and Angular profiles

**Files:** create `catalog/profiles/{nextjs,angular}.json`, `standards/{nextjs,angular}/contracts.md`, `docs/provenance/frontend.md`, `tests/frontend-profiles.test.ts`; extend rules/fixtures.

**Interfaces:** existing APIs and target mode/runtime mappings.

- [ ] Verify official supported versions/modes and record server/client assumptions.
- [ ] Add JS-only Next assertion `assert.equal(resolveProfiles(catalog,['nextjs']).profileIds.includes('typescript'),false)`; Angular never includes Next/Nest/EDP; unknown rendering mode yields compatibility diagnostic.
- [ ] Run focused tests; observe missing profile/mode handling.
- [ ] Author bounded sources/rules and fixture pairs for server-only data and test ownership.
- [ ] Run validation/tests; no inference of model semantic correctness from structural checks.
- [ ] Commit `feat: add Next.js and Angular profile boundaries`.

### Task 8 — #10: EDP consumption profile

**Files:** create `catalog/profiles/edp.json`, `standards/edp/public-contracts.md`, `docs/provenance/edp.md`, `tests/edp-profile.test.ts`; extend rules/fixtures.

**Interfaces:** existing profile APIs; version data comes from inspected EDP source, not conversation memory.

- [ ] Inspect actual public exports, Read/Operation/UseCase contexts, envelope and runtime ownership; pin source revision and package ranges.
- [ ] Add compatibility assertions: inspected version passes, unverified breaking major fails; fixture pair distinguishes duplicated mechanism from service-owned composition.
- [ ] Run focused tests; confirm missing content/bounds fail.
- [ ] Implement public-contract rules and sources without changing EDP or requiring Documents.
- [ ] Run validation/tests; rules remain advisory until #14 evidence.
- [ ] Commit `feat: describe verified EDP consumption contracts`.

## Completion

Each issue produces independently reviewable content and deterministic checks. No inherited evidence is advertised as new calibration; unfinished profiles are not shipped as supported.
