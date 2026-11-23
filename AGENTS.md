# IQARG — Coding Agent Rules

These rules apply to OpenCode, Antigravity, Codex, Claude Code, and any other coding agent working in this repository.

## 1. Project authority

- `docs/PROJECT_BRIEF.md` defines the product goal and MVP scope.
- `docs/ARCHITECTURE.md` defines the approved architectural direction.
- This file defines agent execution and collaboration rules.
- If instructions conflict, stop and report the conflict. Do not guess.
- ChatGPT is the project architecture owner and task orchestrator. The user makes final product decisions.

## 2. Non-negotiable execution rules

1. Work only on the explicitly assigned task.
2. One agent is the implementation owner for a task at a time.
3. Never have two agents edit the same files or overlapping functionality concurrently.
4. Do not start another task until the current owner has handed over and the orchestrator has reviewed the result.
5. Do not redesign, expand, or replace the architecture without approval.
6. Do not add dependencies, services, databases, model runtimes, or infrastructure unless the task authorizes them.
7. Do not modify files outside the task's allowed scope.
8. Do not delete or overwrite user work.
9. Do not commit, push, publish, deploy, or run destructive commands unless explicitly authorized.
10. Do not claim tests passed unless you ran them and can report the command and result.

## 3. Required task assignment format

Every implementation assignment must include:

```text
TASK ID:
TASK TITLE:
ASSIGNED AGENT:
OBJECTIVE:

CONTEXT / APPROVED REFERENCES:
- docs/PROJECT_BRIEF.md
- docs/ARCHITECTURE.md
- Relevant API contract or task-specific document

ALLOWED FILES:
- Explicit file paths or directories

PROHIBITED FILES / ACTIONS:
- Explicit exclusions
- No unrelated refactoring
- No unapproved architecture changes

ACCEPTANCE CRITERIA:
1.
2.
3.

REQUIRED TESTS:
- Exact test commands, or tests to add

HANDOVER:
- Files changed
- Summary of implementation
- Commands run and results
- Known limitations
- Decisions or blockers requiring approval
- Confirmation that no out-of-scope files were changed
```

An agent must ask for clarification if the allowed file scope or acceptance criteria are ambiguous.

## 4. Single-writer protocol

- Only the assigned agent may edit the task's allowed files while the task is active.
- Other agents may inspect or review only if explicitly authorized; reviewers must not edit.
- Before ownership transfers, the current agent provides its handover report.
- The orchestrator checks the diff and test evidence before assigning the next task.
- If unexpected files need modification, stop and request approval to expand scope.
- Do not run multiple agents in parallel against the same working tree.

## 5. Required workflow

For each task:

1. Read this file and the relevant project documents.
2. Inspect the current repository state before editing.
3. Restate the task and planned file changes.
4. Implement only the assigned scope.
5. Add or update relevant tests.
6. Run the required tests.
7. Inspect the final diff for accidental changes.
8. Provide the required handover report.
9. Stop. Do not pick up another task automatically.

## 6. Architecture change control

An agent may identify a better approach, but must submit a proposal rather than silently changing the design.

A proposal must include:

- Current behavior or limitation.
- Why a change is needed.
- Proposed alternative.
- Files/components affected.
- Benefits and trade-offs.
- Hardware, runtime, and dependency implications.
- Tests needed.
- Whether existing work or data must migrate.

Wait for explicit approval before implementing the change.

## 7. Dependency and security rules

- Prefer the smallest dependency set that satisfies the assigned requirement.
- Do not add a package because it may be useful later.
- Pin or constrain versions where appropriate and explain the choice.
- Validate uploaded file type, size, and image decoding.
- Do not trust client-provided filesystem paths.
- Do not expose secrets, environment variables, or internal host paths in API responses or logs.
- Do not disable security checks merely to make a test pass.
- Do not download or execute model code/scripts without explicit task authorization.

## 8. Hardware rules

The initial development machine has 6 GB VRAM and 16 GB system RAM.

- Do not assume a model fits based on advertised parameter count alone.
- Do not launch multiple large models concurrently without measured justification.
- Do not change GPU drivers, CUDA installations, WSL configuration, or Docker resource limits unless explicitly assigned.
- Separate mocked application tests from real model benchmarks.
- Report peak VRAM/RAM and inference latency for model feasibility tasks.

## 9. Git rules

- Inspect `git status` before starting.
- Do not discard, reset, or overwrite uncommitted work.
- Do not create commits unless the assignment explicitly asks for a commit.
- Never force-push.
- Report final `git status` and changed-file list in the handover.

## 10. Handover report template

```text
TASK:
AGENT:
STATUS: Complete / Partial / Blocked

CHANGED FILES:
- path — reason

IMPLEMENTATION:
- What changed

TESTS:
- Command:
- Result:

MANUAL VERIFICATION:
- What was checked

KNOWN LIMITATIONS:
- None / details

OUT-OF-SCOPE CHANGES:
- None / details

GIT STATUS:
- Summary

BLOCKERS / APPROVALS NEEDED:
- None / details

READY FOR REVIEW:
- Yes / No
```

## 11. Stop conditions

Stop work and report to the orchestrator if:

- The task requires an unapproved architecture decision.
- A required file is outside the allowed scope.
- Existing user changes may be overwritten.
- A dependency or model choice materially changes the project.
- Hardware limitations prevent the assigned approach.
- Tests reveal a defect outside the assigned scope.
- The task cannot be completed without changing another agent's work.

Do not solve a stop condition by expanding scope on your own.
