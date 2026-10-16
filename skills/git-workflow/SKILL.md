---
name: git-workflow
description: Master Git workflows including commit messages, branching strategies, rebasing, bisecting, and PR hygiene. Use when writing commits, resolving merge conflicts, creating branches, cleaning up history, debugging with git bisect, or preparing pull requests. Trigger on "git", "commit", "merge conflict", "rebase", "branch", "PR", "pull request", "git bisect", or "commit message".
---

# Git Workflow

Clean history, clear messages, safe collaboration.

## Commit Messages (Conventional Commits)

```
<type>(<scope>): <description>

<body: why, not what>

<footer: breaking changes, issue refs>
```

Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`

Good:
```
feat(auth): add OAuth2 login with Google

Users can now sign in with Google without creating a password.
Implements the flow described in RFC 6749 section 4.1.

Closes #142
```

Bad:
```
fixed stuff
updates
WIP
```

### Rules
- Imperative mood: "add feature" not "added feature"
- First line ≤ 72 characters
- Body explains *why*, wraps at 72 chars
- Breaking changes: `BREAKING CHANGE:` in footer or `!` after type

## Branching Strategies

### Trunk-Based (recommended for small teams)
```
main ──A──B──C──D──E  (always deployable)
              \/
feature ──────F──G──── (short-lived, merge daily)
```

### Git Flow (for scheduled releases)
```
main ───────M────────── (production)
develop ──D──E──F──G── (integration)
           \/
release ───R1──R2───── (stabilization)
```

## Rebase vs Merge

| Use Case | Command |
|----------|---------|
| Update feature branch from main | `git rebase main` |
| Combine commits before PR | `git rebase -i HEAD~3` |
| Merge feature into main | `git merge --no-ff feature` |
| Share work already pushed | `git merge` (don't rebase shared branches) |

### Interactive Rebase (Clean History)
```bash
git rebase -i HEAD~5
# Pick = keep commit
# Squash = merge into previous (keep message)
# Fixup = merge into previous (discard message)
# Drop = remove commit
```

## Merge Conflict Resolution

```bash
git merge feature-branch
# Conflicts appear in files with <<<<<<< ======= >>>>>>>

# 1. Understand both changes
git log --oneline feature-branch  # what they did
git log --oneline main            # what changed here

# 2. Resolve by hand — keep both intents, not just both lines
# 3. Test the resolution
# 4. Mark resolved
git add resolved-file.js
git merge --continue
```

**Principle**: Resolve conflicts by understanding intent, not just picking sides.

## Debugging with Git

```bash
# Find the commit that broke something
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Test at each step: git bisect good / git bisect bad
# Automate: git bisect run npm test

# Find what changed a line (blame)
git blame -L 100,110 file.js

# See what a branch has that another doesn't
git log main..feature --oneline
git diff main...feature  # changes on feature since diverging
```

## Stashing

```bash
git stash save "WIP: fix login bug"
git stash pop           # restore and drop
git stash apply         # restore but keep
git stash list
git stash pop stash@{2}  # restore specific stash
```

## PR Hygiene

- Small PRs (< 400 lines changed when possible)
- One logical change per PR
- Description explains **why** and links to issue
- Self-review before requesting review
- Keep branch updated with rebase (don't merge main into feature repeatedly)
- Delete branches after merge

## Dangerous Commands (Know Before Using)

```bash
git commit --amend        # Rewrites last commit — don't use on pushed commits
git push --force-with-lease  # Safer than --force, but still dangerous
git reset --hard          # Destroys uncommitted work
git clean -fd             # Deletes untracked files
```

## Checklist Before PR

- [ ] Commit messages follow conventional format
- [ ] History is clean (squashed WIP commits)
- [ ] No merge conflicts with main
- [ ] Tests pass locally
- [ ] No secrets or debug code in diff
- [ ] Branch is up to date with main
