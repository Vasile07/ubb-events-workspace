# Repositories

Single source of truth for the project repositories. The `onboarding` skill reads this file to clone them. Do not duplicate these URLs in other files.

## Repository list

| Name | Local path | URL | Default branch |
|------|------------|-----|----------------|
| ubb-events-backend | `repos/ubb-events-backend/` | https://github.com/Vasile07/ubb-events-backend.git | `master` |
| ub-events-frontend | `repos/ubb-events-frontend/` | https://github.com/Vasile07/ubb-events-frontend.git | `master` |

## Setup

Ask your agent to run the `onboarding` skill (for example: "run onboarding"). It will:

1. check the prerequisites (git, the stack tools, GitHub CLI);
2. check your GitHub authentication;
3. clone both repositories into this folder, or fetch and report if they already exist.

Manual alternative, from the workspace root:

```
git clone <backend-url> repos/ubb-events-backend
git clone <frontend-url> repos/ubb-events-frontend
```

## Rules

- `repos/ubb-events-backend/` and `repos/ubb-events-frontend/` are **independent git repositories**. They are listed in the workspace `.gitignore` and are never committed to the workspace repo. This `README.md` is tracked.
- Run every git operation (branch, commit, push, PR) **inside the specific repo folder**, not at the workspace root.
- Branches are named `<task-id>-<short-description>`. Never commit directly to `master`.
- Never commit secrets or `.env` files in any repo.

## Workspace `.gitignore` entries

```
repos/ubb-events-backend/
repos/ubb-events-frontend/
```