# Chapter 11 — Git Stash

## Introduction

As a software developer, your workday is frequently interrupted. You might be in the middle of writing a complex new feature when an urgent production bug report comes in. To fix the bug, you need to switch back to the `main` branch immediately. 

However, Git will often block you from switching branches if you have unfinished, uncommitted changes in your working directory that overlap with changes on the target branch. Even if Git allows the switch, your unfinished modifications will carry over to the other branch, cluttering your workspace. 

You have two choices: make a messy, half-baked "Work In Progress" (WIP) commit just to save your place, or use **Git Stash**. The stash acts as a temporary drawer where you can safely slide your unfinished changes, returning your working directory to a clean state. Once the interruption is handled, you can pull your changes out of the drawer and resume exactly where you left off.

This chapter details how to save, inspect, restore, and manage stashed changes, including how to handle untracked files and create new branches directly from stashes.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain the concept of the Git Stash stack and how it stores unfinished work
- Save your active modifications (tracked and untracked files) to the stash stack
- List and inspect the contents of saved stashes
- Compare and select the correct restoration method: `git stash apply` vs. `git stash pop`
- Delete individual stashes or clear the entire stash stack safely
- Create a new branch directly from a stashed state to avoid merge conflicts

---

## How Git Stash Works

The Git Stash is structured as a **stack** (a Last-In, First-Out / LIFO data structure) managed under the `.git` directory. Every time you stash changes, Git packages your working directory and staging area changes into a commit-like structure and pushes it onto the top of the stack.

```
       Stash Stack
┌─────────────────────────┐
│ stash@{0} (Newest Stash)│ ◄─── HEAD of the stash stack
├─────────────────────────┤
│ stash@{1}               │
├─────────────────────────┤
│ stash@{2} (Oldest Stash)│
└─────────────────────────┘
```

When you restore your work, you can choose to pull the absolute newest changes from the top of the stack (`stash@{0}`) or reference an older stash lower down in the stack.

---

## 1. Saving Your Work

To save your current modifications and clean your working directory, use the stash command.

### A. Basic Stash (Quick Save)
```bash
git stash
```

**What it does:** Saves all modified, tracked files in your working directory and staging area, then runs a hard reset to return your working directory to a clean state.

### B. Stashing with a Message (Recommended)
By default, Git auto-generates a generic message containing the active branch and latest commit. To make it easier to identify your work later, save your stash with a descriptive label:

```bash
git stash push -m "WIP: customer feedback form validation"
```
*(Note: `git stash` is a shorthand for `git stash push`).*

### C. Stashing Untracked Files
By default, `git stash` **ignores** new files that have not yet been tracked by Git. If you added a new file and try to stash without staging it, the file will remain in your working directory.

To stash untracked files along with your modifications, use the `-u` or `--include-untracked` flag:

```bash
git stash -u -m "WIP: add new payment asset files"
```

### D. Stashing Ignored Files
If you want to stash every single file in your directory, including untracked files and files normally ignored by your `.gitignore` (like build directories or dependencies), use the `-a` or `--all` flag:

```bash
git stash -a
```

---

## 2. Listing and Inspecting Stashes

As you work, your stash stack can accumulate multiple entries. You can inspect what you have saved before restoring.

### A. Listing All Stashes
To view the stash stack:

```bash
git stash list
```

**Expected output:**
```
stash@{0}: On feature/payments: WIP: customer feedback form validation
stash@{1}: On main: WIP: temporary server configuration fix
stash@{2}: On feature/auth: WIP: login UI styling experiments
```

### B. Inspecting Stash Contents
To see a summary of what files were modified in your latest stash:

```bash
git stash show
```

To see the actual line-by-line differences (diff) of the changes in a specific stash:

```bash
# Show diff of the newest stash
git stash show -p

# Show diff of a specific stash entry
git stash show -p stash@{1}
```

---

## 3. Restoring Stashed Changes

When you are ready to resume work, you can restore your stashed files using one of two commands.

### A. Apply: `git stash apply`
Re-applies the stashed changes to your current working directory, but **leaves the stash entry intact in the stack**.

```bash
# Apply the newest stash
git stash apply

# Apply a specific stash entry
git stash apply stash@{1}
```

**When to use:** Use `apply` if you want to test the stashed changes on multiple different branches, or if you want to keep the stash entry as a backup copy until you are 100% sure the restored code runs.

### B. Pop: `git stash pop`
Re-applies the stashed changes to your current working directory and **immediately deletes the stash entry from the stack**.

```bash
# Pop the newest stash
git stash pop

# Pop a specific stash entry
git stash pop stash@{1}
```

**When to use:** Use `pop` for standard daily tasks to keep your stash stack clean and prevent old, obsolete entries from piling up.

### C. Handling Stash Conflicts
If you stash changes, make edits to your branch, and then try to apply or pop the stash, you might get a conflict:

```
Auto-merging src/app.py
CONFLICT (content): Merge conflict in src/app.py
The stash entry is kept in case you need it again.
```

**Important Safety Feature:** If a conflict occurs, Git will apply the changes but **will not delete** the stash from the list, even if you used `pop`. This ensures you do not lose your code. Once you resolve the conflict markers and commit, you can manually delete the stash entry.

---

## 4. Deleting Stashes

If you no longer need a stashed state, delete it to keep your stack clean.

* **Delete a single stash entry:**
  ```bash
  git stash drop stash@{0}
  ```
* **Clear all stashes (Delete the entire stack):**
  ```bash
  git stash clear
  ```

---

## 5. Branching from a Stash

If you stashed changes on a branch, and that branch has since undergone major updates, applying the stash might cause severe conflicts. You can instruct Git to create a new branch starting from the exact commit where you originally created the stash, apply the changes, and switch to it:

```bash
git stash branch feature/refactored-payments stash@{0}
```

**What it does:**
1. Creates and switches to a new branch named `feature/refactored-payments` starting from the commit where the stash was created.
2. Applies the stashed changes to the working directory.
3. Drops (deletes) the stash entry from the stack.

**Why it's useful:** It isolates your stashed work from the recent updates of the original branch, allowing you to resolve and test the code in a clean environment.

---

## Git Stash Command Quick Reference

| Command | Action |
|---|---|
| `git stash` | Stash modified tracked files. |
| `git stash push -m "msg"` | Stash with a descriptive tracking message. |
| `git stash -u` | Stash modified tracked files AND untracked files. |
| `git stash list` | Show all entries currently in the stash stack. |
| `git stash show -p <stash>` | Show the line-by-line diff of a stash entry. |
| `git stash apply <stash>` | Restore changes but keep the entry in the stack. |
| `git stash pop <stash>` | Restore changes and delete the entry from the stack. |
| `git stash drop <stash>` | Delete a specific stash entry. |
| `git stash clear` | Permanently delete all entries in the stash stack. |
| `git stash branch <name> <stash>` | Create a new branch and apply the stash to it. |

---

## What Comes Next

Stashing keeps your daily development clean and flexible. Now that we have covered daily workflows, branches, merges, rebases, and undoing mistakes, let's explore versioning and software delivery.

In **Chapter 12 — Tags & Releases**, we will learn how to create permanent pointers to key commits, use semantic versioning, and build releases on GitHub to distribute software.
