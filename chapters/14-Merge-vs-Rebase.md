# Chapter 14 — Merge vs. Rebase

## Introduction

In Git, there are two primary ways to integrate changes from one branch into another: `git merge` and `git rebase`. Both commands solve the same problem—combining features with the main codebase—but they do so using fundamentally different philosophies.

The choice between merging and rebasing is one of the most debated topics in software development. Proponents of merging value historical accuracy and safety, while advocates of rebasing prioritize clean, readable, and linear commit logs. 

This chapter provides a detailed, side-by-side comparison of `git merge` and `git rebase`. We will analyze the internal mechanics, pros and cons of each, how they handle conflicts, and establish a hybrid workflow that is recommended for professional development teams.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Differentiate the structural outcomes of merging vs. rebasing
- Evaluate the advantages and disadvantages of the merge workflow
- Evaluate the advantages and disadvantages of the rebase workflow
- Explain how conflict resolution differs between the two commands
- Implement a hybrid workflow that leverages the strengths of both tools
- Help your team establish a consistent branch integration policy

---

## Side-by-Side Comparison

To choose the right tool for the job, you must understand their structural differences:

| Feature | `git merge` | `git rebase` |
|---|---|---|
| **Underlying Action** | Combines two branch tips using a common ancestor. | Reapplies commits from one branch on top of another. |
| **History Structure** | Non-linear (creates branch forks and merge commits). | Linear (straight line of commits). |
| **Commit Hashes** | Preserved (unchanged). | Rewritten (new commit copies are created). |
| **Chronological Order** | Preserved. | Altered (commits are moved to the tip of target branch). |
| **Conflict Resolution** | All at once (resolved in a single merge commit). | Commit-by-commit (resolved at each step). |
| **Safety** | High (does not rewrite existing history). | Moderate/Low (danger of breaking shared remote branches). |

---

## 1. The Merge Workflow

Merging is a **non-destructive** operation. It does not alter your existing branch history in any way.

```
       Before Merge:
                         ┌─► [Commit F] (main)
                        /
       [Commit A] ─────┼
                        \
                         └─► [Commit D] ───► [Commit E] (feature)

       After Merge (git merge feature):
                         ┌─► [Commit F] ───┐
                        /                   ▼
       [Commit A] ─────┼────────────────► [Merge Commit G] (main HEAD)
                        \                   ▲
                         └─► [Commit D] ───► [Commit E] ───┘
```

### Advantages of Merging:
* **Historical Accuracy:** It preserves the exact chronological history of when work occurred. You can see precisely when a feature branched off and when it was integrated.
* **Traceability:** It creates a dedicated **merge commit** (Commit G in the diagram). If a feature introduces a bug, you can revert the entire feature merge in one operation by reverting the merge commit.
* **Safety:** It does not modify existing commit hashes, making it 100% safe to use on public, shared branches.

### Disadvantages of Merging:
* **Repository Clutter:** If your team has many developers working on short-lived branches, the git history becomes filled with automated "Merge branch..." commits.
* **Complex Git Graphs:** In large projects, the visual representation of history can look like a tangled web of overlapping lines, making it nearly impossible to trace code changes.

---

## 2. The Rebase Workflow

Rebasing is a **destructive** operation because it rewrites history. It recreates your commits at a new base commit.

```
       Before Rebase:
                         ┌─► [Commit F] (main)
                        /
       [Commit A] ─────┼
                        \
                         └─► [Commit D] ───► [Commit E] (feature)

       After Rebase (git rebase main):
       [Commit A] ───► [Commit F] (main) ───► [Commit D'] ───► [Commit E'] (feature HEAD)
```

### Advantages of Rebasing:
* **Clean, Linear History:** The primary benefit. The history is represented as a single straight line. You can read the commit log from top to bottom without navigating forks or merge commits.
* **Simplified Reviews:** It is much easier to review code changes commit-by-commit when they are arranged in a linear sequence.
* **Polished Commits:** Using interactive rebase (`git rebase -i`), you can squash temporary "wip" or "fix typo" commits, rewrite commit messages, and clean up your commits before submitting them.

### Disadvantages of Rebasing:
* **Rewrites History:** By modifying commit hashes, it poses a risk of breaking other developers' local setups if run on shared branches (violating the Golden Rule of Rebasing).
* **Falsified Timeline:** Commits are moved to the tip of the branch. The chronological context of *when* the code was originally written compared to other commits is lost.
* **Intermediate Test Failures:** If you rebase and write a bug during conflict resolution, intermediate commits (like Commit D' in the diagram) might have broken builds, which makes using `git bisect` to locate bugs harder.

---

## 3. Conflict Resolution: Merge vs. Rebase

How you resolve conflicts differs significantly between the two commands:

### Merging Conflicts (All-at-Once)
When you merge and a conflict occurs, Git pauses. You resolve the conflict in the conflicting files, run `git add`, and run `git commit` to create the merge commit. 
* **Conflict resolution is isolated to a single commit.**
* If the merge is too difficult, a single `git merge --abort` undoes everything.

### Rebasing Conflicts (Commit-by-Commit)
When you rebase, Git applies your commits one-by-one. If a conflict occurs, Git pauses at the *first* commit that introduces conflicts.
* You resolve the conflicts for that commit, run `git add`, and run `git rebase --continue`.
* Git then moves to the *next* commit. If that commit also conflicts with changes, Git pauses **again**.
* **You may have to resolve conflicts multiple times** for a single rebase operation if your changes overlap with the target branch across several commits.

---

## 4. The Team Recommendation: The Hybrid Workflow

To get the best of both worlds—the clean history of rebasing and the safety of merging—most professional development teams adopt a **hybrid workflow**.

```
    1. Develop Feature Local  ──►  2. Keep Up-To-Date  ──►  3. Squash & Merge
         (Make Commits)             (git rebase main)        (GitHub Pull Request)
```

### The Rules of the Hybrid Workflow:

#### Rule 1: Rebase Locally
While working on your private feature branch, keep it up-to-date with `main` by rebasing:
```bash
git fetch origin
git rebase origin/main
```
This keeps your local history linear and ensures that when you open a Pull Request, there will be no merge conflicts.

#### Rule 2: Clean Up Locally
Before pushing your branch for review, run an interactive rebase to squash temporary commits:
```bash
git rebase -i origin/main
```
Combine "fix typo," "wip," and "oops" commits into clean, descriptive commits.

#### Rule 3: Squash and Merge on GitHub
When the Pull Request is approved, integrate it using GitHub's **Squash and Merge** option. 

This squashes all your feature commits into a single, clean commit on `main`. It maintains a linear history on the production branch while preventing any local rebase mistakes from affecting your team.

---

## Merge vs. Rebase Summary

Use this simple decision helper for your daily work:

* **Use `git merge` when:**
  * You are integrating changes on public, shared branches (e.g., merging `release` into `main`).
  * You want to preserve the exact chronological history of a complex project.
  * You want to resolve conflicts in a single step.

* **Use `git rebase` when:**
  * You are working on a private, local feature branch and want to catch up with changes on `main`.
  * You want to clean up, reorder, or squash commits before sharing them.
  * You want to keep the overall repository history clean, linear, and readable.

---

## What Comes Next

Understanding the trade-offs between merging and rebasing is a hallmark of an advanced Git user. 

Now that we have covered all the fundamental and intermediate workflows of Git and GitHub, developers will occasionally encounter errors. In **Chapter 15 — Common Git Errors**, we will detail the most common errors developers encounter—such as detached HEAD, index locks, and non-fast-forward push rejections—explaining why they happen and how to resolve them.
