# Chapter 06 — Branching

## Introduction

In typical software development, you rarely work on a single task at a time. You might be writing a new feature, fixing a critical production bug, and experimenting with a new framework all in the same week. If you were to do all of this on a single line of history (like the `main` branch), your code would quickly become a tangled, broken mess.

This is where **branching** comes in. In Git, branching is not a heavy, expensive operation that duplicates your files. Instead, it is a lightweight, instantaneous pointer adjustments that allow you to isolate your work in independent sandboxes. 

This chapter details how Git branches work under the hood, how to create, switch, rename, and delete them, and how to set up tracking relationships with remote servers. We will also explore the dominant industry branching strategies so you can collaborate like a professional.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain how Git branches are structured internally as lightweight reference files
- Create, list, and switch between branches using modern `git switch` commands
- Safely rename branches locally and sync those changes to remote servers
- Delete branches locally and on remotes once their features are merged or discarded
- Define tracking branches and configure local branches to follow remote-tracking branches
- Evaluate and select appropriate branching strategies (GitHub Flow, Git Flow, Trunk-Based Development)
- Apply professional branch naming conventions in a team setting

---

## Under the Hood: What is a Git Branch?

In older version control systems (like SVN), creating a branch meant copying the entire directory of source files to another folder on the server. For large projects, this was slow and consumed significant storage.

In Git, a branch is simply a **lightweight, movable pointer to a commit**. It is literally a plain-text file containing a 40-character SHA-1 commit hash.

```
       HEAD
        │
        ▼
      main ────────┐
                   ▼
               [Commit C]
               ▲        ▲
               │        │
      feature ─┘        │
                        │
                    [Commit B]
                        ▲
                        │
                    [Commit A]
```

When you initialize a repository, Git creates a default branch (typically `main`). When you start making commits, the branch pointer automatically moves forward to point to your latest commit.

* **Creating a branch:** Git creates a new pointer file containing the hash of the current commit you are standing on. No files are copied.
* **Switching branches:** Git changes the `HEAD` pointer to point to the new branch and updates your working directory files to match the snapshot of that branch's latest commit.

Because of this design, branch operations in Git take a fraction of a second, regardless of whether your project has 10 files or 100,000 files.

---

## 1. Creating and Switching Branches

Prior to Git version 2.23, developers used the overloaded `git checkout` command to both switch branches and restore files. To make things clearer, Git introduced `git switch` specifically for branch operations.

### A. Creating a Branch
To create a branch without switching to it:

```bash
git branch feature/user-profile
```

### B. Switching to a Branch
To switch your working directory to an existing branch:

```bash
git switch feature/user-profile
```

**What it does:** Updates the files in your working directory to match the commit that `feature/user-profile` points to, and moves `HEAD` to point to `feature/user-profile`.

### C. Creating and Switching in One Step
The most common daily command for starting new work:

```bash
git switch -c feature/user-profile
```
*(The `-c` flag stands for "create").*

> **Legacy Command:** You will often see older tutorials use `git checkout -b <name>`. While this still works, prefer using the modern `git switch -c <name>` for clarity.

### D. Listing Branches
To see the branches in your repository:

* **List local branches:**
  ```bash
  git branch
  ```
* **List remote-tracking branches:**
  ```bash
  git branch -r
  ```
* **List all branches (local and remote):**
  ```bash
  git branch -a
  ```

---

## 2. Renaming Branches

Names change as features evolve. Renaming a branch in Git is simple, but requires special care if the branch has already been pushed to a remote server.

### A. Renaming Your Current Local Branch
If you are currently checked out on the branch you want to rename:

```bash
git branch -m feature/new-profile-layout
```

### B. Renaming a Different Local Branch
If you are checked out on `main` and want to rename a different branch:

```bash
git branch -m feature/old-name feature/new-name
```

### C. Renaming a Remote Branch
You cannot rename a branch directly on GitHub using local terminal commands. Instead, you must delete the old remote branch and push the renamed local branch:

1. Rename the local branch:
   ```bash
   git branch -m feature/new-name
   ```
2. Push the renamed local branch and reset its upstream tracking:
   ```bash
   git push origin -u feature/new-name
   ```
3. Delete the old branch on the remote:
   ```bash
   git push origin --delete feature/old-name
   ```

---

## 3. Deleting Branches

Once a feature is completed and merged into `main`, or if an experimental branch is abandoned, you should delete the branch to keep the repository clean.

### A. Deleting a Local Branch Safely
Use the lowercase `-d` flag to delete a branch:

```bash
git branch -d feature/user-profile
```

