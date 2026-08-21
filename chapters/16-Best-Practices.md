# Chapter 16 — Git Best Practices

## Introduction

Git is a highly flexible tool. It allows you to write commits as large or small as you want, name branches whatever you like, and merge code using various methods. However, in professional software engineering, this absolute freedom can quickly lead to chaos. Without shared guidelines, a team’s codebase will accumulate messy commits, confusing branch lines, and hard-to-track bugs.

**Git Best Practices** are the industry-standard conventions that keep repositories clean, history reviewable, and deployments stable. Following these rules is what separates junior developers from senior engineers.

This chapter details the core guidelines for writing atomic commits, formatting professional commit messages (Conventional Commits), naming branches, managing Pull Requests, and organizing repository files.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Apply the concept of "Atomic Commits" to keep changes focused and easy to review
- Format commit messages using the universally accepted Conventional Commits standard
- Implement a structured branch naming convention using prefixes and issue tracking
- Optimize the Pull Request (PR) lifecycle for faster reviews and integrations
- Organize your repository directories to keep dependencies and build outputs out of history

---

## 1. The Core Principle: Atomic Commits

An **Atomic Commit** is a commit that does **one single, focused thing**. It contains all the changes related to a single logical task, and nothing else.

```
       [ Non-Atomic Commit (Bad) ]                   [ Atomic Commits (Good) ]
  "Combined feature, refactor, and fix"           "Three small, focused commits"

  ┌─────────────────────────────────────┐         ┌─────────────────────────────────────┐
  │ - Add search bar UI                 │         │ Commit 1: Add search bar UI         │
  │ - Fix typo in config.json           │         ├─────────────────────────────────────┤
  │ - Refactor database helper class    │         │ Commit 2: Fix config.json typo      │
  └─────────────────────────────────────┘         ├─────────────────────────────────────┤
                                                  │ Commit 3: Refactor database helper  │
                                                  └─────────────────────────────────────┘
```

### Why Atomic Commits Matter:
* **Easier Code Review:** Reviewers can inspect your changes line-by-line and understand the purpose of each change.
* **Safe Reverts:** If Commit 3 (refactoring) introduces a bug but Commit 1 (search bar) is correct, you can revert Commit 3 without losing the search bar UI.
* **Clean History:** When inspecting history via `git log`, the commit list reads like a clear, step-by-step changelog.

---

## 2. Conventional Commits Standard

To make commit histories machine-readable and easy for humans to scan, professional teams follow the **Conventional Commits** specification.

The format is:
```text
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### A. Commit Types (`<type>`)
* **`feat`:** A new feature for the user (e.g., `feat(auth): add email verification`).
* **`fix`:** A bug fix (e.g., `fix(database): resolve connection timeout`).
* **`docs`:** Documentation changes only (e.g., `docs(readme): update setup guide`).
* **`style`:** Formatting, semicolons, white-space changes (no production code changes).
* **`refactor`:** Code changes that neither fix a bug nor add a feature (e.g., restructuring database helpers).
* **`test`:** Adding missing tests or correcting existing tests.
* **`chore`:** Updating build tasks, package manager dependencies, or configurations (e.g., `chore(deps): bump lodash from 4.17.15 to 4.17.21`).

### B. The Description
* Use the **imperative mood** (e.g., "add profile validation" instead of "added profile validation" or "adds profile validation"). Think of it as completing the sentence: *"If applied, this commit will..."*
* Do not capitalize the first letter.
* Do not end the description with a period.

### Conventional Commit Example:
```text
feat(payments): integrate Stripe API checkout portal

Setup Stripe SDK elements on checkout component and link checkout button
to direct user to payments gateway portal.

Closes JIRA-402
```

---

## 3. Branch Naming Conventions

Branch names should be structured so that any developer can instantly understand the purpose of the branch and locate the corresponding project management ticket.

### Recommended Prefix Structure:
Use forward slashes (`/`) to categorize your branches:

* **`feat/` or `feature/`:** New feature developments (e.g., `feat/login-page`).
* **`fix/` or `bugfix/`:** Bug fixes (e.g., `fix/navbar-overlap`).
* **`hotfix/`:** Emergency patches for production bugs.
* **`docs/`:** Adding or updating documentation.
* **`refactor/`:** Reorganizing file directory layouts or cleaning code.

### Integrating Ticket IDs:
If your team uses project management tools (like Jira, Trello, or GitHub Issues), include the ticket identifier at the beginning of the branch name:

```text
feature/JIRA-402-stripe-checkout
bugfix/GH-18-resolve-auth-crash
```

---

## 4. Pull Request (PR) Etiquette

The Pull Request is where team coordination happens. Managing it efficiently keeps your development speed high.

* **Keep PRs Small:** Code reviews are most effective when they are small. Aim for PRs under **400 lines of code**. If a feature is large, break it down into smaller, incremental PRs.
* **Write a Detailed Description:** Don't leave the PR description blank. Use a template that explains *what* changed, *why* it was done, and *how* to test it.
* **Self-Review First:** Before assigning reviewers, open your PR’s diff tab on GitHub and read through your changes. You will catch 90% of minor typos, formatting issues, and forgotten print statements before your colleagues see them.
* **Delete Branches Immediately:** Once a PR is merged, delete the feature branch both on GitHub and locally (`git branch -d <name>`) to prevent repository clutter.

---

## 5. Repository Housekeeping

Keep your repository directories organized to prevent performance degradation and merge conflicts:

1. **Keep Secrets Out of History:** Never commit `.env` files, SSH keys, or passwords. Add them to `.gitignore` immediately. If a secret is committed, revoke it immediately (deleting the commit from history is not enough; the secret must be treated as compromised).
2. **Do Not Commit Dependencies:** Never commit folders like `node_modules/` or Python `venv/` to Git. Dependencies should be installed from manifest files (like `package.json` or `requirements.txt`).
3. **Use LFS for Large Binary Files:** Git is optimized for text files. Storing large binary files (like raw images, videos, audio, or compiled `.jar`/`.exe` assets) will bloat the `.git` directory, making clone times extremely slow. Use **Git Large File Storage (LFS)** for these assets.

---

## Best Practices Command Summary

```bash
# Keep local main clean (Always use branches)
git switch main
git pull origin main

# Standard branch creation
git switch -c feat/JIRA-12-auth-recovery

# Conventional commit example
git commit -m "fix(auth): correct password reset validation logic"

# Clean up local branches after merge
git branch -d feat/JIRA-12-auth-recovery
git remote prune origin
```

---

## What Comes Next

Following Git best practices establishes a standard of excellence inside your development team.

When working on open-source software or contributing to repositories where you do not have direct push permissions, the collaboration workflow changes. In **Chapter 17 — Open Source Workflow**, we will explore forks, cloning forks, upstream remotes, syncing histories, and contributing to public codebases.
