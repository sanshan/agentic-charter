# Review calibration and curator Implementation Plan

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


**Goal:** Implement #13–#15: honest Codex review integration, controlled calibration and manually started curator tasks.

**Architecture:** Deterministic validation records facts and rejects bad bindings. Codex performs review externally; an owner assesses ambiguous clean evidence and accepts catalog changes.

## Review Focus

1. Same-head clean-looking prose hides an unresolved finding — #13 stays blocked.
2. Review matches SHA but wrong policy digest — #13 marks it stale.
3. Calibration prompt contains expected answer or old finding — #14 rejects contaminated trial.
4. A hidden provider model version is invented — #14 stores provider-managed/unknown.
5. Replayed or hostile cross-project observation — #15 deduplicates and cannot change authority.

## File Structure and Interfaces

- `src/review/types.ts`: ReviewResult, ReviewContext, ProviderCapabilities, Finding, Assessment; fields/enums follow spec §5.
- `src/review/codex.ts`: `codexCapabilities(): ProviderCapabilities`; machineClean=false, generalTaskRunner=false, requiresOwnerAssessment=true.
- `src/review/validate-result.ts`: `validateReviewResult(result: unknown, context: ReviewContext): ValidationResult`; validates structure/bindings, never returns permission to merge.
- `src/review/state.ts`: `assessRecordedState(result: ReviewResult, context: ReviewContext): ReviewState`.
- ReviewContext = {repository, base_sha, head_sha, effective_policy_digest, currentBlockingFindingIds, ciSucceededBeforeReview}; ReviewState uses exact spec enum.
- `src/calibration/types.ts`: CalibrationCase, TrialRecord, CalibrationReport; expected and observed stay separate.
- `src/calibration/prepare.ts`: `prepareCase(input: CalibrationCase): EvaluatorInput`.
- `src/calibration/validate.ts`: `validateCalibration(rule: Rule, cases: readonly CalibrationCase[], trials: readonly TrialRecord[]): CalibrationReport`.
- `src/curator/types.ts`: Observation, IntakeDecision, CuratorLimits, LedgerEntry.
- `src/curator/intake.ts`: `evaluateIntake(observation: Observation, ledger: readonly LedgerEntry[], limits: CuratorLimits): IntakeDecision`.
- IntakeDecision = {state: 'candidate'|'duplicate'|'rejected'|'paused', reason, key}; key binds project_id/source/content digest, never global text alone.

### Task 11 — #13: generated workflow and provider evidence

**Files:** create `templates/workflows/agentic-charter-check.yml`, `templates/review/codex-setup.md`, `templates/review/triggering.md`, review modules above, `tests/workflow.test.ts`, `tests/review-state.test.ts`, `docs/review/provider-capabilities.md`; extend renderBundle.

**Consumes:** Part 1 base PolicyContext and schema validation; Part 2 bundle generator and CLI check.
**Produces:** workflow/setup assets, validated evidence records, no provider-owned synthetic status.

- [ ] Write workflow parser tests: trigger pull_request, read-only contents, no pull_request_target, no secrets/write token, checkout persist-credentials=false. Existing consumer CI stays byte-identical.
- [ ] Write state cases: unconfigured, pending, failed(auth/quota/timeout/cancelled/unknown), stale SHA/digest, policy-integrity failure, blocked current finding, indeterminate prose, completed owner assessment. Assert capabilities.machineClean=false.
- [ ] Add explicit tests: clean-looking later response + currentBlockingFindingIds => blocked; same SHA/wrong digest => stale; self-review or pre-CI result cannot complete independent review.
- [ ] Run focused tests; expect absent templates/validation failures.
- [ ] Implement deterministic workflow template using selected manager and committed lockfile. Use frozen install, disable unnecessary install scripts, provide only read permissions. Never execute PR head under privileged event context.
- [ ] Implement result schema validation/state handling. Owner-assessed clean is labeled as such; a validate command success only means record valid, not machine review passed. Do not parse Codex prose/reactions to infer result.
- [ ] Verify current official Codex setup docs and document owner App/account/environment/permissions/quota setup, draft→CI→ready trigger and retry after new head. Note historical evidence separately; no live configuration claim without real probe.
- [ ] Document base policy access for reviewer and owner verification of review scope. Current integration cannot enforce model instruction selection; ambiguous scope remains indeterminate.
- [ ] Run `pnpm verify`; optional live smoke only after owner setup and explicit trigger authorization. Report skipped/unconfigured honestly.
- [ ] Commit `feat: deliver deterministic workflow and Codex review runbook`.

