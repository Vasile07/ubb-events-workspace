# AGENTS.md

Root instructions for every AI agent working in this workspace. They apply to all agents and skills. If an agent or skill file conflicts with this file, this file wins.

## 1. Purpose

This workspace supports a team building an enterprise event management system for UBB student organizations (university project). GenAI use is part of what is graded, so the process must stay **human-in-the-loop, traceable and explainable**. The student is responsible for everything produced and must be able to explain and defend it.

Agents assist. They do not make decisions that belong to the user.

## Mandatory workflow routing

Before taking action, classify the user's request.

- If the user says they want to start, begin, resume, continue, or work on a task identified by an ID or name, read and follow `skills/start-working/SKILL.md`.
- If the user asks to create, define, or add a task, read and follow `skills/task-creation/SKILL.md`.
- If the user asks to onboard, set up, clone, or check prerequisites, read and follow `skills/onboarding/SKILL.md`.
- If the user asks to open or create a pull request, read and follow `skills/create-pr/SKILL.md`.

These are mandatory routes, not suggestions. Do not improvise an equivalent workflow. For example, “I want to start working on task T12” must use `skills/start-working/SKILL.md` before any task or repository changes are made.

## 2. Workspace layout

```
agents/      planner.md, implementer.md
skills/      onboarding, task-creation, start-working, create-pr
context/     project-context.md (stack, layering rules, naming, branch convention, repo paths)
repos/       backend/ and frontend/ (separate git repos, cloned here, not tracked by the workspace)
changes/     task-template.md
  todo/        <task-id>-<short-description>.md
  active/      <task-id>-<short-description>.md
  completed/   <task-id>-<short-description>.md
```

- Full documentation lives in Google Drive, written manually by the team. Agents do **not** create or edit project documentation.
- Agents get context only from: `context/project-context.md`, the active task, the relevant code in `repos/`, and tasks in `changes/completed/`. Do not scan unrelated tasks or folders.

## Skills and agents

Each skill is defined in `skills/<name>/SKILL.md`. Read the file and follow it when the user's request matches.

| Skill | Use when the user wants to... |
|-------|-------------------------------|
| `onboarding` | set up the workspace, or `repos/` is empty (clones the repos, checks prerequisites) |
| `task-creation` | create or define a new task (creates a file in `changes/todo/`) |
| `start-working` | start or resume a task, e.g. "start working on task T12" (activates it, checks repos, launches the planner) |
| `create-pr` | open the PR(s) for a task after the push, either automatically after Gate 2 or when requested separately |

| Agent | Role |
|-------|------|
| `planner` (`agents/planner.md`) | writes and refines the `Plan` section; launched by `start-working` |
| `implementer` (`agents/implementer.md`) | TDD implementation after Gate 1; commits and pushes after Gate 2 |

If a request matches a skill, use the skill instead of improvising the same steps.

## 4. Task files

- Every task is a single Markdown task file based on `changes/task-template.md`. It holds metadata, plan, test log, human corrections, design notes and the status checklist.
- **Never rename, remove or reorder the section headings** of the template. Agents locate sections by heading.
- Keep the metadata (requirement, use case, workflow, owner, type, Drive/ADR link, branch) filled in as information becomes available. Do not invent values. If a value is unknown, ask the user.
- Task file and branch names follow `<task-id>-<short-description>`; task files use the `.md` extension.

## 5. Task types

`workflow`, `artifact`, `research`, `architecture`, `deployment`, `no-ai`.

- **`no-ai` tasks:** agents must **not** write, generate, modify or suggest code for the task. Before doing anything, `start-working` checks the `type` field. If it is `no-ai`, stop and tell the user the task is reserved for manual implementation. Agents may only act when the user explicitly asks for something non-generative, such as answering a conceptual question or reviewing a design after the user has finished.
- **`research`, `architecture`, `deployment`:** follow the same flow (plan, gates, corrections). The output may be notes, a decision, configuration or scripts instead of application code. Record the result in the task and point to the Drive document or ADR.

## 6. Workflow

