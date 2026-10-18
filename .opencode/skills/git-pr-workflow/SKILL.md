---
name: git-pr-workflow
description: Git workflow for this repository. Create a feature branch from master, commit with English messages, push the branch, and open a GitHub Pull Request with gh. Use when the user asks to commit, push, create a branch, or open a pull request. Never push to master.
---

# Git PR Workflow

Base branch: `master`. Remote: `origin` (GitHub).

## Hard rules

- Never run `git push` to `master`, and never push directly to `master`.
- Always work in a feature branch and open a Pull Request.
- Ask for approval before every git command that changes state (`checkout -b`, `add`, `commit`, `push`, `gh pr create`).
- Never commit secrets: `*.pem`, keys, tokens, passwords.

## Steps

1. Read-only checks (no approval needed):
   ```bash
   git status
   git diff
   git log --oneline -10
   git remote -v
   ```
2. Show the planned branch name and list of files. Wait for approval.
3. Create a branch from up-to-date `master`:
   ```bash
   git checkout -b feature/<short-english-description>
   ```
4. Stage only the intended files by name (avoid `git add -A`).
5. Commit with an English message in imperative mood, e.g. `Add base packages role`.
6. Show the commit and wait for approval, then push the branch:
   ```bash
   git push -u origin feature/<short-english-description>
   ```
7. Open the PR after approval:
   ```bash
   gh pr create --base master --title "<English title>" --body "<English description>"
   ```
8. Return the PR URL to the user.

## Before creating a PR

- Review all commits on the branch, not only the last one: `git log master..HEAD --oneline`.
- Confirm `ansible-playbook site.yml --syntax-check` passes.
