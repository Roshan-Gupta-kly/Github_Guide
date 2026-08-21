# Chapter 15 — Common Git Errors & Troubleshooting

## Introduction

No matter how experienced you are, you will eventually run into Git errors. Some errors print lengthy, intimidating terminal warnings that can make it look like your repository has crashed or your code is lost. 

In reality, Git is very protective of your files. Most errors are not system failures; they are safety triggers designed to prevent you from overwriting your team's code, committing in the wrong state, or leaking credentials. 

This chapter compiles the most common errors developers encounter. For every error, we explain **Why it happens**, **How to fix it**, and **How to prevent it** from happening again.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Troubleshoot and resolve SSH and HTTPS authentication failures
- Recover from a Detached HEAD state without losing local commits
- Safely remove index lock files and editor swap files
- Resolve non-fast-forward push rejections and branch divergence
- Configure line-ending conversions (LF vs. CRLF) to prevent cross-platform file noise
- Diagnose and fix common configuration errors (like refspec errors and remote naming conflicts)

---

## 1. Authentication Failed / Permission Denied

```
fatal: Authentication failed for 'https://github.com/user/repo.git/'
# OR
git@github.com: Permission denied (publickey).
```

### Why it happens:
Your credentials have changed, expired, or are not configured correctly.
* **HTTPS:** Your Personal Access Token (PAT) has expired, or your Git Credential Manager has cached outdated login details.
* **SSH:** The SSH key on your machine does not match the public key registered on your GitHub account, or the SSH agent is not running.

### How to fix:
* **For SSH:** Verify if Git can connect to GitHub:
  ```bash
  ssh -T git@github.com
  ```
  If it fails, start the SSH agent and add your private key:
  ```bash
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```
  Ensure your public key (`~/.ssh/id_ed25519.pub`) is added to **Settings > SSH and GPG keys** on GitHub.
* **For HTTPS:** Force the credential manager to prompt you for new credentials:
  ```bash
  git credential-manager reject
  ```
  Generate a new Personal Access Token (PAT) on GitHub under **Settings > Developer Settings > Personal Access Tokens** and use it as your password.

### Prevention:
Use SSH keys instead of HTTPS passwords where possible. If using HTTPS, ensure your Git Credential Manager is installed and active.

---

## 2. Detached HEAD State

```
Note: switching to '7c8d9e0'.
You are in 'detached HEAD' state. You can look around, make experimental changes and commit them...
```

### Why it happens:
You ran `git checkout <commit-hash>` or `git checkout origin/main`. Rather than checking out a local branch pointer, you checked out a specific commit. `HEAD` is now pointing directly to a commit object instead of a branch. Any commits you make in this state do not belong to any branch and will be lost when you switch away.

```
       Normal State:                    Detached HEAD State:
   HEAD ──► main ──► [Commit C]            HEAD ────────┐
                                                        ▼
                                             main ──► [Commit C]
```

### How to fix:
* **To discard changes and go back:** Switch back to your active branch:
  ```bash
  git switch main
  ```
* **To save changes you made while detached:** Create a new branch pointing to your current detached commit:
  ```bash
  git switch -c feature/saved-detached-work
  ```
  This creates a branch pointer at your current commit, making it safe.

### Prevention:
Use `git switch <branch-name>` instead of `git checkout` for switching branches, reserving checkout commands for reading past commits.

---

## 3. Index Lock Exists

```
fatal: Unable to create 'C:/project/.git/index.lock': File exists.
Another git process seems to be running in this repository...
```

### Why it happens:
To prevent data corruption, Git locks its index database (`.git/index`) while running write operations. If a Git command crashes, is terminated forcefully (like using Ctrl+C), or an IDE extension is running a Git command in the background, the lock file `.git/index.lock` remains behind, blocking all future commands.

### How to fix:
1. Ensure no terminal window or IDE is currently running a Git operation.
2. Manually delete the lock file:
   * **Windows (PowerShell):**
     ```powershell
     Remove-Item .git/index.lock
     ```
   * **macOS / Linux:**
     ```bash
     rm .git/index.lock
     ```

### Prevention:
Avoid closing your terminal window mid-command. Let operations like `git push` or `git pull` finish.

---

## 4. Push Rejected / Non-Fast-Forward

```
error: failed to push some refs to 'github.com:user/repo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref.
```

### Why it happens:
You are trying to push commits to a remote branch, but the remote branch contains other commits that you do not have locally. Git rejects the push to prevent you from overwriting your team's changes.