1. User asks to start a task. `start-working` moves the task file to `changes/active/<task-id>-<short-description>.md`.
2. Agent reads the task and gathers only the relevant context.
3. **Planner** writes the plan into the `Plan` section of the task file.
4. User refines the plan. Changes stay visible in the task file.
5. **GATE 1: user approves the plan.**
6. **Implementer** implements following the TDD rules below.
7. User reviews and refines the implementation.
8. **GATE 2: user approves before commit and push.**
9. After Gate 2 approval, the agent commits and pushes (inside the specific repo), then runs the `create-pr` workflow automatically if GitHub CLI is installed and authenticated.
10. If GitHub CLI is unavailable or unauthenticated, the agent stops after pushing and gives the user the manual PR details. The user may also request the `create-pr` skill separately later.
11. PR review happens.
12. **GATE 3: the user merges. Agents never merge.**
13. Task is moved to `changes/completed/` (see section 9).

Never skip or merge gates. Do not proceed past a gate without an explicit approval message from the user in the current conversation. Silence or a previous approval of a different step is not approval.

## 7. Implementation rules (TDD)

The implementer follows this order and records it in the `Test log` section:

1. Write the tests first, plus the minimum code (stubs) needed to compile.
2. Run the tests and confirm they **fail**. Record which tests and why they failed.
3. Implement the functionality.
4. Run the tests and confirm they **pass**. Record the result.
5. Run the wider test suite of the affected repo to check for regressions.

Also:

- Implement only what the approved plan and acceptance criteria describe. If something outside the plan seems necessary, stop and ask.
- Handle errors and enforce business rules explicitly (validation, capacity, registration status, permissions).
- Follow the layering, naming and conventions in `context/project-context.md`.
- Never claim tests pass without having run them.

## 8. Human Corrections

The `Human Corrections` section is evidence of human oversight.

- Agents write an entry **only when the user points out a problem or asks for a change** to the plan or implementation, and only in the user's terms: what was wrong, what was changed or rejected, and why.
- Agents must **not** add entries on their own initiative, and must not invent or soften corrections.
- Planner and implementer record the entry as soon as the user gives the correction, then apply it.

## 9. Git and repositories

- `repos/ubb-events-backend` and `repos/ubb-events-frontend` are independent git repos. Run every git operation (branch, commit, push, PR) **inside the specific repo folder**, and state which repo is being used.
- A task touching both repos uses the same branch name in each, with one PR per repo.
- Branch name: `<task-id>-<short-description>`. Never commit directly to the default branch.
- Small, focused commits with clear messages. Never force-push, rewrite published history, or merge.
- Default branch: `master`.
- **Repository state:** before planning (`start-working`) and again before creating the task branch (implementer, after Gate 1), check the repo with `git status`. Only fetch, pull and switch to `master` when the repo is clean with nothing unpushed. If there are uncommitted changes or unpushed commits, stop and ask the user. Never discard or stash changes without the user's explicit consent, and never discard without showing what would be lost.
- The implementer creates the task branch from an up-to-date `master` after Gate 1. Planning never creates branches.
- Never commit secrets, credentials or `.env` files.
- Do not commit anything inside `repos/` from the workspace root, and do not modify the workspace git history unless the user asks.

## 10. Completing a task

The user (or an agent on explicit user request) moves the task file from `changes/active/` to `changes/completed/` only when:

- the PR is merged (Gate 3 passed), with PR links recorded in the task;
- the `Test log` has the red and green evidence;
- the `Human Corrections` section is reviewed by the user (empty is fine if there were no corrections);
- `Design decisions, alternatives, limitations` is filled in, so the user can use it as a study note for the defense;
- the metadata (requirement, use case, tests, Drive/ADR link) is complete for traceability;
- the status checklist is up to date.

If something is missing, list what is missing instead of moving the task.

## 11. General behavior

- Ask when requirements are ambiguous. Do not invent requirements, business rules or user needs. Functionality must be justified by identified user needs.
- Prefer the simplest solution that satisfies the acceptance criteria.
- Be explicit about limitations and risks. Do not hide uncertainty.
- Keep responses short and concrete. Reference files and sections instead of repeating their content.
