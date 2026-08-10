# Chapter 07 — Merge

## Introduction

Branching allows you to work in isolation, but a feature branch is only useful if it is eventually integrated back into the main codebase. In Git, the primary method for combining separate lines of development is **merging**. 

While merging is often automatic and seamless, it is also one of the most common sources of anxiety for developers. The phrase "merge conflict" can strike fear into beginners, but once you understand how Git determines file differences and how to read conflict markers, resolving them becomes a routine, logical task.

This chapter details the two core types of merges—Fast-Forward merges and Three-Way merges—explains when and why merge commits are created, and provides a step-by-step framework for resolving conflicts and safely aborting problematic merges.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Differentiate between a Fast-Forward merge and a Three-Way merge
- Explain how Git uses a common ancestor to perform a three-way comparison
- Execute a merge command and understand the structure of a merge commit
- Configure and use the `--no-ff` flag to preserve feature history
- Identify, read, and resolve merge conflicts systematically
- Safely cancel an active merge using `git merge --abort`

---

## The Two Types of Merges

When you run `git merge <source-branch>`, Git looks at the history of your current branch (the target) and the branch you want to merge (the source) to determine the best integration strategy.

```
       [ PATH A: Fast-Forward ]                      [ PATH B: Three-Way ]
  "Linear history. Main has not moved             "Divergent history. Both branches
  since Feature branched off."                    have new commits since ancestor."

  main ───► [Commit A]                            main ────► [Commit A] ───► [Commit C]
                 \                                                \
  feature ────────► [Commit B]                    feature ─────────► [Commit B]
```

---

## 1. Fast-Forward Merge

A **Fast-Forward (FF)** merge occurs when the target branch has not diverged since the source branch was created. In other words, there are no new commits on the target branch.

### How Git Handles It:
Because the history is linear, Git does not need to combine file contents. Instead, it simply moves the target branch pointer forward to point to the latest commit of the source branch. No new merge commit is created.

```
Before Merge:
main (HEAD) ───► [Commit A]
                    \
feature ─────────────► [Commit B]

After Merge (git merge feature):
main (HEAD) ─────────► [Commit B]
                       ▲
feature ───────────────┘
```

### Commands:
```bash
git switch main
git merge feature/login
```

**Expected output:**
```
Updating a1b2c3d..e4f5g6h
Fast-forward
 src/login.py | 12 ++++++++++++
 1 file changed, 12 insertions(+)
```

### Forcing a Merge Commit: `--no-ff`
Sometimes, you want to preserve the historical record that a feature branch existed, even if a Fast-Forward merge is possible. You can force Git to create a merge commit using the `--no-ff` (no fast-forward) flag:

```bash
git merge --no-ff feature/login
```

**Why we use it:** This creates a visual "bump" in your git graph, grouping all feature commits together and marking exactly where the feature was integrated.

---

## 2. Three-Way Merge

A **Three-Way** merge occurs when the histories of the two branches have diverged. This means new commits have been made on the target branch *and* on the source branch since they split.

### How Git Handles It:
Git cannot simply move a branch pointer forward. Instead, it performs a **three-way comparison** using three snapshots:
1. The tip of the target branch (e.g., `main`).
2. The tip of the source branch (e.g., `feature/login`).
3. The **common ancestor** (the point where the feature branch split off from `main`).

```
Before Merge:
                  ┌─► [Commit C] (main Tip)
                 /
[Commit A] ─────┼ (Common Ancestor)
                 \
                  └─► [Commit B] (feature Tip)

After Merge (git merge feature):
                  ┌─► [Commit C] ───┐
                 /                   ▼
[Commit A] ─────┼────────────────► [Merge Commit D] (main HEAD)
                 \                   ▲
                  └─► [Commit B] ───┘
```

Git automatically combines the changes. If the changes do not overlap (e.g., they modify different files or different lines of the same file), Git successfully merges the code and automatically creates a new commit—a **merge commit**—which uniquely has **two parent commits**.

