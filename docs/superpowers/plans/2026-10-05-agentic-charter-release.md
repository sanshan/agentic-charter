# Release and adoption Implementation Plan

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


**Goal:** Implement #16–#17: a complete verified tarball, controlled release procedure and real consumer adoption PR.

**Architecture:** Pack and offline installation prove delivery before publication. Pilot evidence and owner-controlled publication remain distinct actions.

## Review Focus

1. Tarball works only from source checkout — #16 installs and runs in a separate directory.
2. Source/fixture paths leak private material — #16 tests package allowlist and rejects private observations/transcripts.
3. Release publishes a different commit than the approved one — #16 binds tag/version/digest to approved SHA.
4. Consumer PR references a local temporary tarball — #17 provides a durable, verifiable prepublication artifact.
5. Restore only manifest version after rollback — #17 restores dependency/config/lockfile/output together.

## File Structure and Interfaces

- `scripts/verify-package.mjs`: standalone tarball installation verification; no registry access during tests.
- `src/release/validate.ts`: `validateRelease(input: ReleaseInput): ValidationResult`.
- ReleaseInput = {package_version, tag_version, changelog_version, source_sha, approved_sha, tarball_digest, expected_tarball_digest, pilot_accepted}.
- `tests/package-install.test.ts`, `tests/release.test.ts`, `tests/pilot.test.ts`: package and acceptance scenarios.
- `.github/workflows/release.yml`: explicit owner-controlled release, never pull-request publishing.
- `docs/releases/{process,compatibility}.md`, `docs/consumer/{adoption,troubleshooting}.md`.
- `evidence/pilot/report.md`: factual results and remaining setup, no synthetic clean.

### Task 14 — #16: release tooling and complete tarball

**Files:** create paths above except pilot-specific files; modify package files allowlist and CI package-smoke step.

**Consumes:** schemas/catalog, delivery CLI, templates and provider docs.
**Produces:** pack-verified artifact and release validation. This issue does not publish first release.

- [ ] Write isolated-package test: packed bin runs --help/version/init/check/sync outside checkout; selected sources/templates/schemas are readable and no source-only imports exist.
- [ ] Write release mismatch matrix: version/tag/changelog/SHA/digest/pilot mismatch each rejects release. Assert no private fixtures/transcripts/secrets in tarball file list.
- [ ] Run tests and capture packaging/policy failures.
- [ ] Implement deterministic allowlist packaging and installation smoke; preserve required attribution/license files. No lifecycle hooks modifying consumers.
- [ ] Implement release validation and manual workflow: protected owner approval, exact approved commit, tag/version/changelog consistency, complete package verification and recorded digest.
- [ ] Document SemVer rules: new/wider blocking or incompatible schema/CLI => major; opt-in/advisory => minor; editorial => patch; pre-1.0 incompatible => minor with migration.
- [ ] Verify current npm publishing documentation and scope rights. Owner configures trusted publishing where available; otherwise minimal token in protected release context. Never expose credentials to PR.
- [ ] Run `pnpm verify` and `pnpm release:check` against fixture metadata; produce unpublished pack artifact with digest. A dry-run is not publish evidence.
- [ ] Commit `build: verify package releases and protected publication`.

### Task 15 — #17: end-to-end pilot and consumer PR

**Files:** create `tests/pilot.test.ts`, `fixtures/consumers/{clean,existing,mixed}/README.md`, `docs/consumer/adoption.md`, `docs/consumer/troubleshooting.md`, `evidence/pilot/report.md`.
**Consumer files:** root package manifest/lockfile, agentic-charter.config.json, .agentic-charter generated bundle, AGENTS managed block, owned charter workflow; inspect current applicable instructions before edits.

**Interfaces:** packed CLI only for consumer acceptance; no importing unexported source functions to bypass packaging.

- [ ] Add a pilot test using tarballs A and compatible B: install A → init → check 0 → local additions → upgrade B → sync → check 0 → conflict → assert no writes → restore full prior snapshot → check 0.
- [ ] Add existing AGENTS/Nx/workflow and mixed-profile cases. Delete node_modules after successful generation and assert all selected rule/source references resolve locally; CLI verification itself still requires installation.
- [ ] Run pilot tests; fix only concrete uncovered integration defects in owning modules and preserve task boundaries.
- [ ] Inspect current workspace-template branch and all applicable AGENTS. Decide adoption base from real state, not old source snapshot. Retain database/SQL CI and local business/architecture ownership.
- [ ] Prepare a separate draft consumer PR. Before npm publication, use a durable GitHub-hosted tarball in the consumer PR or another owner-approved immutable artifact with checksum; a committed temporary pilot vendor tarball is allowed in that draft only. Never leave a local /tmp dependency or nonexistent npm version.
- [ ] Run consumer deterministic checks against installed tarball and generated content. Replace duplicate general review ownership with routing; retain genuinely local requirements.
- [ ] If owner configured and authorized provider review, run actual fresh-head review smoke after CI and verify new-head invalidation. Otherwise record unconfigured/pending and exact owner steps; do not label it passed.
- [ ] Write report mapping every epic criterion to task/commit/test/artifact, distinguishing deterministic, simulated and live runs. Block publication on failed integral tests, unresolved findings or missing owner acceptance.
- [ ] Commit `test: verify adoption and document pilot evidence`.
- [ ] Only after accepted pilot and explicit publication authorization, execute the #16 release path. Then replace draft consumer's pilot artifact dependency with published version, regenerate lock/output and rerun CI/review for that new head before consumer merge.

## Completion

Delivery, evidence, release authorization and actual publication have separate recorded statuses. Epic merge happens only after integral verification and required current-head review; no missing credential or quota is converted into success.
