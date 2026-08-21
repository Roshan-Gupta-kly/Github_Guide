# Chapter 17 — Open Source Workflow

## Introduction

So far, we have assumed you are working inside repositories where you have direct write (push) permissions. In team environments and private repositories, this is standard. However, when contributing to open-source software (OSS) or repositories owned by other organizations, you will not have permission to push branches directly to the remote server.

To contribute to these projects, you must use the **Forking Workflow**. Instead of working on the main repository, you create a personal copy in the cloud, make your changes there, and submit those changes to the original project via a Pull Request.

This chapter details the mechanics of the Forking Workflow: explaining forks, setting up multiple remote tracking pointers (`origin` and `upstream`), syncing your local work with parent project updates, and submitting contributions.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain the difference between a Fork and a Clone
- Fork a public repository on GitHub and clone it locally
- Configure and manage multiple remote targets (`origin` vs. `upstream`)
- Sync your fork with changes made to the parent repository
- Submit contributions using Pull Requests from your fork to the parent repository

---

## Fork vs. Clone

Before starting, it is crucial to understand how forks and clones differ:

* **Fork (Server-to-Server Copy):** A fork is a personal copy of a repository that lives on GitHub’s servers, associated with your personal account. It is created entirely in the cloud. You have full read/write permissions to your fork.
* **Clone (Server-to-Local Copy):** A clone is a copy of a repository downloaded to your local computer. It creates a local `.git` directory so you can write code.

```
   Parent Repository (GitHub)
   [owner/project-repo]
            │
            │ (Fork)
            ▼
    Your Remote Fork (GitHub) ◄─── Named `origin`
   [your-username/project-repo]
            │
            │ (Clone)
            ▼
      Local Computer ◄──────────── Named `upstream` (connected to parent)
```

To contribute, you fork the parent project to your account, clone your fork to your computer, add a connection to the parent repository, write code, push to your fork, and request a merge.

---

## 1. Setting Up the Forking Workflow

### Step 1: Fork the Project
1. Navigate to the repository of the open-source project you want to contribute to on GitHub.
2. Click the **Fork** button in the upper-right corner of the page.
3. Select your personal account as the destination. GitHub will copy the repository to your account.

### Step 2: Clone Your Fork Locally
Copy the clone URL (SSH or HTTPS) of **your personal fork**, and run:

```bash
git clone git@github.com:your-username/project-repo.git
cd project-repo
```

---

## 2. Configuring Remotes: `origin` vs. `upstream`

When you clone your repository, Git automatically registers a remote named `origin` pointing to your fork. To get updates from the original repository (the parent project), you must register it as a second remote, conventionally named **`upstream`**.

```bash
git remote add upstream git@github.com:original-owner/project-repo.git
```

### Verify Your Remotes
Check your remote connections to ensure both are registered correctly:

```bash
git remote -v
```

**Expected output:**
```
origin    git@github.com:your-username/project-repo.git (fetch)
origin    git@github.com:your-username/project-repo.git (push)
upstream  git@github.com:original-owner/project-repo.git (fetch)
upstream  git@github.com:original-owner/project-repo.git (push)
```

---

## 3. Syncing Your Fork with the Parent Project

Open-source projects move quickly. Before you start writing code, and before you submit a Pull Request, you must sync your local repository with the `upstream` repository to prevent merge conflicts.

### The Syncing Steps:

1. **Switch to your default branch:**
   ```bash
   git switch main
   ```
2. **Fetch updates from the original parent project:** Downloads commits from `upstream` but does not modify your working files:
   ```bash
   git fetch upstream
   ```
3. **Merge upstream updates into your local branch:**
   ```bash
   git merge upstream/main
   # OR: git rebase upstream/main
   ```
4. **Push the synchronized history to your GitHub fork:**
   ```bash
   git push origin main
   ```

---

## 4. Contributing: Submitting a Pull Request

Once your repository is synchronized, you can write and submit your changes.

### Step 1: Create a Feature Branch
Always work on a branch. Never make commits directly to your `main` branch:

```bash
git switch -c feature/fix-docs-typo
```

### Step 2: Write Code and Commit
Make your modifications, stage them, and commit following Conventional Commit guidelines:

```bash
git add README.md
git commit -m "docs: correct installation command typo"
```

### Step 3: Push to Your Fork
Upload the branch to **your** remote repository (`origin`):

```bash
git push -u origin feature/fix-docs-typo
```

### Step 4: Open the Pull Request on GitHub
1. Go to the original parent repository page on GitHub (`original-owner/project-repo`).
2. GitHub will display a banner showing that you recently pushed a branch to your fork. Click **Compare & pull request**.
3. Select the target repository and branch:
   * **base repository:** `original-owner/project-repo` (base branch: `main`)
   * **head repository:** `your-username/project-repo` (compare branch: `feature/fix-docs-typo`)
4. Fill in the PR description template clearly and click **Create pull request**.

---

## Forking Workflow Quick Reference

```bash
# 1. Clone your fork
git clone git@github.com:your-username/project-repo.git
cd project-repo

# 2. Add parent repository connection
git remote add upstream git@github.com:original-owner/project-repo.git

# 3. Fetch and sync upstream updates
git fetch upstream
git switch main
git merge upstream/main
git push origin main

# 4. Create feature branch and push edits
git switch -c feature/my-patch
# (Make edits and commit)
git push -u origin feature/my-patch
```

---

## What Comes Next

Contributing to open source expands your engineering reach. As repositories grow, teams automate code quality, build checks, and testing suites.

In **Chapter 18 — GitHub Actions (CI/CD)**, we will explore how to set up automated pipelines that build, test, and deploy your code automatically whenever you push commits or open Pull Requests.
