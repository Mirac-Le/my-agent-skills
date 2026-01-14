---
name: git-workflow
description: Git workflow helper for commits, branches, and PR management. Use when creating commits, reviewing staged changes, managing branches, or preparing pull requests. Follows Conventional Commits format.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Git Workflow

## Commit Message Format

Use **Conventional Commits** format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no behavior change |
| `perf` | Performance improvement |
| `test` | Adding/updating tests |
| `chore` | Build, config, dependencies |

### Rules

- Subject: max 72 characters, imperative mood, no period
- Scope: optional, lowercase (e.g., `api`, `db`, `ui`)
- Body: explain *what* and *why*, not *how*
- Footer: reference issues (`Closes #123`)

## Creating Commits

### Step 1: Review Changes

```bash
git status
git diff --staged
```

### Step 2: Analyze and Categorize

- Group related changes by scope
- Identify if changes should be split into multiple commits
- Check for unintended changes

### Step 3: Craft Message

```bash
git commit -m "$(cat <<'EOF'
feat(api): add user authentication endpoint

- Implement JWT token generation
- Add password hashing with bcrypt
- Create login/logout handlers

Closes #42
EOF
)"
```

## Branch Naming

```
<type>/<ticket>-<description>
```

Examples:
- `feat/PROJ-123-user-auth`
- `fix/PROJ-456-login-bug`
- `chore/update-deps`

## Pull Request Workflow

### Before Creating PR

```bash
# Ensure up to date with main
git fetch origin
git rebase origin/main

# Check all commits
git log origin/main..HEAD --oneline
```

### PR Description Template

```markdown
## Summary
- Brief description of changes

## Changes
- [ ] Change 1
- [ ] Change 2

## Test Plan
- How to verify the changes work

## Related Issues
Closes #XX
```

## Common Operations

### Amend Last Commit (unpushed only)

```bash
git add .
git commit --amend --no-edit
```

### Interactive Rebase (clean up commits)

```bash
git rebase -i HEAD~3
```

### Stash Changes

```bash
git stash push -m "WIP: description"
git stash pop
```

## Safety Rules

- **NEVER** force push to `main`/`master`
- **NEVER** amend pushed commits without team agreement
- **ALWAYS** verify branch before destructive operations
- **ALWAYS** use `--dry-run` for risky commands first
