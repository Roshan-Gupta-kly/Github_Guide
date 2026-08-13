# Chapter 10 — Undoing Mistakes

## Introduction

One of the most terrifying feelings for a developer is making a mistake that seems to delete hours of work, commit the wrong code to a repository, or leak api keys. Fortunately, Git is designed with safety in mind. Almost every operation in Git is additive, meaning it writes new data rather than overwriting the old. Once a commit is recorded in your local repository database, it is very difficult to lose.

The key to fixing mistakes is knowing which command to use at which stage of the Git lifecycle. This chapter details how to discard modifications in your working directory, unstage files, undo local commits using `git reset`, undo pushed commits safely using `git revert`, prune untracked files with `git clean`, and use Git's ultimate safety net: `git reflog`.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Discard local modifications in your working directory using `git restore`
- Unstage files from the staging area without losing your edits
- Compare and select the correct `git reset` mode (`--soft`, `--mixed`, `--hard`)
- Undo commits safely on shared public branches using `git revert`
- Prune temporary and untracked files safely using `git clean`
- Recover seemingly deleted commits or aborted operations using `git reflog`

---

## The Undoing Matrix

Before running a command, determine where the mistake lives:

```
 Working Directory ──────► Staging Area ──────► Local Repository ──────► Remote (GitHub)
     (Unstaged)             (Staged)              (Committed)               (Pushed)
         │                     │                      │                        │
         ▼                     ▼                      ▼                        ▼
    git restore        git restore --staged       git reset <mode>         git revert
```

---

## 1. Undoing Unstaged Changes

If you edited a file (e.g., `src/main.py`) but realize your modifications are incorrect and you want to revert to the state of the last commit:

```bash
git restore src/main.py
```

**What it does:** Overwrites the modifications in your working directory with the version of the file stored in the last commit (or staging area).

> [!WARNING]
> **This is destructive!** Any modifications you have made to this file since your last commit or stage will be permanently discarded. Git cannot recover unstaged changes.

### Legacy Command:
In older tutorials, you will see:
```bash
git checkout -- src/main.py
```
While this works, prefer `git restore` as it was introduced in Git 2.23 specifically to separate file-restoring actions from branch switching.

---

## 2. Unstaging Files

If you ran `git add .` and staged all files, but realize you staged a config file (like `.env` or `config.json`) that you do not want to commit yet:

```bash
git restore --staged config.json
```

**What it does:** Removes the file from the Staging Area, but **keeps your edits intact** in your working directory. The file transitions from "staged" to "unstaged" (modified).

---

## 3. Undoing Local Commits: `git reset`

If you committed changes locally but need to undo that commit, use `git reset`. Reset works by moving the current branch pointer backward in history to a specific commit.

```
       HEAD ───► Commit C (Old Tip)
                   │
    git reset ┌────┘
              ▼
           Commit B (New Tip)
```

There are three modes of reset, and choosing the right one is critical:

| Mode | Moves Branch Pointer? | Clears Staging Area? | Discards Working Directory Edits? |
|---|---|---|---|
| `--soft` | Yes | No (Keeps Staged) | No |
| `--mixed` (Default) | Yes | Yes (Unstages) | No |
| `--hard` (Danger) | Yes | Yes (Unstages) | Yes (Deletes Code) |

### A. Soft Reset: `git reset --soft`
Use this if you want to undo your last commit but keep your changes staged so you can edit the commit message or add more files before committing again:

```bash
git reset --soft HEAD~1
```
*(Note: `HEAD~1` refers to "one commit before HEAD").*

### B. Mixed Reset: `git reset` (Default)
Use this if you want to undo your commit and unstage the files, but keep your edits intact in your working directory:

```bash
git reset HEAD~1
```

### C. Hard Reset: `git reset --hard` (Dangerous)
Use this if you want to completely discard your last commit, unstage the changes, and **permanently delete** all edits in your working directory:

```bash
git reset --hard HEAD~1
```

> [!CAUTION]
> **`git reset --hard` is highly destructive.** It overwrites all files in your working directory. Ensure you do not have uncommitted work you want to keep before running this.

---

## 4. Undoing Pushed Commits: `git revert`

If you have already pushed your commits to GitHub, you **must not** use `git reset`. Doing so violates the Golden Rule of Rebasing (rewriting shared history), which will break the workspace for your collaborators.

Instead, use `git revert`:

```bash
git revert <commit-hash>
```

**What it does:** Instead of deleting the target commit, Git creates a **brand new commit** that introduces the exact opposite changes of the target commit. If the target commit added a line, the revert commit deletes it.

```
History before revert:
[Commit A] ───► [Commit B (Introduced Bug)] ───► [Commit C]

History after revert (git revert B):
[Commit A] ───► [Commit B] ───► [Commit C] ───► [Commit D (Undoes changes from B)]
```

**Why it's safe:** It does not rewrite history. It only adds new history, meaning your colleagues can pull changes normally without experiencing conflicts.

---

## 5. Cleaning Untracked Files: `git clean`

`git status` showing too many untracked logs or files (like build artifacts or test output)? Use `git clean` to prune them.

* **Dry Run (Always do this first!):** Tells you exactly what files Git plans to delete:
  ```bash
  git clean -n
  ```
* **Clean Files:** Delete untracked files:
  ```bash
  git clean -f
  ```
* **Clean Files & Directories:** Delete untracked files and folders:
  ```bash
  git clean -fd
  ```

---

## 6. The Ultimate Lifeline: `git reflog`

What happens if you run a `git reset --hard` and realize you deleted commits you actually needed? Since the commits were recorded locally before, they still exist in Git's database (for about 30 days before being garbage collected).

Git tracks every movement of the `HEAD` pointer (commits, checkouts, resets, merges, rebases) in a reference log:

```bash
git reflog
```

**Expected output:**
```
4a9f1b2 HEAD@{0}: reset: moving to HEAD~1
7c8d9e0 HEAD@{1}: commit: Add database integration
e4f5g6h HEAD@{2}: commit: Fix login UI layout
```

### How to Recover:
1. Identify the point *just before* the mistake in the log (e.g., `HEAD@{1}` or hash `7c8d9e0`).
2. Point your current branch back to that commit:
   ```bash
   git reset --hard 7c8d9e0
   # OR
   git switch -c recovered-branch 7c8d9e0
   ```
Your commits and working directory are restored.

---

## Undoing Command Quick Reference

| Problem | Command | Safety |
|---|---|---|
| Unstaged edits in a file you want to discard | `git restore <file>` | ⚠️ **Destructive** (Edits deleted) |
| Staged a file by mistake; keep changes | `git restore --staged <file>` | Safe (Keeps edits) |
| Undo last local commit; keep files staged | `git reset --soft HEAD~1` | Safe (Keeps edits) |
| Undo last local commit; unstage files | `git reset HEAD~1` | Safe (Keeps edits) |
| Delete last local commit and discard edits | `git reset --hard HEAD~1` | ⚠️ **Destructive** (Edits deleted) |
| Undo a commit that has been pushed to GitHub | `git revert <commit-hash>` | Safe (Creates new history) |
| List untracked files that would be deleted | `git clean -n` | Safe (Preview) |
| View history of HEAD movements to find lost commits | `git reflog` | Safe |

---

## What Comes Next

Understanding how to undo mistakes is a crucial skill that turns Git into a workspace where you can experiment without fear. 

Sometimes, you need to switch tasks, but you aren't ready to commit your unfinished work. In **Chapter 11 — Git Stash**, we will explore how to temporarily shelf your changes and restore them later.