---

## 3. Resolving Merge Conflicts

If you and a teammate edit the exact same line of the same file on different branches, or if one branch deletes a file that the other branch modifies, Git cannot automatically merge them. It halts the process, leaves your repository in an "Unmerged" state, and asks you to resolve the conflict.

### Step 1: Detect the Conflict
When a conflict occurs, `git merge` will output a message like this:

```
Auto-merging src/index.html
CONFLICT (content): Merge conflict in src/index.html
Automatic merge failed; fix conflicts and then commit the result.
```

If you run `git status`, you will see the conflicting files under **Unmerged paths**:

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   src/index.html
```

### Step 2: Read the Conflict Markers
Open the conflicting file in your text editor. Git inserts special **conflict markers** around the competing lines:

```html
<<<<<<< HEAD
<h1>Welcome to Our App (Main Branch Version)</h1>
=======
<h1>Join Us Today! (Feature Branch Version)</h1>
>>>>>>> feature/login
```

* **`<<<<<<< HEAD`**: Marks the beginning of the conflicting lines on your *active* branch (the branch you are merging *into*).
* **`=======`**: The divider between the two competing changes.
* **`>>>>>>> feature/login`**: Marks the end of the conflicting lines on the incoming branch (the branch you are merging *from*).

### Step 3: Resolve and Clean Up
To resolve the conflict, you must manually edit the file to determine what the final code should look like. You can:
- Keep the `HEAD` version.
- Keep the `feature/login` version.
- Combine both versions.
- Write entirely new code.

**Crucial Step:** You must delete all the conflict marker lines (`<<<<<<<`, `=======`, `>>>>>>>`) from the file before saving.

**Resolved File Example:**
```html
<h1>Welcome to Our App! Join Us Today!</h1>
```

### Step 4: Stage and Commit the Resolution
Once you have saved the file, stage the resolved file to tell Git the conflict is resolved:

```bash
git add src/index.html
```

Then, complete the merge commit:
```bash
git commit
```
*(Git will open your default editor with a pre-populated merge commit message like "Merge branch 'feature/login' of ...". You can simply save and exit).*

---

## 4. Aborting a Merge: `git merge --abort`

If you trigger a merge, encounter complex conflicts, and realize you are not ready to resolve them (or merged the wrong branch by mistake), you can completely cancel the process.

```bash
git merge --abort
```

**What it does:** Reverts your working directory and staging area back to the exact state they were in before you ran `git merge`. It is a safe, non-destructive undo button for active merges.

---

## Merging Best Practices

- **Pull Before You Merge:** Always ensure your target branch is up-to-date with the remote repository before running a merge. Run `git pull origin main` first.
- **Merge Often:** If you work on a feature branch for weeks, the common ancestor becomes older, increasing the risk of massive conflicts. Frequently merge `main` into your feature branch to stay up to date.
- **Keep Commits Clean:** Fix build errors and test failures *before* merging. A merge commit should represent stable integration.
- **Use GUI Tools for Complex Conflicts:** While resolving conflicts in plain text works, modern editors like VS Code provide visual buttons ("Accept Current Change", "Accept Incoming Change") and side-by-side merge editors that make resolution easier.

---

## Merging Command Quick Reference

| Command | Action |
|---|---|
| `git merge <branch>` | Merge `<branch>` into your currently active branch. |
| `git merge --ff-only <branch>` | Merge only if a Fast-Forward is possible; fail otherwise. |
| `git merge --no-ff <branch>` | Force a merge commit to preserve branch history. |
| `git status` | Check which files have unmerged conflicts. |
| `git merge --abort` | Cancel the current merge and revert all files to pre-merge state. |

---

## What Comes Next

Merging is the safest, most non-destructive way to integrate branches because it preserves the exact history of when commits occurred. However, it can make the git graph look cluttered with merge commits.

In **Chapter 08 — Rebase**, we will explore an alternative integration method that rewrites commit bases to produce a clean, linear history.
