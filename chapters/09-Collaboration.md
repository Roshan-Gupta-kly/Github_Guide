# Chapter 09 — Collaboration

## Introduction

So far, we have focused on local operations and simple synchronization commands. However, Git's true power is unlocked when engineers work in teams. Collaboration requires more than just knowing how to run `git push`. It demands a structured process to ensure code quality, prevent teammates from overwriting each other's work, and deliver stable software to production.

In professional software development, teams use collaboration platforms like **GitHub**, **GitLab**, or **Bitbucket** to manage integration. This chapter details the professional team workflow, exploring feature branches, Pull Requests (PRs), code review mechanics, branch protection rules, and the management of release and hotfix branches.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Describe the complete lifecycle of the Feature Branch Workflow
- Open, document, and manage Pull Requests (PRs) / Merge Requests (MRs)
- Conduct constructive code reviews and distinguish between blocking and non-blocking feedback
- Compare the three merge strategies on GitHub: Merge Commits, Squash and Merge, and Rebase and Merge
- Configure Branch Protection Rules to secure critical branches from direct pushes or untested code
- Create and manage Release Branches to stabilize scheduled software versions
- Scaffold and merge Hotfix Branches to patch critical production bugs

---

## The Feature Branch Workflow

The **Feature Branch Workflow** is the industry standard for team collaboration. Instead of having everyone push changes directly to the `main` branch, all feature development is isolated in dedicated branches.

```
                   Collaborator A (Feature branch)
                   ┌───[Commit]───[Commit]───┐
                  /                           ▼ (Pull Request)
  main ──────────┴────────────────────────────*───► (Production)
                  \                           ▲ (Pull Request)
                   └───[Commit]───[Commit]───┘
                   Collaborator B (Feature branch)
```

### The Lifecycle of a Feature:
1. **Prepare:** Update your local `main` branch to match the remote (`git pull origin main`).
2. **Branch:** Create a new branch with a descriptive prefix (e.g., `git switch -c feature/add-payment-portal`).
3. **Develop:** Write code, build, and test locally. Make small, focused commits.
4. **Push:** Push your feature branch to the remote server (`git push -u origin feature/add-payment-portal`).
5. **Review:** Open a Pull Request on GitHub to propose merging your changes into `main`.
6. **Integrate:** Once review comments are addressed and automated checks pass, merge the PR and delete the remote and local feature branches.

---

## 1. Pull Requests and Merge Requests

A **Pull Request (PR)** on GitHub (called a **Merge Request (MR)** on GitLab) is a user interface where developers can compare two branches, discuss changes, review code line-by-line, and run automated testing suites.

### How to Create a Pull Request:
1. Push your branch to GitHub:
   ```bash
   git push -u origin feature/add-payment-portal
   ```
2. Navigate to your repository page on GitHub. You will see a banner saying *"Compare & pull request"*. Click it.
3. Write a clear description:
   * **What was done:** Explain the changes.
   * **Why it was done:** Provide business context or reference an issue number (e.g., `Closes #142`).
   * **How to test:** Detail manual verification steps.
4. Select reviewers and submit.

### Draft Pull Requests
If your feature is not finished, but you want feedback on your architecture or want to run automated CI builds early, choose **Create draft pull request** from the dropdown menu. A draft PR cannot be merged until you mark it as *"Ready for review"*, which prevents unfinished code from being accidentally integrated.

---

## 2. Professional Code Review

Code reviews are a quality-assurance gate. They are not about finding fault, but about improving code quality, sharing knowledge, and catching bugs early.

### For Authors:
* **Self-Review First:** Review your own diff before assigning reviewers. You will often spot typos, left-over debug logs, or missing comments.
* **Keep PRs Small:** Reviewers are much better at finding bugs in a 100-line PR than a 1,000-line PR. Aim to keep PRs focused on a single logical task.
* **Be Receptive:** Treat review comments as constructive opportunities to improve the codebase.

### For Reviewers:
* **Be Kind and Respectful:** Focus on the code, not the person. Use phrases like *"Should we name this variable..."* instead of *"You named this variable incorrectly."*
* **Separate Blockers from Suggestions:** Clearly state if a comment is blocking (needs to be fixed before merging) or non-blocking (nice-to-have, but mergeable). Use conventions like `NIT: (nitpick)` for minor styling fixes.
* **Verify Correctness:** Don't just look for typos. Check logic, edge cases, error handling, and test coverage.

