---
name: start-working
description: >
  Starts or resumes work on an existing task from changes/todo/ or changes/active/.
  Use when the user says “I want to start working on task X”, “start task X”,
  “begin task X”, “resume task X”, or “continue task X”. Activates the task,
  checks prerequisites and repository state, then prepares the plan. Stops at Gate 1.
---

# Start working

Starts the workflow for an existing task: activates it, gathers the relevant context and launches the planner. It stops when the plan is ready for the user's review.

Follow `AGENTS.md` at all times.

## Steps

### 1. Identify the task

- The user refers to the task by ID or name (e.g. `T12`, `event registration`).
- Look in `changes/todo/` first, then `changes/active/`.
  - Found in `todo/`: continue with step 2.
  - Found in `active/`: the task is already started. Tell the user, read its task file, check its Status checklist, and resume from the first unchecked step instead of restarting.
  - Found in `completed/`: tell the user and stop. Do not reopen it.
  - Several candidates or none: list them and ask the user which one. Do not guess.

### 2. Read the task and check its type (before anything else)

- Read the task file fully.
- If `Type: no-ai`: **stop**. Tell the user the task is reserved for manual implementation, and that agents will not plan or write code for it. Do not move the file, do not launch any agent. Offer only non-generative help if the user asks for it explicitly.
- If the `Type` is empty, ask the user to set it before continuing.

### 3. Activate the task

- Move the task file to `changes/active/<task-id>-<short-description>.md`. Use `git mv` if the file is tracked by the workspace repo, otherwise a normal move. Do not create a task folder or rename the file.
- Check the first item of the Status checklist when done.

### 4. Check the preconditions

- For tasks that touch code, check that the needed repos exist in `repos/backend/` and `repos/frontend/`. If one is missing, tell the user to run the `onboarding` skill and stop.
- Check that `Repo(s)` and `Branch` in the Metadata are set. The branch name is `<task-id>-<short-description>`. Ask the user only for what is missing.
- If key values in `context/project-context.md` that the task depends on are still `TODO` (stack, run/test commands), tell the user. Do not guess them.
- Do **not** create branches here. The implementer creates the branch after Gate 1.

**Check the state of every repo the task touches** (inside the repo folder, state which repo you are checking). The default branch is `master`; read it from `context/project-context.md`. Run `git status` and check the current branch and any unpushed commits.

| Repo state | Action |
|---|---|
| On `master`, clean | Fetch and pull `master`, continue. |
| On another branch, clean, nothing unpushed | Switch to `master`, fetch and pull, continue. |
| On this task's own branch (resuming) | Stay on it. Report whether it is behind `master`. |
| Uncommitted changes (tracked or untracked) | **Stop and ask the user.** Show `git status` and the options below. |
| Unpushed commits on another branch | **Stop and ask the user** (push, keep, or leave as is). Never switch away silently. |

Options to offer for uncommitted changes (recommend one, the user chooses):

- **Commit** them on their current branch. Usually the right choice if they belong to another task.
- **Stash** them with a label (e.g. `stash before <task-id>`) and tell the user the stash exists. Only with the user's consent.
- **Discard** them. Destructive and irreversible: only after the user has seen exactly what would be lost and explicitly confirms.
- **Leave it:** the user cleans up manually and re-runs the skill.

Never discard or stash on your own initiative.

### 5. Gather the relevant context

Only what is relevant, as `AGENTS.md` requires:

- `context/project-context.md`.
- The code in `repos/` related to the task (entities, services, controllers, components, tests the task will touch or follow as a pattern).
- Completed tasks in `changes/completed/` that relate to the same workflow, requirement or code area.

Do not scan unrelated tasks, folders or the whole repo.

Write the result in the `Context` section of the task file: relevant code (paths), related completed tasks, constraints and assumptions. Keep it short and factual. List open questions you found and ask the user about them.

### 6. Launch the planner

- Hand off to the planner agent (`agents/planner.md`) with: the path of the task file, the type, and the gathered context.
- The planner writes the plan in the `Plan` section of the task file and updates the Status checklist.

### 7. Stop at Gate 1

- Tell the user the plan is ready in the task file and ask them to review and refine it.
- User refinements go into the plan. When the user points out a problem, record it in `Human Corrections` as `AGENTS.md` describes.
- **Do not start the implementation.** It begins only after the user explicitly approves the plan, and the Gate 1 item is checked in the Status checklist.

## Rules

- Never start implementation from this skill.
- Never skip the `no-ai` check.
- Never overwrite existing content in the task file, only fill the sections this skill is responsible for (`Context`, status items).
- Never commit, push or create branches.
- If anything about the task is unclear or contradictory, ask the user before launching the planner.
