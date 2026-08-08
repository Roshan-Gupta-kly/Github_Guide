# Chapter 05 — Daily Workflow

## Introduction

Once your repository is initialized and connected to GitHub, you enter the active development phase. In professional software engineering, you will repeat the same fundamental Git actions dozens of times a day: writing code, staging changes, committing updates, reviewing history, and syncing with your team. 

This daily developer loop is the heart of version control. However, many developers execute these commands on autopilot, leading to messy history, accidental commits of temporary files, or unnecessary merge conflicts. This chapter details Git's daily workflow commands—`status`, `diff`, `add`, `commit`, `log`, `fetch`, `pull`, and `push`—focusing on best practices, internal mechanics, and professional-grade options that make you a more efficient engineer.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Visualize and explain the core daily Git loop (the developer cycle)
- Use `git status` to inspect tracked and untracked changes, including its short-format view
- Compare modifications across different states (working directory, staging area, and repository) using `git diff`
- Stage changes selectively and interactively using `git add -p`
- Write high-quality, professional commit messages following the 50/72 character rule
- Navigate and filter project history using `git log` with advanced formatting flags
- Differentiate between `git fetch` and `git pull`, explaining why `fetch` is safer
- Safely push local commits and pull remote updates to keep your project synchronized

---

## The Daily Git Loop

The daily workflow of a software developer follows a continuous cycle:

```
            ┌────────────────────────────────────────┐
            ▼                                        │
     1. Modify Files (Working Directory)             │
            │                                        │
            ▼                                        │
     2. Inspect Status & Diff (`status`, `diff`)     │ (Iterate)
            │                                        │
            ▼                                        │
     3. Stage Selected Changes (`add`)               │
            │                                        │
            ▼                                        │
     4. Commit Changes (`commit`) ───────────────────┘
            │
            ▼
     5. Sync with Remote (`fetch`, `pull`, `push`)
```

---

## 1. Inspecting the Workspace: `git status`

The first command you should run before staging or committing is `git status`. It answers the question: *What is the state of my repository right now?*

```bash
git status
```

### Understanding the Three Output Sections
When you run `git status`, Git categorizes files into three main sections:

1. **Changes to be committed:** Files that are in the Staging Area. These will be included in your next commit.
2. **Changes not staged for commit:** Tracked files that have been modified in the Working Directory but have not yet been moved to the Staging Area.
3. **Untracked files:** New files that Git has never tracked before.

### Pro-Tip: The Short Status Syntax
When working in large repositories with many files, the default `git status` output can be verbose. Use the `-s` or `--short` flag for a compact summary:

```bash
git status -s
```

**Example output:**
```
M  README.md
 M src/main.py
?? config/temp.json
A  src/utils.py
```

**How to read the two-column status codes:**
* **Left Column (Staging Area status):** Represents changes staged for the next commit.
  - `M ` = Modified in working directory AND staged.
  - `A ` = Newly added to staging area.
* **Right Column (Working Directory status):** Represents changes not yet staged.
  - ` M` = Modified in working directory but NOT staged.
  - `??` = Untracked file.

---

## 2. Comparing Changes: `git diff`

`git status` tells you *which* files changed, but `git diff` tells you *exactly what lines* changed inside those files.

```
       Working Directory
               │
               │  git diff (Unstaged changes)
               ▼
          Staging Area
               │
               │  git diff --staged (Staged changes)
               ▼
        Local Repository (HEAD Commit)
```

### A. Comparing Working Directory vs. Staging Area
To see modifications that you have written but have not yet staged:

```bash
git diff
```

### B. Comparing Staging Area vs. Last Commit
To see changes that you have staged and are ready to commit:

```bash
git diff --staged
# OR
git diff --cached
```

### C. Comparing Working Directory vs. Last Commit
To see all changes (staged and unstaged) compared to the last commit:

```bash
git diff HEAD
```

### D. Comparing Specific Files
To limit the comparison to a specific file or directory:

```bash
git diff path/to/file.py
```

---

## 3. Staging Changes: `git add`

Staging is the process of preparing a snapshot. It allows you to commit only a subset of your changes, keeping commits clean and thematic.

### Staging Methods:
* **Stage a single file:**
  ```bash
  git add path/to/file.txt
  ```
* **Stage all files in the current folder (recursively):**
  ```bash
  git add .
  ```
* **Stage all modified, deleted, and untracked files in the whole repo:**
  ```bash
  git add -A
  ```

### Interactive Staging: `git add -p` (The Professional Way)
If you modified a single file but want to split those modifications into two separate commits (e.g., you fixed a bug AND added documentation in the same file), use the patch flag:

```bash
git add -p filename.py
```

Git will break your file changes down into small chunks (hunks) and ask you how to handle each one:

```
Stage this hunk? [y,n,q,a,d,s,e,?]:
```

**Common options:**
* `y` — Stage this hunk.
* `n` — Do not stage this hunk.
* `s` — Split the current hunk into smaller hunks.
* `q` — Quit interactive mode (saves whatever you staged so far).

---

## 4. Recording History: `git commit`

Committing takes the staged snapshot and writes it permanently into the Git database.

