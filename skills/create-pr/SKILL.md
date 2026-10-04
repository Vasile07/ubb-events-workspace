---
name: create-pr
description: Creates GitHub pull requests for the active task, one per repo, after the branch has been pushed (Gate 2 passed). It may run automatically after the implementer pushes or when the user asks to create or open the PR. Never merges.
---

# Create PR

Opens one pull request per repo touched by the active task and records the links in the task file. It does not review, approve or merge anything.

Follow `AGENTS.md` at all times.

## Preconditions

Read `changes/active/<task-id>-<short-description>.md` and verify:

1. `Gate 2` is checked and `Committed and pushed` is checked. If not, stop and tell the user.
2. The `Test log` has the red and green evidence. If not, tell the user before continuing.

For every repo listed in `Repo(s)` (run inside that repo folder, and state which repo you are acting on):

3. The current branch is the task branch `<task-id>-<short-description>`.
4. The working tree is clean and the branch is pushed (no unpushed commits). If not, stop and ask the user.
5. The branch has commits compared with `master`.

## Steps

1. **Check the tooling.** Verify that `gh` is installed and authenticated (`gh auth status`). If it is not, do not fail: prepare the title and body (step 3) and give the user the details to open the PR manually on GitHub, with the branch name and base branch.

2. **Check for an existing PR** for the branch (for example `gh pr list --head <branch>`). If one exists, report its link and do not create a duplicate.

3. **Prepare the title and body.**
   - **Title:** `<task-id>: <short title>`.
   - **Body**, taken from the task file and the actual diff (do not invent content):
     - Summary of what was built and why.
     - Task reference (id and title), requirement, use case and workflow from the Metadata.
     - Main changes (layers and files, briefly).
     - Tests: the tests written, the red and green results, and the full-suite result from `Test log`.
     - Acceptance criteria as a checklist.
     - Deviations from the plan and known limitations.
     - Link to the PR in the other repo, if the task touches both.
   - Do not paste secrets, local paths or the content of `Human Corrections` unless the user asks.

4. **Create the PR** against `master`, from the task branch, in each repo:
   - Use `gh pr create` with the title and body.
   - If the task touches both repos, create the backend and frontend PRs and add each link to the other PR's body.

5. **Record the result in the task file:**
   - Put the PR link(s) in `PR link(s)` in the Metadata.
   - Check `PR created` in the Status checklist.

6. **Report** the PR link(s) to the user and tell them the next steps: review the PR (the `PR reviewed` item is checked by the user), and merge it themselves (Gate 3).

## Rules

- Never merge, approve, close or force-push. Merging is Gate 3 and belongs to the user.
- Never push to `master`, and never create a PR from `master`.
- Do not commit workspace changes unless the user asks.
- If the PR cannot be created (authentication, permissions, network), explain the cause and give the manual fallback. Do not retry in loops.
- Do not edit the code in this skill. If the review requires changes, go back to the implementer.
