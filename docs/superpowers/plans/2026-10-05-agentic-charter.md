# Agentic Charter Implementation Plan

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


**Goal:** Deliver the approved Agentic Charter v1 specification through the existing epic without losing task boundaries or making unverified provider claims.

**Architecture:** Four linked plans share typed interfaces and one delivery contract. The repository stays a single npm package; implementation proceeds through separate task PRs into the epic branch.

## Review Focus

1. Missing instructions/dependencies between task PRs — reread epic/all tasks and consume only merged interfaces.
2. Bootstrapping CI is mistaken for waiving independent code review — #3 still requires independent review before merge.
3. Profile content is called supported before source verification — #6–#10 and release admission reject premature claims.
4. Human review evidence is mistaken for machine authorization — #13 keeps them separate.
5. Pilot precedes first npm release — #16 produces tarball, #17 validates it before publication.

## Status and Execution Choice

Spec approved by owner message “Посмотрел давай начнем с этого.” on 2026-10-05. This document and its four linked parts are newly prepared and await plan review; no code task has started.

Recommended execution: **Native**, sequential issue-by-issue in this session using superpowers:executing-plans. The interfaces are coupled and the existing GitHub Codex reviewer remains the required independent reviewer after CI for every code task PR, plus whole-epic review at integration. This preserves the agreed per-PR gates; “Native” does not postpone all review until the end.

Alternative: **Subagent-driven**, fresh implementation/review contexts per task using superpowers:subagent-driven-development, while retaining the same external CI/review/owner gates. No subagents are launched by writing this plan. The owner chooses execution method before implementation.

The writing-plans skill normally offers whole-branch review for Native. The already-approved stronger repository requirement wins: each code PR gets independent review. If the configured provider is absent, prepare the branch and report the blocker; do not silently substitute self-review.

## Linked Plans and Ownership

| Plan | Issues | Independently testable deliverable |
| --- | --- | --- |
| [1. Catalog and standards](2026-10-05-agentic-charter-catalog.md) | #3–#10 | Package CI, schemas, profiles, sources and fixtures |
| [2. Delivery and synchronization](2026-10-05-agentic-charter-delivery.md) | #11–#12 | Standalone init/sync/check and recovery |
| [3. Review, calibration and curator](2026-10-05-agentic-charter-review.md) | #13–#15 | Workflow, evidence contract and manual task procedure |
| [4. Release and adoption](2026-10-05-agentic-charter-release.md) | #16–#17 | Verified tarball and consumer pilot PR |

The plans split implementation detail, not product scope or approval authority. Read this index, the spec, the owning part and its prerequisites before each task. File names in these plans are proposed new paths, not claims that code already exists.

## Common Development Commands (#3 defines them)