### Task 12 — #14: controlled calibration

**Files:** create calibration modules above, `schemas/calibration-{case,trial}.schema.json`, `scripts/{prepare-calibration,validate-calibration}.mjs`, `docs/review/calibration.md`, `tests/calibration.test.ts`, `tests/calibration-input.test.ts`, `evidence/calibration/README.md`.

**Interfaces:** CalibrationCase = {id, rule_id, expected, context, diff, baseline_revision}; TrialRecord binds semantic/context/fixture digests, context_id, provider, nullable model metadata, CI refs, review refs, full SHA, observed, origin and owner verification. CalibrationReport = {eligible, diagnostics, evidence_refs}; eligibility is necessary structure, not automatic promotion.

- [ ] Write fixture matrix: two fresh known-violation trials + each boundary eligible; reused context, wrong digest, omitted boundary, unrelated CI failure, contradiction or fake origin ineligible.
- [ ] Assert `!Object.hasOwn(prepareCase(input),'expected')`; evaluator input excludes prior classifications/history as well as expected values. Clean boundary observed result does not invent a subtype.
- [ ] Run focused tests and capture missing checker/input isolation.
- [ ] Implement canonical digest binding, trial identity checks and ALL-attempt accounting. Candidate blocking definition is what trials evaluate; do not reuse advisory digest when promoting severity.
- [ ] Record unavailable model identity as provider-managed/unknown. Do not silently claim identical hidden model across trials.
- [ ] Add runbook: prepare disposable PR → unrelated deterministic CI green → independent Codex review → owner records artifacts → close without merge. Repeats use new PR contexts; quota/auth failures remain recorded.
- [ ] Run offline checks. Perform actual trials only when provider is configured and owner authorizes sending review requests; otherwise keep candidate rules advisory and record pending live calibration.
- [ ] Commit `feat: validate isolated calibration evidence`.

### Task 13 — #15: manually invoked curator

**Files:** create `schemas/observation.schema.json`, curator modules above, `templates/curator/task.md`, `docs/curator/runbook.md`, `observations/README.md`, `observations/ledger.json`, `tests/curator-intake.test.ts`, `tests/curator-task.test.ts`.

**Interfaces:** Observation follows spec §11 plus public_reuse_allowed and confirmed flags. CuratorLimits = {allowed_paths, max_candidates, max_attempts}; task rendering takes explicit owner input, no scheduled trigger.

- [ ] Add replay test: same project/source/digest after candidate PR exists => duplicate. Different project is separate evidence; unsupported/unconfirmed/private material cannot become a publishable candidate.
- [ ] Add limits/malicious cases: attempt to alter policy gates/permissions rejected for normal curator task; exhausted attempts => paused; injected instruction text cannot change allowed_paths or owner requirements.
- [ ] Run focused tests and confirm absent intake guard.
- [ ] Implement deterministic schema checks/dedupe/limits; semantic analysis remains the manually launched agent's task. Do not claim arbitrary prompt injection is solved by a string scanner.
- [ ] Author task contract: inspect existing rules and deterministic owners, prepare one bounded draft PR or reasoned rejection, update ledger, preserve project boundaries, no credentials/private transcripts, no merge or self-approval.
- [ ] Document resume/retry/quota failures, rejection reasons and rollback; do not create a scheduled/API worker.
- [ ] Run `pnpm verify` and fixture walkthrough from intake to draft proposal. Real Codex task remains separately recorded; offline output is not live-agent success.
- [ ] Commit `feat: prepare bounded curator tasks and observation intake`.

## Completion

Generated check validates freshness only. Owner acceptance remains mandatory. Calibration and curator can be exercised offline without pretending an actual reviewer has run; missing live setup is explicit.
