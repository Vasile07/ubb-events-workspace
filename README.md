# UBB Events: AI Workspace

AI-assisted, human-in-the-loop workspace for the UBB student organizations event management project (university project). It organizes the work as **tasks**: each task is one `task.md` file that holds the requirement links, the plan, the test evidence and the human corrections, so the whole process stays traceable and explainable.

The project documentation (requirements, use cases, architecture, ERD, API specification, ADRs) is written manually in Google Drive. This workspace holds the task evidence and the instructions for the agents.

## Structure

```
AGENTS.md                  rules for every agent (CLAUDE.md contains: @AGENTS.md)
context/project-context.md stack, layering rules, conventions, commands
agents/                    planner.md, implementer.md
skills/                    onboarding, task-creation, start-working, create-pr
repos/                     README.md + backend/ and frontend/ (separate git repos, ignored)
changes/
  task-template.md         template for every task
  todo/                    <task-id>-<short-description>.md
  active/                  <task-id>-<short-description>/task.md
  completed/               <task-id>-<short-description>/task.md
.codex/agents/             Codex wrappers (planner.toml, implementer.toml)
```

## First-time setup

1. Clone this workspace.
2. Fill in the repository URLs in `repos/README.md`.
3. Fill in the stack, conventions and commands in `context/project-context.md`.
4. Make the agents and skills visible to your tool (see "Tool setup").
5. Ask the agent: **"run onboarding"**. It clones the backend and frontend into `repos/` and checks your prerequisites.

## Workflow

| Step | Who | What |
|------|-----|------|
| 1 | You + agent | "Create a task for ..." (`task-creation`) produces `changes/todo/<task>.md` |
| 2 | You | "I want to start working on task X" (`start-working`) |
| 3 | Agent | Moves the task to `changes/active/`, checks the repo state, gathers context |
| 4 | Planner | Writes the plan in `task.md` |
| 5 | You | Refine the plan. Corrections you point out are recorded in `Human Corrections` |
| | | **Gate 1: you approve the plan** |
| 6 | Implementer | Creates the branch, writes tests first (red), implements (green), records evidence |
| 7 | You | Review and refine the implementation |
| | | **Gate 2: you approve commit and push** |
| 8 | Implementer | Commits and pushes inside the specific repo |
| 9 | Agent | "Create the PR" (`create-pr`), one PR per repo |
| 10 | You | Review the PR |
| | | **Gate 3: you merge. Agents never merge** |
| 11 | You (or agent on request) | Move the task to `changes/completed/` when the checklist is complete |

## Task types

`workflow`, `artifact`, `research`, `architecture`, `deployment`, `no-ai`.

`no-ai` tasks are for the feature each student implements **without GenAI**. Agents refuse to plan or write code for them.

## Agents and models

The instructions live once, in `agents/<name>.md`. Each tool only adds its own model setting.

| Agent | Role | Claude Code (`model:` in the frontmatter of `agents/<name>.md`) | Codex (`model` in `.codex/agents/<name>.toml`) |
|-------|------|------------------------------------------------------------------|------------------------------------------------|
| planner | Writes and refines the plan | `claude-sonnet-5-5` | `gpt-5.6-luna` |
| implementer | TDD implementation, commit and push | `claude-sonnet-5-5` | `gpt-5.6-luna` |

To change a model, edit only that one line.

## Skills

| Skill | Use it to |
|-------|-----------|
| `onboarding` | One-time setup: clone repos, check prerequisites and GitHub authentication |
| `task-creation` | Create a new task in `changes/todo/` |
| `start-working` | Activate a task, check repos, gather context, launch the planner |
| `create-pr` | Open the PR(s) after the push |

## Tool setup

The tools look for agents and skills in their own folders, so link them to the root folders (check the current documentation of each tool, as paths may change).

- **Claude Code:** `CLAUDE.md` contains `@AGENTS.md`. Link `.claude/agents` to `agents/` and `.claude/skills` to `skills/`, with symlinks (`ln -s ../agents .claude/agents`, `ln -s ../skills .claude/skills`) or by copying. On Windows, symlinks may need developer mode or Git configured for them.
- **Codex:** reads `AGENTS.md` natively. The agent wrappers in `.codex/agents/*.toml` set the model and point to `agents/<name>.md`.

## Rules in short

Full rules are in `AGENTS.md`.

- Three approval gates: plan, push, merge. Silence is not approval.
- Human Corrections are written only when you point out a problem.
- Git operations happen inside the specific repo folder. Branch name: `<task-id>-<short-description>`. Default branch: `master`.
- Never discard or stash changes without your explicit consent.
- Tests come first, and red then green is recorded in the task.
- Agents use only the relevant code and completed tasks as context.
- The workspace itself is under git. Its history is part of the project evidence.