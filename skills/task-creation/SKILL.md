---
name: task-creation
description: Creates a new task file in changes/todo/ from changes/task-template.md. Use when the user wants to add, create, define or write down a new task (e.g. "create a task for event registration", "add a task for the ERD"). Does not start working on the task.
---

# Task creation

Creates a task in `changes/todo/` so it can be started later with the `start-working` skill. This skill only **defines** the task. It never plans or implements anything.

Follow `AGENTS.md` at all times.

## Inputs

The user describes the task in free text. Possible information: what to do, why, related requirement or use case, type, owner, repo(s), acceptance criteria.

## Steps

1. **Read** `changes/task-template.md` and `context/project-context.md` (for the task ID and branch conventions).

2. **Determine the next task ID.** List the files and folders in `changes/todo/`, `changes/active/` and `changes/completed/`, find the highest existing ID, and use the next one, following the ID format in `project-context.md` (e.g. `T12`). If no tasks exist yet, start from `T1`. Never reuse an ID.

3. **Determine the short description.** Lowercase, words separated by hyphens, 2 to 5 words (e.g. `event-registration`). The file name is `<task-id>-<short-description>.md`, so the branch will be `<task-id>-<short-description>`.

4. **Fill in what the user provided, and only that.**
   - `Metadata`: Task ID, Type, Owner, Requirement(s), Use case(s), Workflow, Drive doc / ADR, Repo(s), Branch (derived from the ID and short description).
   - `Description`: the user's intent in clear wording, including the user need it addresses.
   - `Acceptance criteria`: verifiable, testable statements. If the user gave none, **propose** a short list and clearly mark it for the user to confirm or edit.
   - Leave every unknown value empty. Never invent requirements, use cases, owners or links.

5. **Ask about missing essentials in one short message** (not one question at a time). The essentials are:
   - Type: `workflow`, `artifact`, `research`, `architecture`, `deployment` or `no-ai`.
   - Owner.
   - Related requirement or use case, if any.
   - Repo(s) involved, if the task touches code.

   If the user prefers to skip these, create the file anyway with the fields empty.

6. **`no-ai` tasks:** set `Type: no-ai` exactly. Note in the `Description` that the task is reserved for manual implementation. Do not propose code, an approach or a design for it.

7. **Create the file** at `changes/todo/<task-id>-<short-description>.md`, keeping every heading of the template unchanged and in the same order. Replace the title line with `# <task-id> - <Short title>`. Sections not yet relevant (`Context`, `Plan`, `Test log`, `Human Corrections`, `Design decisions, alternatives, limitations`) stay as in the template.

8. **Report** to the user: the file path, the ID, the branch name that will be used, and which metadata fields are still empty. Ask whether to adjust anything.

## Rules

- Do not create the task folder in `changes/active/`, do not create branches, and do not read the repos. That is the job of `start-working`.
- Do not write a plan, steps or an approach in the `Plan` section.
- Do not create several tasks from one request unless the user explicitly asks. If the request looks too big, suggest a split and let the user decide.
- Do not edit or move existing tasks.
- Do not commit anything unless the user asks.