```bash
git commit -m "Fix login button click alignment"
```

### Writing Professional Commit Messages: The 50/72 Rule
In professional environments, commit logs must be clean and readable. Follow this standard template:

1. **Subject Line (under 50 characters):** A short summary written in the **imperative mood** (e.g., "Add feature X" instead of "Added feature X" or "Adds feature X"). Start with a capital letter, and do not end with a period.
2. **Blank Line:** Separates the subject from the body.
3. **Body (wrapped at 72 characters):** Explains the *why* and *what*, not the *how* (the code shows how; the commit message explains the reasoning and context).

**Example of a professional commit message:**
```
Add validation for customer email domain on sign-up

We received reports that users could register with invalid domains.
This change uses regex to check the domain suffix against the MX
records list before allowing account creation.

Closes #482
```

### Amending the Last Commit: `git commit --amend`
If you committed your changes but realized you made a typo in your commit message, or forgot to add a small change to a file:

1. Stage the missed change: `git add missed-file.py`
2. Amend the commit:
   ```bash
   git commit --amend
   ```
This will open your text editor, allowing you to edit the commit message. Git will replace the last commit with a brand new commit containing the corrected content.

> **Caution:** Only amend commits that have **not** been pushed to GitHub. Rewriting shared history can break the workspace for your collaborators.

---

## 5. Reviewing History: `git log`

To see the list of commits made in the repository, use `git log`.

### Standard Log
```bash
git log
```
*Shows author, email, date, SHA-1 hash, and the full commit message.*

### The Professional "One-Line Graph"
The default `git log` output takes up a lot of screen space. To see a clean, tree-like visualization of your branches and commits:

```bash
git log --oneline --graph --decorate --all
```

**Visual Output Example:**
```
* 4a9f1b2 (HEAD -> main, origin/main) Fix email regex validation
* f7c3b9d Add user registration form
|\  
| * a1b2c3d (feature/oauth) Setup GitHub Login OAuth client
|/  
* 9e8d7c6 Initial commit
```

### Filtering History
* **Limit the number of commits:** `git log -n 5`
* **Filter by author:** `git log --author="Jane Doe"`
* **Filter by message content:** `git log --grep="bugfix"`
* **Filter changes for a specific file:** `git log --oneline -- path/to/file.py`

---

## 6. Remote Synchronization: `fetch` vs. `pull`

When collaborating, you must keep your local repository in sync with the remote repository on GitHub.

```
       Local Repository                    Remote Repository
┌────────────────────────────┐          ┌─────────────────────┐
│                            │   fetch  │                     │
│  [origin/main] ◄───────────┼──────────┼── [main]            │
│      │                     │          │                     │
│      │ merge               │          │                     │
│      ▼                     │   push   │                     │
│   [main] ──────────────────┼──────────┼─► [main]            │
└────────────────────────────┘          └─────────────────────┘
```

### A. Updating Remotes Safely: `git fetch`
`git fetch` downloads all new data from the remote repository to your local machine, updating remote-tracking branches (like `origin/main`). 

```bash
git fetch origin
```

**Key Benefit:** It does **not** change your local working files. It is 100% safe. You can inspect the downloaded changes before merging them:

```bash
git log HEAD..origin/main --oneline
```
*(This shows all commits that are on the remote branch but not on your local branch).*

### B. Syncing and Merging: `git pull`
`git pull` is a shortcut command. It runs `git fetch` and immediately runs `git merge` to integrate the remote changes into your active local branch.

```bash
git pull origin main
```

**Risk:** If the remote branch has changes that conflict with your local edits, `git pull` will pause and prompt you to resolve merge conflicts.

> **Best Practice:** Many professional teams prefer `git pull --rebase origin main` instead of a standard pull. This rebases your local commits on top of the newly fetched remote commits, maintaining a clean, linear git history.

### C. Pushing Local Commits: `git push`
Once you have committed your changes locally and are ready to share them with the world:

```bash
git push origin main
```

---

## Daily Workflow Command Reference

| Command | Purpose | When to Use |
|---|---|---|
| `git status -s` | Quick summary of modified files | Before running `git add` or `git commit`. |
| `git diff` | Shows unstaged changes | To double-check exactly what code lines you modified. |
| `git diff --staged` | Shows staged changes | To review the exact lines that are going into your next commit. |
| `git add -p` | Interactive hunk staging | To split multiple unrelated edits into separate, clean commits. |
| `git commit -m "msg"` | Saves staged changes locally | Once you finish a logical, tested unit of work. |
| `git log --oneline` | Clean single-line history view | To quickly inspect recent commits. |
| `git fetch` | Downloads remote metadata safely | To see if teammates pushed updates without modifying your files. |
| `git pull` | Downloads and merges updates | To bring your workspace up-to-date with remote work. |
| `git push` | Uploads local commits | When you are ready to share finished commits to GitHub. |

---

## What Comes Next

With the daily workflow down, you can edit and track files efficiently. However, working directly on the `main` branch becomes dangerous when collaborating or working on multiple features at the same time.

In **Chapter 06 — Branching**, you will learn how to create isolated parallel lines of development, rename branches, prune stale branches, and establish a professional branching strategy for your projects.
