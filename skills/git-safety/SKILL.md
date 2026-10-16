---
name: git-safety
description: "Use before any commit or push to keep secrets and junk out of Git history."
---

# git-safety Guidelines

Before the first commit, confirm .gitignore exists and covers node_modules, .env, build output, and IDE folders. Never commit .env files, API keys, or database dumps with real data. Provide a .env.example instead. Never force push to a shared branch. Check git status before committing and keep each commit to one logical change.
