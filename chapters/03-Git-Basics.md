# Chapter 03 — Git Basics

## Introduction

Chapter 03 is the first practical chapter in this guide. After Chapter 01 explained why version control exists and how Git and GitHub fit together, and Chapter 02 ensured your machine is correctly installed and configured, this chapter defines the core Git concepts you will use every day.

This lesson is intentionally concise and focused. It maps the Git model into simple terms so that the commands in later chapters feel natural instead of mysterious.

---

## Learning Objectives

By the end of this chapter, you should be able to:

- Describe what a Git repository is and how it differs from the working directory
- Explain the three main Git states: working directory, staging area, and local repository
- Understand what a commit represents and why it is the fundamental unit of Git history
- Define `HEAD` and how it points to the current branch tip
- Explain the purpose of branches and how they make parallel work possible
- Know the difference between local and remote repositories
- Recognize how merge and rebase fit into Git's history model

---


## A Git repository is more than just files

A Git project has several layers. Understanding each one and how they interact is essential for correct day-to-day usage:

- Working directory (your editable files)
- Staging area (the index)
- Local repository (commits on your machine)
- Remote repository (shared copies on servers like GitHub)

These layers let Git track, stage, record, and share changes reliably.

---

## Repository

A repository (repo) is the complete project tracked by Git. It includes:

- A working directory containing the current files
- A .git directory that stores the local repository, configuration, and metadata

Commands:

- Initialize a new repository:

  git init my-project

- Clone an existing remote repository:

  git clone https://github.com/owner/repo.git

Example: After git clone, you have a working directory populated with files and a .git folder containing the full history.

---

## Working Directory

The working directory (also called working tree) is the files you can edit right now. Git compares the working directory to the index (staging area) and the last commit to determine changes.

Common commands and examples:

- Show status of working directory vs index:

  git status

- See changes in a file:

  git diff path/to/file  # unstaged changes (working directory vs index)

Example workflow:

1. Edit README.md
2. Run git status → shows README.md as modified
3. Run git diff README.md → shows the actual line changes

---

## Staging Area (Index)

The staging area is where you prepare the exact changes that will go into the next commit. Staging lets you make partial commits and craft coherent commit messages.

Commands and examples:

- Stage a file:

  git add README.md

- Stage portions of a file interactively:

  git add -p README.md

- See what is staged:

  git diff --staged  # differences between index and HEAD

Example: You modify fileA and fileB but only want to commit fileA. Run git add fileA, then git commit -m "Add fileA changes". fileB remains unstaged for later.

---

## Local Repository

The local repository is the database of commits stored under .git. Each commit is an immutable object that records a snapshot of the project and metadata.

Commands and examples:

- Create a commit from staged changes:

  git commit -m "Add feature X"

- View commit history:

  git log --oneline --graph --decorate --all

- Inspect a commit's content:

  git show <commit-hash>

Example: After several commits you can traverse history, check out old commits, or create branches from any point in history.

---

## Remote Repository

A remote repository is a hosted copy of the repo (for example on GitHub). Remotes let teams share changes and back up history.

Common commands:

- Add a new remote:

  git remote add origin git@github.com:owner/repo.git

- Push local branch to remote:

  git push -u origin feature-branch

- Fetch updates from remote without merging:

  git fetch origin

- Pull (fetch + merge) from remote:

  git pull origin main

Example: Use git fetch to get updated branches and inspect them before merging. git pull is convenient but can create merge commits unexpectedly; professionals often prefer git fetch + git merge or git fetch + git rebase.

---

## Commit

A commit records a snapshot of the project. It contains the tree (file contents), parent commit(s), author/committer metadata, and a message.

Best practices:

- Make small, focused commits
- Use meaningful commit messages with a short summary and optional body
- Stage only related changes together

Example:

  git add src/module.py tests/test_module.py

  git commit -m "Add input validation to module and tests"

Inspecting commits:

  git log --pretty=fuller  # shows author/committer metadata

Rewriting commits (advanced):

  git commit --amend  # change the last commit (only for local, not pushed shared history)

---

## HEAD

HEAD is a symbolic reference that points to the current branch tip (the commit you have checked out). When you create a commit, the branch that HEAD points to advances to the new commit.

Common scenarios:

- Detached HEAD: when you check out a specific commit (git checkout <hash>), HEAD points directly to a commit. Commits made in this state are not on any branch unless you create one.

Commands:

  git checkout main         # move HEAD to branch 'main'

  git checkout <commit-hash>  # detached HEAD

  git switch -c new-branch  # create branch and switch (safer than checkout)

Example: If you accidentally commit in a detached HEAD, create a branch to preserve work:

  git switch -c save-my-work

---

## Branch

A branch is a movable pointer to a commit. Branches are cheap and encouraged for isolating work.

Common commands:

- Create and switch to a branch:

  git switch -c feature/login

- List branches:

  git branch --all

- Delete a local branch:

  git branch -d feature/old

Example workflow:

1. Start from main: git switch -c feature/auth
2. Make commits on feature/auth
3. When ready, merge or rebase onto main

Branch naming conventions (recommended):

- feature/<short-name>
- fix/<issue-number>-short-desc
- release/<version>

---

## Merge

Merge combines the tips of two branches and preserves history. The default merge creates a merge commit when branches have diverged.

Commands:

- Merge a branch into the current branch:

  git merge feature/auth

- Perform a fast-forward only merge (fail if merge commit required):

  git merge --ff-only feature/auth

Example merge:

Suppose main and feature both advanced. On main:

  git merge feature/auth

If both branches touched different files, Git creates a merge commit with two parents. If they touched the same lines, Git reports a conflict that you must resolve and then finish with git add <resolved-files> and git commit (or git merge --continue).

---

## Rebase

Rebase reapplies commits from one branch onto another, creating a linear history by changing commit bases.

Commands and examples:

- Rebase feature branch onto main:

  git switch feature/auth 

  git rebase main

- Interactive rebase to edit, squash, or reorder commits:

  git rebase -i HEAD~4

Example: If feature/auth branched from an older main, rebasing moves feature commits to the tip of main so the history looks as if work started from the latest main.

Conflict resolution during rebase is similar to merge: resolve files, git add them, then git rebase --continue. To abort a rebase use git rebase --abort.

When to use which:

- Use merge to preserve the true historical record and when collaborating on shared branches.
- Use rebase to keep a clean, linear history for private feature branches before pushing. Do not rebase public/shared commits.

---

## Short example: from zero to integrated feature

1. Initialize and create a branch:

  - git init

  - git switch -c feature/hello

2. Edit files, stage and commit:

  - git add hello.txt

  - git commit -m "Add hello.txt"

3. Switch to main and merge:

  - git switch main

  - git merge feature/hello

Or rebase workflow (private branch):

  - git switch feature/hello

  - git rebase main

  - git switch main

  - git merge --ff-only feature/hello

These commands show how working directory → staging → commit → branch → merge/rebase interact in a real flow.

---

## What comes next

Once you understand these core Git concepts, you are ready to create your first repository and make your first commits in Chapter 04.

Chapter 03 is the bridge between setup and practical workflow: you now know the Git model, and the next chapters will show you how to use it.