---

## 3. GitHub Merge Options

When merging a Pull Request, GitHub offers three integration options. Teams should choose one standard method to keep their repository history consistent.

```
Option A: Create a Merge Commit (Standard Merge)
  - Commits from the feature branch are integrated with a new merge commit.
  - Preserves the exact sequence and context of commits, but can look messy.

Option B: Squash and Merge (Highly Recommended for Features)
  - Combines all feature commits into a single commit on the target branch.
  - Cleans up "fix typo" or "wip" commits, maintaining a simple, linear `main` history.

Option C: Rebase and Merge
  - Commits from the feature branch are reapplied individually on top of `main`.
  - Maintains individual commits while keeping a clean, linear history.
```

---

## 4. Branch Protection Rules

To prevent accidental code loss or unstable deployments, collaboration hosts allow you to lock critical branches (like `main`) using **Branch Protection Rules**.

### Recommended Configurations for `main`:
1. **Require a Pull Request before merging:** Prevents anyone (including administrators) from running `git push origin main` directly from their terminal. All changes must go through a PR.
2. **Require Approvals:** Specifies that a PR must receive at least 1 or 2 approvals from designated code owners before it can be merged.
3. **Require Status Checks to Pass:** Integrates with Continuous Integration (CI) systems. The PR cannot be merged unless automated tests (e.g., GitHub Actions, Jenkins) pass successfully.
4. **Require Signed Commits:** Blocks pushes of commits that do not have verified GPG/SSH signatures, ensuring author authenticity.

---

## 5. Release and Hotfix Branches

In versioned software environments (where you release specific versions like `v1.2.0` to users), teams use dedicated branching paths to stabilize code and patch bugs.

### A. Release Branches (`release/*`)
When the `develop` branch has accumulated enough features for a scheduled release, you fork a release branch:

```bash
git switch develop
git switch -c release/v1.2.0
```

* **Purpose:** A stabilization buffer.
* **What goes here:** Only bug fixes, documentation edits, and release-metadata changes (like bumping version numbers). No new features.
* **Lifecycle:** Once the code is stable and QA-tested, the release branch is merged into `main` (and tagged `v1.2.0`) and also merged back into `develop` so feature branches get the bug fixes.

### B. Hotfix Branches (`hotfix/*`)
If a critical security exploit or crash is discovered in production, you cannot wait for the next scheduled release. You must fix it immediately using a hotfix branch:

```bash
git switch main
git switch -c hotfix/patch-login-bypass
```

* **Purpose:** Emergency patches.
* **What goes here:** The minimal amount of code needed to resolve the critical bug.
* **Lifecycle:** The hotfix is branched directly from the production state on `main`. Once resolved and tested, it is merged immediately into both `main` (and tagged e.g., `v1.2.1`) and `develop` to ensure the patch is not lost in future work.

---

## Collaboration Best Practices

- **Never Commit directly to `main`:** Set up branch protection rules immediately on any new repository.
- **Merge PRs Promptly:** Once a PR is approved and checks pass, merge it. Leaving open, approved PRs invites merge conflicts as other branches get integrated.
- **Delete Branches After Merging:** Select the option on GitHub to *"Automatically delete head branches"* upon merge.
- **Write Clear PR Descriptions:** A well-documented PR speeds up the review process and serves as documentation for future developers looking back at history.

---

## Collaboration Cheat Sheet

```bash
# Update local main branch
git switch main
git pull origin main

# Create feature branch
git switch -c feature/new-component

# Push feature branch to GitHub and set upstream tracking
git push -u origin feature/new-component

# Start a release branch
git switch develop
git switch -c release/v2.1.0

# Start a hotfix branch from production
git switch main
git switch -c hotfix/critical-exploit
```

---

## What Comes Next

Collaborative workflows keep development organized, but mistakes still happen. A developer might commit code that breaks the build, commit a database password by accident, or corrupt their working directory files.

In **Chapter 10 — Undoing Mistakes**, we will master Git's recovery commands—`restore`, `reset`, `revert`, `clean`, and `reflog`—giving you the tools to resolve mistakes safely.
