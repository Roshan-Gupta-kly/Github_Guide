# Chapter 19 — Git Cheat Sheet

## Introduction

This cheat sheet compiles all the common Git commands covered in this developer guide into a single, categorized reference page. Use it as a quick lookup during your daily development work.

---

## 1. Setup & Configuration

Configure Git settings globally on your computer.

| Command | Action |
|---|---|
| `git --version` | Verify the installed Git version. |
| `git config --global user.name "Name"` | Set your commit attribution name. |
| `git config --global user.email "email@example.com"` | Set your commit attribution email. |
| `git config --global core.autocrlf true` | Configure Windows line-ending conversions. |
| `git config --global core.autocrlf input` | Configure macOS/Linux line-ending conversions. |
| `git config --list --show-origin` | Show all active configuration settings and their source files. |

---

## 2. Starting a Project

Initialize new repositories or copy existing ones.

| Command | Action |
|---|---|
| `git init` | Initialize an empty Git repository in the current folder. |
| `git clone <url>` | Clone a remote repository to your local machine. |
| `git remote add origin <url>` | Link a local repository to a remote repository on GitHub. |
| `git remote -v` | Verify remote connections. |

---

## 3. Daily Workflow

Inspect, stage, and save changes.

| Command | Action |
|---|---|
| `git status` | View the status of files in your repository. |
| `git status -s` | View status in a compact, short format. |
| `git diff` | View modifications in your working directory (unstaged). |
| `git diff --staged` | View modifications staged for commit. |
| `git add <file>` | Move a specific file to the staging area. |
| `git add .` | Move all modified and new files to the staging area. |
| `git add -p <file>` | Interactively review and stage changes hunk-by-hunk. |
| `git commit -m "message"` | Save staged changes with a single-line message. |
| `git commit --amend` | Edit the last commit message or add forgotten changes (unpushed commits only). |

---

## 4. Branch Management

Isolate features, bug fixes, and releases.

| Command | Action |
|---|---|
| `git branch` | List all local branches. |
| `git branch -a` | List all local and remote branches. |
| `git switch -c <name>` | Create a new branch and switch to it immediately. |
| `git switch <name>` | Switch to an existing branch. |
| `git branch -m <new-name>` | Rename the active branch. |
| `git branch -d <name>` | Safely delete a merged branch. |
| `git branch -D <name>` | Force delete a branch (even if it contains unmerged work). |
| `git push origin --delete <name>` | Delete a branch on GitHub. |

---

## 5. Integrating Changes: Merge & Rebase

Combine branch histories.

| Command | Action |
|---|---|
| `git merge <branch>` | Merge `<branch>` into your currently active branch. |
| `git merge --no-ff <branch>` | Force a merge commit to preserve feature branch history. |
| `git merge --abort` | Cancel a merge and return files to pre-merge state. |
| `git rebase <branch>` | Reapply commits from the active branch on top of `<branch>`. |
| `git rebase -i HEAD~N` | Open interactive menu to squash, reorder, or drop the last `N` commits. |
| `git rebase --continue` | Resume rebase after resolving conflicts. |
| `git rebase --abort` | Cancel an active rebase and return files to pre-rebase state. |

---

## 6. Remote Synchronization

Keep local and remote servers in sync.

| Command | Action |
|---|---|
| `git fetch origin` | Download all remote changes safely without modifying local files. |
| `git pull origin <branch>` | Download and merge remote changes into your active branch. |
| `git pull --rebase origin <branch>` | Download remote changes and rebase your local commits on top of them. |
| `git push origin <branch>` | Upload local commits to the remote branch. |
| `git push -u origin <branch>` | Upload local commits and set upstream tracking for future pushes. |
| `git push --force-with-lease` | Safely force push local commits after history rewriting (rebasing). |

---

## 7. Undoing Mistakes

Recover files, commits, and history.

| Command | Action |
|---|---|
| `git restore <file>` | Discard modifications to a file in your working directory. |
| `git restore --staged <file>` | Unstage a file, keeping modifications in your working directory. |
| `git reset --soft HEAD~1` | Undo last commit, keeping changes staged. |
| `git reset HEAD~1` | Undo last commit, unstaging files (edits kept). |
| `git reset --hard HEAD~1` | Undo last commit and permanently delete all changes (dangerous). |
| `git revert <commit-hash>` | Create a new commit that undoes the changes of `<commit-hash>` (safe for remote). |
| `git clean -n` | List untracked files that would be deleted. |
| `git clean -fd` | Delete untracked files and folders. |
| `git reflog` | View history of HEAD pointer changes to locate lost commits. |

---

## 8. Git Stash

Temporarily shelf unfinished work.

| Command | Action |
|---|---|
| `git stash` | Save modified, tracked files and clean the workspace. |
| `git stash push -m "msg"` | Save stash with a descriptive label. |
| `git stash -u` | Save modified tracked files AND untracked files. |
| `git stash list` | View all entries in the stash stack. |
| `git stash show -p <stash>` | View the line-by-line diff of a stash entry. |
| `git stash apply` | Restore the newest stash and keep it in the stack. |
| `git stash pop` | Restore the newest stash and delete it from the stack. |
| `git stash drop <stash>` | Delete a specific stash entry. |
| `git stash clear` | Delete all entries in the stash stack. |

---

## 9. Tags & Releases

Mark milestones and publish software versions.

| Command | Action |
|---|---|
| `git tag` | List all local tags. |
| `git tag -a <tag> -m "msg"` | Create a new local annotated tag. |
| `git show <tag>` | Inspect tag details, tagging message, and commit details. |
| `git tag -d <tag>` | Delete a local tag. |
| `git push origin <tag>` | Push a specific tag to remote. |
| `git push origin --tags` | Push all local tags to remote. |
| `git push origin --delete <tag>` | Delete a tag on remote. |