| Command | Exact responsibility |
| --- | --- |
| pnpm lint | ESLint on implementation/tests/scripts, no formatter-driven unrelated changes |
| pnpm typecheck | TypeScript --noEmit for source and tests |
| pnpm build | Compile source to dist with tsconfig.build.json |
| pnpm build:test | Compile source/tests into dist-tests with rootDir "."; tests import ../src modules |
| pnpm test | Clean/build test outputs, then node --test on explicitly discovered compiled *.test.js files |
| pnpm catalog:validate | Build then run scripts/validate-catalog.mjs against committed catalog (#4 adds) |
| pnpm package:smoke | Build/pack, install in temp directory without registry lookup, execute bin smoke |
| pnpm verify | lint + typecheck + test + build + package smoke; include catalog validate when available |
| pnpm release:check | Build then validate explicit local release metadata (#16 adds); does not publish |

Focused test command after #3: `pnpm build:test && node --test dist-tests/tests/<file>.test.js`. On first test-driven step, a missing export/compiler error is valid RED only if it is caused by the feature being absent; infrastructure/network failure is not RED. Then implement minimum behavior, rerun the same test and commit. Do not keep a permanently red skeleton task.

node:test is the one test framework. Use TypeScript compilation rather than requiring a second TS execution runtime. Create scripts/run-tests.mjs to enumerate compiled tests recursively to avoid shell glob differences. Package installation smoke reuses the pnpm store populated by the frozen repository installation and runs with --offline --ignore-scripts against the local tarball; it must not silently fetch missing dependencies. A cold offline cache is an explicit environment/setup error, not successful verification. Test installs preserve the declared runtime dependencies rather than bypassing them.

Commands in this table become real only when their owner task implements them.

## Dependency and Acceptance Sequence

| Issue | Dependencies | Part/task |
| --- | --- | --- |
| #2 | Spec + plan review + execution choice | This document and PR #18 |
| #3 | #2 | Part 1 task 1 |
| #4 | #2, #3 | Part 1 task 2 |
| #5 | #2, #4 | Part 1 task 3 |
| #6 | #4, #5 | Part 1 task 4 |
| #7 | #4, #5, #6 | Part 1 task 5 |
| #8 | #4, #5, #6, #7 | Part 1 task 6 |
| #9 | #4, #5, #7 | Part 1 task 7 |
| #10 | #4, #5, #6, #7 | Part 1 task 8 |
| #11 | #3, #4, #5, #6 | Part 2 task 9 |
| #12 | #4, #11 | Part 2 task 10 |
| #13 | #2, #5, #11, #12 | Part 3 task 11 |
| #14 | #4, #5, #13 | Part 3 task 12 |
| #15 | #5, #6, #12, #13, #14 | Part 3 task 13 |
| #16 | #3, #4, #11, #12, #13 | Part 4 task 14 |
| #17 | #6–#16 | Part 4 task 15 |

Default execution is numerical order #3→#17. #16 can be prepared earlier once its prerequisites merge; first publication still follows #17.

For each task:

- [ ] Read current epic/all subtasks, owning plan, spec and applicable AGENTS.
- [ ] Create task/<issue>-<slug> from current epic/1-agentic-charter; use an isolated checkout/worktree and preserve other work.
- [ ] Execute its small test→implementation→verification steps; commit coherent changes.
- [ ] Open a draft PR to epic branch, self-review, wait for exact-head deterministic CI.
- [ ] Request independent review only after green CI, using authorized Codex mechanism; all new significant heads restart CI/review.
- [ ] Resolve current blocking findings and record honest evidence; owner accepts catalog changes and merges per approved process.
- [ ] Close issue only after its criteria are met and change integrated. Never mark live integration or publication done based on mocks.

These checkboxes are a repeatable process template, not claims that a task has already been checked.

## Bootstrap and First Executable Task

Repository baseline is README commit 654f849b78f37d25a11318b9c4e084502f7dd9bd. PR #18 targets epic/1-agentic-charter and contains the spec/plan only.

After owner approves plan and chooses execution:
1. Record approval against the document commit.
2. The owner can accept the documentation-only bootstrap PR under spec §14; no invented CI/provider success.
3. Start #3 from updated epic branch and implement actual CI.
4. Verify #3 CI and independent review before code merge. Provider setup is a real prerequisite to merge, not to preparing code.
5. Later tasks start from integrated epic state and do not bypass missing predecessors.

No branch protection, App, secret or paid usage is configured by this plan. Ask for actual setup only when that concrete gate blocks progress.

## Spec Coverage Self-Review

| Spec section | Owner |
| --- | --- |
| 1–3 purpose, decisions, sources | #3 instructions + #6 provenance + this index |
| 4 package modules/runtime | #3, #4, #11, #13 |
| 5 data contracts | #4, #5, #12, #13, #14, #15 |
| 6 consumer ownership | #11, #12 |
| 7 CLI/conflict/recovery | #11, #12 |
| 8 trust/base policy/workflow | #5, #13 |
| 9 Codex capabilities/setup | #13 |
| 10 independent calibration | #14 |
| 11 manual curator | #15 |
| 12 release/rollback | #12, #16, #17 |
| 13 acceptance scenarios | Focused tests across all parts and #17 pilot |
| 14 branches/bootstrap/backlog | This index and #3 instructions |
| 15 design/plan approval | #2 remains open until accepted |

Interfaces checked across parts: ResolvedCatalog/ConsumerConfig → renderBundle; DesiredOutput/Snapshot → planSync; Plan → applyPlan; PolicySnapshot → PolicyContext; ReviewContext → recorded-state checks; Rule/digests → calibration. Result/evidence validation never becomes permission to merge.

No unresolved product placeholder is intentionally carried forward. Technology-specific rules/ranges are explicit research deliverables of #7–#10; they are not guessed in this plan. Exact dependency pins are selected against current metadata during #3. Provider UI/auth and npm publication capabilities are verified against official sources at #13/#16 before use.
