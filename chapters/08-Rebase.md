# Chapter 08 — Rebase

## Introduction

In Chapter 07, we explored how `git merge` integrates branches by combining their files and creating a new merge commit. Merging is safe and non-destructive because it leaves historical commits untouched. However, if multiple developers are working on short-lived branches, a project's git history can quickly become cluttered with "Merge branch..." commits and a tangled web of branch lines.

**Rebasing** offers an alternative way to integrate changes. Instead of merging histories, rebasing rewrites history by moving a sequence of commits to a new base commit. This creates a clean, linear chain of commits as if all development happened on a single line.

This chapter details how rebasing works under the hood, how to perform basic and interactive rebases, how to resolve conflicts commit-by-commit, and outlines the critical safety rules regarding when to rebase and how to update remote servers safely.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain how `git rebase` differs from `git merge` in its handling of history
- Perform a basic rebase to keep your feature branch up-to-date with `main`
- Resolve conflicts during a rebase using `--continue` and `--abort`
- Clean up your commit history using interactive rebase (`git rebase -i`) and its sub-commands (`pick`, `reword`, `squash`, `fixup`, `drop`)
- Explain the Golden Rule of Rebasing and why rewriting public history is dangerous
- Update remote branches after rebasing safely using `git push --force-with-lease`

---

## What is a Git Rebase?

Rebasing is the process of moving the starting point (the base) of your branch to a new commit. 

Imagine you branch `feature` off of `main` at Commit B. While you are working on `feature` and making commits D and E, a colleague merges their work into `main`, creating commits F and G. Your branch is now out of date.

```
Diverged History (Before Rebase):
                  ┌─► [Commit F] ───► [Commit G] (main Tip)
                 /
[Commit A] ───► [Commit B] (Common Ancestor)
                 \
                  └─► [Commit D] ───► [Commit E] (feature Tip)
```

If you rebase `feature` onto `main`:
1. Git temporarily shelves your feature commits (D and E).
2. Git updates your feature branch pointer to match the tip of `main` (Commit G).
3. Git reapplies your shelved commits (D and E) one by one on top of Commit G.

```
Linear History (After Rebase):
[Commit A] ───► [Commit B] ───► [Commit F] ───► [Commit G] (main) ───► [Commit D'] ───► [Commit E'] (feature HEAD)
```

### Critical Detail: Commits are Recreated
Notice that the post-rebase commits are labeled **D'** and **E'**. Because their parent commits have changed, their commit metadata is updated, which means **their SHA-1 hashes are completely rewritten**. You are not moving the original commits; you are creating new, identical commit copies in their place.

---

## 1. Basic Rebasing Workflow

To catch up your local feature branch with the latest changes from `main`:

### Step 1: Fetch the Latest Changes
Ensure your local `main` is up-to-date:
```bash
git fetch origin
git switch main
git merge origin/main
```

### Step 2: Switch to Your Feature Branch and Rebase
```bash
git switch feature/login
git rebase main
```

**Expected output:**
```
Successfully rebased and updated refs/heads/feature/login.
```

---

## 2. Resolving Rebase Conflicts

Unlike `git merge`, which resolves all differences at once and creates a single merge commit, `git rebase` applies your commits **one by one**. If a conflict occurs, Git pauses at the specific commit causing the conflict.

### Step-by-Step Rebase Conflict Resolution:

1. **Git Pauses:** When Git detects a conflict, it prints:
   ```
   CONFLICT (content): Merge conflict in src/main.py
   error: Failed to merge in the changes.
   Patch failed at 0001 Add login button
   ```