**Safety Check:** Git will block this command if the branch contains commits that have not yet been merged into your active branch. This prevents accidental loss of work.

### B. Force Deleting a Local Branch (Danger)
If you want to discard an experimental branch containing unmerged changes, force the deletion with an uppercase `-D`:

```bash
git branch -D feature/failed-experiment
```

### C. Deleting a Remote Branch
To delete a branch on GitHub:

```bash
git push origin --delete feature/user-profile
```

---

## 4. Tracking Branches

When you clone a repository, Git automatically associates your local `main` branch with the remote `origin/main` branch. This linkage makes it a **tracking branch** (or an upstream branch).

### Why Tracking Matters
If a branch is configured to track a remote counterpart, commands like `git push` and `git pull` automatically know where to upload/download changes without requiring extra arguments. It also allows `git status` to tell you if your local branch is ahead, behind, or has diverged from the remote.

### A. Checking Tracking Relationships
To see details of all local branches and which remote branches they track:

```bash
git branch -vv
```

**Expected output:**
```
* main               4a9f1b2 [origin/main] Fix email validation regex
  feature/profile    7c8d9e0 [origin/feature/profile: ahead 1] Add bio section
  feature/experimental 8e9f2a1 [origin/feature/experimental: behind 3] Edit layout
```

### B. Setting Upstream Tracking for an Existing Branch
If you created a local branch and want to bind it to a remote branch:

```bash
git branch -u origin/feature/profile
# OR
git push -u origin feature/profile
```

---

## 5. Professional Branching Strategies

In team environments, engineers must agree on a set of rules governing how branches are used. The three most common patterns in the industry are:

### Strategy 1: GitHub Flow (Recommended for Continuous Delivery)
A simple, lightweight workflow centered around the `main` branch:

```
  main  ────────────────────────────────────────────────────────► (Production)
          \                                           /
  feature  └───[Commit]───[Commit]───[PR Review]─────┘ (Merge)
```

1. The `main` branch is always stable and deployable.
2. To work on anything new, create a descriptive branch off of `main` (e.g., `feature/login`).
3. Commit changes locally and push them to GitHub.
4. Open a **Pull Request (PR)** for review.
5. Once approved and passing tests, merge the branch into `main` and deploy.

### Strategy 2: Git Flow (Recommended for Scheduled releases)
A strict, structured model introducing multiple persistent branches:

* `main` — Contains only official release history.
* `develop` — The integration branch for features. All features branch off of and merge back into `develop`.
* `feature/*` — Temporary branches for individual features.
* `release/*` — Temporary branches to prepare for production releases.
* `hotfix/*` — Emergency fixes branched off `main` and merged to both `main` and `develop`.

*Use when you have defined, versioned releases (e.g., desktop apps or embedded systems).*

### Strategy 3: Trunk-Based Development (Recommended for Large Teams)
Developers merge short-lived feature branches back into the "trunk" (`main`) multiple times a day.
* Encourages small, incremental changes.
* Reduces merge pain by preventing branches from diverging over days or weeks.
* Relies heavily on automated tests (CI) and feature flags to hide incomplete work.

---

## Branching Best Practices

- **Create a Branch for Everything:** Never write commits directly to `main` (unless you are working alone on a small personal repository). Always work on a separate branch.
- **Keep Branches Short-Lived:** The longer a branch is separated from `main`, the harder it will be to merge later due to drift. Try to merge branches within a few days.
- **Use Prefixes for Naming:** Structure your branch names with prefixes to make their purpose obvious:
  - `feature/login-validation`
  - `bugfix/issue-42`
  - `hotfix/cors-error`
  - `docs/api-guide`
- **Delete Branches After Merging:** Do not let stale, merged branches accumulate locally or on GitHub. Keep your workspaces clean.

---

## Branching Command Quick Reference

| Command | Action |
|---|---|
| `git switch -c <name>` | Create and switch to a new branch. |
| `git switch <name>` | Switch to an existing branch. |
| `git branch` | List local branches. |
| `git branch -a` | List local and remote branches. |
| `git branch -m <new-name>` | Rename the current branch. |
| `git branch -d <name>` | Safely delete a merged branch. |
| `git branch -D <name>` | Force delete an unmerged branch. |
| `git push origin --delete <name>` | Delete a branch on GitHub. |
| `git branch -vv` | Verify branch tracking/upstream details. |

---

## What Comes Next

Branching isolates work, but eventually, those branches must be brought back together. In **Chapter 07 — Merge**, we will explore how Git integrates branches, the difference between Fast-Forward merges and Three-Way merges, and how to resolve the dreaded merge conflict.
