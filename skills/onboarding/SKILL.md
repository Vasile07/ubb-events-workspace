---
name: onboarding
description: One-time setup for a teammate. Clones the backend and frontend repositories into repos/, checks prerequisites and GitHub authentication. Use when the user asks to onboard, set up the workspace, or when repos/ is empty or missing.
---

# Onboarding

Sets up the local workspace for one teammate: clones the project repositories into `repos/` and checks that the tools needed by the other skills are available. It is meant to run once per teammate (and can be re-run safely).

Follow `AGENTS.md` at all times. This skill has no approval gate, but it asks before touching anything that already exists.

## Steps

### 1. Read the repository list

- Read `repos/README.md`. It is the single source of truth for the repository names, URLs and default branch. Do not hardcode URLs anywhere else.
- If the URLs are still placeholders or missing, ask the user for them and tell them to add them to `repos/README.md`.

### 2. Check the prerequisites

Check and report each one as OK or missing, with how to install it. Do not install anything system-wide without asking.

- `git` is installed.
- The runtimes and build tools for the stack, as listed in `context/project-context.md`. If the stack is still `TODO`, report that and skip this check.
- GitHub CLI (`gh`), needed by the `create-pr` skill. If missing, say that PRs will have to be opened manually.

On Windows, use these shell-neutral checks from the workspace environment:

```text
git --version
java -version
gh --version
```

Do not pipe `gh --version` through PowerShell commands such as `Select-Object` when running in Command Prompt. If a tool works in a separate terminal but not in the workspace environment, restart the Codex app so it reloads the system `PATH`.

### 3. Check GitHub authentication

- Verify that git can reach the repositories (for example `git ls-remote <url>`) and that `gh auth status` succeeds.
- Treat `gh --version` as the installation check and `gh auth status` as the separate authentication check. Never record or print the token value.
- If authentication fails, tell the user what to do (log in with `gh auth login`, or set up SSH or a token). Never ask for the user's password or token in the chat, and never store credentials in the workspace.

### 4. Clone the repositories

For each repository listed (backend, frontend), in `repos/<name>/`:

| State of `repos/<name>/` | Action |
|---|---|
| Does not exist or is empty | Clone it from the URL in `repos/README.md`. |
| Is a git repo of the expected remote | Do **not** re-clone. Run `git fetch` and report: current branch, whether the repo is clean, and whether it is behind `master`. Pull only if it is on `master` and clean. |
| Is a git repo with a different remote | **Stop and ask the user.** |
| Contains something else (not a git repo) | **Stop and ask the user.** Never delete or overwrite existing content. |

### 5. Verify the repositories

- Confirm that the default branch of each repo is `master`. If it is not, tell the user, because `AGENTS.md` and `project-context.md` assume `master`.
- Confirm the working tree state of each repo (current branch, clean or not).
- Dependencies: if `project-context.md` contains the install commands, **ask the user** whether to run them. Do not install dependencies silently.

### 6. Check the workspace `.gitignore`

- It must ignore `repos/backend/` and `repos/frontend/`, because they are separate git repositories. `repos/README.md` stays tracked.
- If entries are missing, tell the user and offer to add them. Do not edit `.gitignore` without the user's consent.

### 7. Report

Give a short summary: prerequisites (OK or missing), authentication, state of each repo (cloned, updated, skipped, needs attention), and what the user still has to do. Point to `start-working` for the next step.

## Rules

- Never modify the workspace git history and never commit anything. Do not commit anything inside `repos/`.
- Never delete, overwrite or reset existing folders or repositories.
- Never discard or stash changes, and never switch branches in a repo with uncommitted changes.
- Never store or print credentials.
- Do not create branches or tasks. That is the job of `start-working` and the implementer.