2. **Identify Conflicts:** Run `git status` to see the conflicting files.
3. **Resolve Conflict:** Open the file, find the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`), resolve the conflict, and delete the marker lines.
4. **Stage the Fix:** Stage the resolved file:
   ```bash
   git add src/main.py
   ```
5. **Resume the Rebase:** Instead of running `git commit`, tell Git to continue applying the remaining commits:
   ```bash
   git rebase --continue
   ```
6. **Repeat:** If you have multiple commits that conflict, Git will pause at the next one. Repeat steps 2-5 until the rebase is finished.

### Other Options:
* **Abort the Rebase:** If the conflicts are too complex and you want to completely cancel and return your branch to its exact pre-rebase state:
  ```bash
  git rebase --abort
  ```
* **Skip the Commit:** If you want to completely skip the commit that is causing the conflict (discarding your local changes in that specific commit):
  ```bash
  git rebase --skip
  ```

---

## 3. Interactive Rebasing: Cleaning Up History

Interactive rebasing (`git rebase -i`) is a powerful tool that allows you to rewrite, clean up, and organize your commits before sharing them with your team. It is like an editor for your commit history.

To rebase the last 4 commits on your current branch:
```bash
git rebase -i HEAD~4
```

This opens a text file in your default editor containing a list of your commits, ordered from oldest to newest:

```text
pick a1b2c3d Add user profile model
pick e4f5g6h Fix typo in profile bio
pick j7k8l9m Add bio validation tests
pick n0p1q2r Change bio field character limit

# Rebase HEAD~4 onto e4f5g6h
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# d, drop <commit> = remove commit
```

### How to Edit Your History:
To change a commit's behavior, replace the word `pick` with your chosen command, then save and close the file. Git will execute your commands from top to bottom.

* **`pick` (Default):** Keeps the commit exactly as it is.
* **`reword`:** Pauses the rebase and opens your text editor so you can fix a typo or rewrite a commit message.
* **`squash`:** Merges the commit into the one above it. Git will prompt you to combine their commit messages into a single, cohesive message.
* **`fixup`:** Merges the commit into the one above it, but discards this commit's message. This is perfect for "fix typo" or "fix build" commits that don't need their own spot in the permanent history.
* **`drop`:** Deletes the commit entirely. Any code changes made in this commit are discarded.
* **Reordering:** You can physically move the lines in this file to change the order in which the commits occurred.

---

## 4. The Golden Rule of Rebasing

Because rebasing rewrites commit hashes, it is critical that you follow this rule:

> [!IMPORTANT]
> **The Golden Rule of Rebasing:**
> Never rebase a branch that has been pushed to a public/shared remote repository where other developers are actively working.

If you rewrite commits on a shared branch (like `main` or a shared feature branch) and push it, your teammates' local history will still contain the old commits. When they pull, Git will attempt to merge their old history with your rewritten history, producing duplicate commits, massive merge conflicts, and severe confusion.

* **Safe to rebase:** Local branches that only exist on your machine, or private feature branches on GitHub where you are the sole developer.
* **Unsafe to rebase:** `main`, `master`, `develop`, or any branch multiple people are pushing and pulling from.

---

## 5. Force Pushing Safely: `--force-with-lease`

If you rebase a private feature branch that you already pushed to GitHub (for a backup or Pull Request review), GitHub will reject a standard `git push`. This happens because the remote branch has your original commits, while your local branch has the rewritten commits (different hashes).

To update the remote branch, you must force the push.

### The Dangerous Way:
```bash
git push --force origin feature/login
```
**Why it's dangerous:** If a colleague checked out your branch and pushed a commit while you were rebasing, `git push --force` will overwrite and permanently delete their commits on the remote server.

### The Safe Way:
```bash
git push --force-with-lease origin feature/login
```
**Why it's safe:** `--force-with-lease` acts as a safety check. It tells Git: *"Only force push if the remote branch has not changed since the last time I fetched."* If someone else pushed changes, Git will reject the push, preventing you from accidentally overwriting their work.

---

## Rebasing Command Quick Reference

| Command | Action |
|---|---|
| `git rebase main` | Reapply current branch commits on top of the `main` branch. |
| `git rebase -i HEAD~N` | Start an interactive rebase for the last `N` commits. |
| `git rebase --continue` | Resume a rebase after resolving a conflict. |
| `git rebase --abort` | Cancel an active rebase and return to the pre-rebase state. |
| `git rebase --skip` | Skip the current conflicting commit during a rebase. |
| `git push --force-with-lease` | Safely update a rewritten remote branch. |

---

## What Comes Next

Now that you understand both merging (Chapter 07) and rebasing (Chapter 08), you have the core tools to integrate code. In **Chapter 09 — Collaboration**, we will explore how developers combine these techniques in teams, walk through Pull Request workflows, write code reviews, and set up branch protection rules to secure the codebase.
