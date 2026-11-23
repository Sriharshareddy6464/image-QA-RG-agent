# IQARG --- Parallel Execution Plan

**Document:** `docs/PLAN.md`\
**Project:** IQARG (Image QA Regeneration)\
**Execution branch:** `stage`\
**Plan version:** 1.0\
**Status:** Execution baseline

------------------------------------------------------------------------

## 1. Objective

Build the IQARG local-first MVP through four independently owned
engineering territories, executed in parallel against one shared Git
branch: `stage`.

The project Bible is the authority for product scope, architecture,
engineering rules, and API behaviour. Agents implement within those
boundaries. They do not independently redefine the product or change
shared contracts.

The objective is parallel delivery without branch management,
uncontrolled shared-file edits, speculative refactoring, or unverified
integration.

## 2. Source of truth

All agents must read these files before implementation:

1.  `AGENTS.md` --- agent operating rules.
2.  `docs/PROJECT_BRIEF.md` --- product requirements and MVP boundaries.
3.  `docs/ARCHITECTURE.md` --- architecture and component
    responsibilities.
4.  `docs/API_CONTRACT.md` --- frontend/backend interface contract.
5.  `docs/PLAN.md` --- territory ownership, execution protocol,
    dependencies, and acceptance.

If a required file is missing, an agent must report that fact. It must
not invent the missing specification.

### Change control

-   No agent may silently alter the project Bible.
-   A genuine conflict, omission, or implementation blocker must be
    reported with the exact file/section and evidence.
-   Proposed contract or architecture changes require approval before
    implementation.
-   Keep confirmed defects, architectural risks, and optional
    improvements separate.
-   Do not make speculative code changes or broad refactors.

## 3. Four agent territories

Each agent owns the paths listed below. Ownership is exclusive.

  -------------------------------------------------------------------------------------
  Agent             Territory         Owned paths               Primary responsibility
  ----------------- ----------------- ------------------------- -----------------------
  Codex             Backend           `backend/**`,             FastAPI application,
                    application       `tests/backend/**`        API implementation,
                                                                backend services,
                                                                backend tests

  OpenCode          Frontend          `frontend/**`,            React/Vite/TypeScript
                    application       `tests/frontend/**`       UI, canvas interaction,
                                                                client API integration,
                                                                frontend tests

  Antigravity       Local runtime and `infrastructure/**`,      Docker Compose, local
                    model adapters    `models/**`,              service wiring,
                                      `tests/runtime/**`        model/runtime adapters,
                                                                ComfyUI integration and
                                                                runtime tests

  Claude Code       Integration QA    `tests/integration/**`,   Cross-boundary
                                      `docs/QA_REPORTS/**`      integration tests,
                                                                contract verification,
                                                                reproducible defect
                                                                reports and test
                                                                evidence
  -------------------------------------------------------------------------------------

### Ownership restrictions

-   Agents must not edit files outside their owned paths.
-   No agent may edit `AGENTS.md`, `PROJECT_BRIEF.md`,
    `ARCHITECTURE.md`, `API_CONTRACT.md`, or this plan without explicit
    approval.
-   The root `tests/` directory is divided by the subdirectories above.
    Do not place files directly in `tests/` unless approved.
-   Shared configuration, root-level manifests, and shared generated
    files are not automatically owned by any agent. Report the need and
    wait for assignment.
-   Do not create duplicate implementations of the same service or
    interface in another territory.
-   Claude's final review is read-only. Claude may create a QA report
    under `docs/QA_REPORTS/`, but must not patch another agent's
    implementation during review.

## 4. Work packages

These are substantial territory-level work packages, not one-endpoint
exercises. Each agent should decompose its territory internally into
coherent implementation tasks and commit each completed task separately.

### T1 --- Backend application (Codex)

Deliver a working backend aligned with the API contract:

-   FastAPI application structure and startup.
-   Configuration and environment handling.
-   Health endpoint and centralized error handling.
-   Image upload, validation, metadata, and persistent local source
    storage.
-   Segmentation request/service boundary and adapter interface.
-   Mask retrieval and refinement/approval service boundaries.
-   Generation/job orchestration API boundary.
-   Progress/event interface required by the contract.
-   Backend unit and API tests.
-   Backend run instructions and dependency declarations within
    `backend/`.

Do not invent a segmentation model or inpainting implementation. Adapter
interfaces may be implemented before a model is selected, but
model-specific behaviour must remain explicit and replaceable.

### T2 --- Frontend application (OpenCode)

Deliver the complete MVP user-facing workflow:

-   React/Vite/TypeScript application structure.
-   Image upload and source-image display.
-   Canvas interaction using original-image pixel coordinates.
-   Point/region selection UI and segmentation request integration.
-   Candidate mask display, refinement controls, and explicit approval
    state.
-   Prompt entry and generation submission.
-   Job progress display and result retrieval.
-   Before/after comparison and accept/reject/retry controls.
-   Clear loading, empty, validation, and error states.
-   Frontend tests and local run instructions within `frontend/`.

The frontend must use the API contract. It must not call ComfyUI or
model services directly.

### T3 --- Local runtime and model adapters (Antigravity)

Deliver a reproducible local execution environment:

-   Docker Compose and service definitions under `infrastructure/`.
-   Local volume/storage wiring and environment configuration.
-   Backend/frontend service connectivity where required by the
    architecture.
-   ComfyUI integration boundary and backend-side adapter implementation
    under `models/` where appropriate.
-   Model/runtime capability checks and explicit configuration.
-   Hardware-aware execution defaults for the documented local machine.
-   Startup, health, failure, and shutdown behaviour.
-   Runtime tests under `tests/runtime/`.
-   Local setup and troubleshooting documentation within owned paths.

Do not assume that a candidate model fits the available GPU memory. Do
not silently download or select large models. Document model
prerequisites, VRAM/RAM expectations, and any unverified capability.
Cloud GPU provisioning is out of scope.

### T4 --- Integration QA (Claude Code)

Deliver independent integration evidence:

-   Read and understand all four Bible documents and this plan.
-   Create integration tests under `tests/integration/` based on the
    published API contract.
-   Verify frontend/backend request and response compatibility using
    available implementations.
-   Verify workflow state transitions and required approval gates.
-   Verify failure handling and job progress behaviour where
    implemented.
-   Verify source-image immutability and the pixel-preservation
    requirement at the compositing boundary when that capability exists.
-   Produce a QA report under `docs/QA_REPORTS/` with test commands,
    results, reproducible defects, and untested areas.

Claude must not implement backend, frontend, or runtime features. During
final review, Claude is an independent reviewer, not a second
implementation owner.

## 5. Dependencies and parallelism

All four agents can begin independent work after reading the Bible and
confirming the repository state.

Expected dependency relationships:

-   Frontend and backend agree on `API_CONTRACT.md`; neither changes it
    unilaterally.
-   Runtime work consumes the backend/frontend service interfaces; it
    must not invent alternate API routes.
-   Integration tests can be authored against the contract before all
    features are complete. Tests that cannot yet run must be clearly
    marked as blocked by missing implementation, not reported as passed.
-   Integration QA performs its final end-to-end run after the three
    implementation territories report completion.

Agents must not wait idly for unrelated territories. They should
implement against the documented contract and use mocks or test doubles
where appropriate, clearly identifying what remains unverified against a
real service.

## 6. Single-branch Git protocol

**All work is performed on `stage`. No feature branches and no
worktrees.**

Before starting, each agent must:

1.  Run `git status --short --branch`.
2.  Confirm the current branch is `stage`.
3.  Inspect existing changes before editing.
4.  Preserve all pre-existing user or agent changes.
5.  Stop if the branch is not `stage` or if an unexpected change
    overlaps its territory.

### Shared working-tree rules

-   Agents may edit concurrently only when their file paths are
    disjoint.
-   No two agents may edit the same file.
-   No agent may run `git reset`, `git checkout --`, `git clean`, or
    other destructive Git operations.
-   No agent may stage or commit another territory's files.
-   Do not use `git add .` or `git add -A`.
-   Do not commit while another agent is performing a Git staging/commit
    operation.
-   Git staging and commit operations are serialized: only one agent may
    stage/commit at a time.
-   Before committing, inspect `git status` and `git diff --cached` to
    confirm only that task's owned files are staged.
-   Commit completed tasks individually using explicit paths.
-   All commits must be made on `stage`.
-   Do not push, deploy, or publish unless explicitly authorized.

### Required commit procedure

For each completed task:

1.  Run relevant tests and record exact commands/results.
2.  Run `git diff --check`.
3.  Review `git diff` for scope and unintended changes.
4.  Stage only explicit task-owned paths, for example:
    `git add backend/path/to/file.py tests/backend/path_to_test.py`
5.  Inspect `git diff --cached` and `git status`.
6.  Commit with a descriptive message containing the task ID, for
    example: `git commit -m "IQARG-T1.1: establish backend foundation"`
7.  Report the resulting commit hash and final status.

If another agent has staged files, do not commit or alter the index.
Stop and notify the execution coordinator. Resume only when the index is
clear and the commit turn is available.

## 7. Task IDs and commit granularity

Use these territory prefixes:

-   `T1` --- Codex backend.
-   `T2` --- OpenCode frontend.
-   `T3` --- Antigravity runtime/model integration.
-   `T4` --- Claude integration QA.

Each agent must define a short internal task sequence before substantial
edits. Example:

-   `IQARG-T1.1` --- backend foundation.
-   `IQARG-T1.2` --- image ingestion and storage.
-   `IQARG-T1.3` --- segmentation/mask API boundary.
-   `IQARG-T1.4` --- generation/job API boundary.

These are illustrative decomposition examples, not permission to ignore
the actual contract or to claim unfinished functionality is complete.

Each completed task gets its own commit. Do not create one enormous
end-of-project commit.

## 8. Definition of done

A task is complete only when:

-   Its deliverables exist in the assigned territory.
-   Its implementation follows the Bible and API contract.
-   Relevant automated tests pass, or blocked tests are explicitly
    identified.
-   `git diff --check` passes.
-   The diff contains no unrelated changes.
-   No secrets, credentials, local machine-specific absolute paths, or
    large model binaries are committed.
-   The task is committed on `stage`.
-   A handover report is provided.

A test must not be described as passing unless it was actually executed
and its result observed. Mocked tests must be identified as mocked.

## 9. Mandatory agent handover format

Every agent reports:

1.  **Agent and territory**
2.  **Task IDs completed**
3.  **Files created/modified**
4.  **Implementation delivered**
5.  **Tests executed** --- exact commands and pass/fail counts
6.  **Tests not run** --- reason
7.  **Contract/architecture deviations** --- none, or exact details
8.  **Confirmed defects discovered** --- evidence and reproduction steps
9.  **Known limitations and blockers**
10. **Git commits** --- task ID, commit hash, message
11. **Final Git status**
12. **Next dependency or required decision**

Do not report only "done" or "all tests pass."

## 10. Final integration review

After T1, T2, and T3 implementation tasks are committed, and T4's
integration tests are ready:

1.  Run the integration suite against the combined `stage` state.
2.  Verify the actual frontend-to-backend-to-runtime workflow.
3.  Verify that the UI cannot bypass mask approval.
4.  Verify that failed generations do not overwrite accepted results.
5.  Verify source-image immutability and exact preservation outside the
    approved final compositing mask.
6.  Verify coordinate conventions and mask semantics across boundaries.
7.  Verify local startup and shutdown from the documented instructions.
8.  Record failures with exact reproduction steps and evidence.
9.  Produce a consolidated QA report.

The reviewer must classify findings as:

-   **P0 --- Critical:** data loss, source corruption, security breach,
    or fundamental workflow failure.
-   **P1 --- High:** core MVP workflow broken or contract
    incompatibility.
-   **P2 --- Medium:** significant functional defect with a workaround.
-   **P3 --- Low:** minor defect or usability issue.
-   **Risk:** plausible concern not yet reproduced.
-   **Suggestion:** optional improvement, not a defect.

No code correction is authorized merely because a reviewer prefers a
different style. Corrections require a confirmed defect, contract
violation, or approved design decision.

## 11. Product and technical guardrails

-   MVP runs locally through Docker containers.
-   Cloud GPU usage is deferred.
-   No premature Kubernetes, Redis, database, authentication, or
    distributed orchestration.
-   Do not assume a model's capabilities or hardware fit; benchmark and
    document them.
-   Original uploaded image remains immutable.
-   Candidate mask requires explicit user approval before generation.
-   Pixels outside the final approved compositing mask must remain
    identical to the original.
-   A failed generation must not overwrite an accepted result.
-   Coordinate system: original-image pixel coordinates, top-left
    origin, x increases right, y increases down, zero-based.
-   Mask values: `0` outside and `255` inside, unless the API contract
    is formally revised.
-   Frontend communicates with model/generation services through the
    backend only.
-   No agent may expand MVP scope without approval.

## 12. Execution start checklist

Before launching the four agents:

-   [ ] Confirm current branch is `stage`.
-   [ ] Confirm the four Bible files exist.
-   [ ] Inspect existing `git status` and preserve any in-progress Codex
    work.
-   [ ] Confirm each agent has read the same plan and Bible.
-   [ ] Give each agent only its territory prompt and ownership paths.
-   [ ] Start independent work in disjoint directories.
-   [ ] Serialize all staging and commit operations.
-   [ ] Collect all four handover reports.
-   [ ] Run consolidated integration QA.
-   [ ] Decide acceptance based on evidence, not agent self-assessment.

------------------------------------------------------------------------

**Operating principle:** Parallelize implementation, not ownership. One
Bible, one branch, exclusive file territories, serialized Git commits,
evidence-based integration review.