### How to fix:
1. Pull the remote changes and integrate them into your branch:
   ```bash
   git pull origin main
   ```
2. (Or rebase to maintain a clean timeline):
   ```bash
   git pull --rebase origin main
   ```
3. Resolve any merge conflicts, commit them, and then push:
   ```bash
   git push origin main
   ```

### Prevention:
Always pull remote changes before attempting to push.

---

## 5. Diverged Branches

```
Your branch and 'origin/main' have diverged,
and have 1 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)
```

### Why it happens:
You have made commits locally on your branch, and someone else has pushed different commits to the same branch on the remote. The histories have diverged.

### How to fix:
Integrate the two histories:
* **Option A: Merge** (Creates a merge commit combining the histories):
  ```bash
  git pull origin main
  ```
* **Option B: Rebase** (Reapplies your local commits on top of the remote commits):
  ```bash
  git pull --rebase origin main
  ```

### Prevention:
For shared branches, fetch and pull frequently. Work on isolated feature branches instead of pushing to shared branches directly.

---

## 6. LF vs. CRLF Line Ending Warnings

```
warning: LF will be replaced by CRLF in src/main.py.
The file will have its original line endings in your working directory.
```

### Why it happens:
Windows uses Carriage Return + Line Feed (`CRLF`, `\r\n`) to represent new lines, while macOS and Linux use Line Feed (`LF`, `\n`). When developers mix operating systems, Git warns you about converting these characters. If not configured correctly, files will show massive, noisy diffs where every line appears modified.

### How to fix:
Configure `core.autocrlf` globally on your machine:
* **Windows:**
  ```bash
  git config --global core.autocrlf true
  ```
* **macOS / Linux:**
  ```bash
  git config --global core.autocrlf input
  ```

### Prevention:
Add a `.gitattributes` file to the root of your repository to enforce line-ending behavior for everyone on your team:
```text
# Enforce LF endings for all text files
* text=auto
*.py text eol=lf
*.js text eol=lf
```

---

## 7. Remote Already Exists

```
fatal: remote origin already exists.
```

### Why it happens:
You ran `git remote add origin <url>`, but the shorthand name `origin` is already assigned to a URL in this repository.

### How to fix:
* **Update the existing URL:**
  ```bash
  git remote set-url origin <new-url>
  ```
* **Delete the old remote first:**
  ```bash
  git remote remove origin
  git remote add origin <url>
  ```

---

## 8. Not a Git Repository

```
fatal: not a git repository (or any of the parent directories): .git
```

### Why it happens:
You ran a Git command in a directory that has not been initialized with `git init`, or you are in the parent directory of your cloned folder and forgot to navigate into it.

### How to fix:
Verify your active folder path:
```bash
pwd
# Navigate to the repository directory
cd my-project-folder
```

---

## 9. Refspec Error

```
error: src refspec main does not match any.
error: failed to push some refs to 'github.com:user/repo.git'
```

### Why it happens:
You ran `git push origin main`, but the branch `main` does not exist locally. This typically happens because:
* You initialized a local repository but have not made your first commit yet.
* Your default local branch is named `master` instead of `main`.

### How to fix:
1. Verify if you have made commits: `git log`. If not, commit a file first.
2. Check your active branch name:
   ```bash
   git branch
   ```
3. Rename your local branch to match GitHub's default if it is named `master`:
   ```bash
   git branch -M main
   ```
4. Push: `git push -u origin main`.

---

## 10. Merge Message Swap File Warning

```
Found a swap file by the name ".git/.MERGE_MSG.swp"
owned by: user ...
```

### Why it happens:
While Git was waiting for you to type a merge or commit message in Vim, the terminal window crashed, closed forcefully, or crashed the process. Vim left a hidden swap file (`.MERGE_MSG.swp`) behind in your administrative folder.

### How to fix:
Delete the swap file manually:
* **Windows (PowerShell):**
  ```powershell
  Remove-Item .git/.MERGE_MSG.swp
  ```
* **macOS / Linux:**
  ```bash
  rm .git/.MERGE_MSG.swp
  ```

---

## What Comes Next

Errors are part of the daily developer experience. Knowing how to diagnose and resolve them keeps you productive and confident.

Now that we have covered troubleshooting, let's establish professional workflows. In **Chapter 16 — Best Practices**, we will detail the core coding standards for Git: structuring commit messages, choosing branch naming strategies, handling pull request integration, and optimizing repository structures.
