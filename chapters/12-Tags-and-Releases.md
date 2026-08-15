# Chapter 12 — Tags & Releases

## Introduction

As your project grows, you will reach milestones that represent stable versions of your software—such as launching a beta version, shiping a version to customers, or deploying an official release. While you can mark these moments by writing a commit message like "Deploy version 1.0.0," finding that specific commit later in a log of thousands of entries is tedious and error-prone.

Git provides a feature specifically for marking milestones: **tags**. A tag is an immutable bookmark that points to a specific commit. Unlike branches, which constantly move forward as you write new commits, a tag stays locked to its target commit forever. 

Collaboration platforms like **GitHub** build on top of Git tags to create **Releases**, allowing you to publish release notes, upload packaged installers or binary files, and mark versions as draft or pre-release.

This chapter details the differences between branches and tags, how to create and manage local and remote tags, the principles of Semantic Versioning (SemVer), and the creation of GitHub Releases.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain the structural difference between branches and tags in Git
- Differentiate between lightweight tags and annotated tags
- Create, list, inspect, and delete tags locally
- Push local tags to remote servers and delete remote tags
- Apply the principles of Semantic Versioning (SemVer) to your release cycle
- Build and publish GitHub Releases with release notes and downloadable assets

---

## Tags vs. Branches

While both branches and tags are pointers to commits, they serve completely different purposes:

```
        HEAD
         │
         ▼
       main ─────────────────────► [Commit D] (Branch moves forward)
                                        ▲
                                        │
 v1.0.0 (Tag) ───► [Commit B] ──────────┘ (Tag remains locked here)
```

* **Branch (Mutable):** A moving target. It represents an active line of development. When you check out a branch and commit, the branch pointer automatically moves to the new commit.
* **Tag (Immutable):** A static snapshot. It represents a history milestone. Once created, a tag remains locked to its specific commit hash. If you check out a tag and commit, you enter a "detached HEAD" state, and the tag does not move.

---

## 1. Types of Git Tags

Git supports two types of tags, depending on how much metadata you need to store.

### A. Lightweight Tags
A lightweight tag is simply a pointer to a commit. It contains no additional metadata—no author, no date, no message. It is essentially a private, temporary bookmark.

```bash
git tag v1.0.0-lw
```

### B. Annotated Tags (Recommended for Releases)
Annotated tags are stored as full objects in the Git database. They are checksummed and contain:
* The tagger's name and email
* The date and time the tag was created
* A tagging message (similar to a commit message)
* Optional GPG cryptographic signatures for verification

```bash
git tag -a v1.0.0 -m "Release version 1.0.0 with user auth"
```
*(The `-a` flag specifies "annotated," and `-m` provides the message).*

---

## 2. Managing Tags Locally

### A. Listing Tags
To see all tags in your repository:
```bash
git tag
```

To search for tags matching a specific pattern (e.g., all version 1 tags):
```bash
git tag -l "v1.*"
```

### B. Inspecting Tag Details
To see the tagging metadata, the author, the date, and the commit information associated with a tag:

```bash
git show v1.0.0
```

**Expected output (Annotated Tag):**
```
tag v1.0.0
Tagger: Jane Doe <jane.doe@example.com>
Date:   Sat Aug 15 20:57:13 2026 +0545

Release version 1.0.0 with user auth

commit 7c8d9e0f1g2h3i4j5k6l7m8n9o0p1q2r3s4t5u6v (HEAD -> main)
Author: Jane Doe <jane.doe@example.com>
Date:   Fri Aug 14 18:32:00 2026 +0545

    Implement JWT validation middleware
```

### C. Tagging Past Commits
If you forgot to tag a release commit three days ago, you can tag it retroactively by specifying the commit hash:

```bash
git tag -a v0.9.0 4a9f1b2 -m "Beta release 0.9.0"
```

### D. Deleting a Local Tag
If you made a typo or tagged the wrong commit locally:

```bash
git tag -d v1.0.0
```

---

## 3. Syncing Tags with Remotes

By default, Git's `git push` command only uploads commits; **it does not push tags** to remote servers. You must transfer them explicitly.

### A. Pushing a Single Tag
```bash
git push origin v1.0.0
```

### B. Pushing All Local Tags
To upload all tags that exist locally but do not exist on the remote:

```bash
git push origin --tags
```

### C. Deleting a Remote Tag
If you delete a tag locally and need to remove it from GitHub:

```bash
git push origin --delete v1.0.0
```

---

## 4. Semantic Versioning (SemVer)

When naming your tags, you should follow **Semantic Versioning (SemVer)**. This is a universally accepted standard for version numbering that communicates the nature of code changes to users and dependencies.

The SemVer format is: `MAJOR.MINOR.PATCH` (e.g., `v2.4.1`)

```
   v2 . 4 . 1
   │    │   │
   │    │   └─── PATCH: Backward-compatible bug fixes
   │    └─────── MINOR: Backward-compatible new features
   └──────────── MAJOR: Incompatible API changes (breaking changes)
```

1. **MAJOR version** changes when you make incompatible API updates (users must rewrite parts of their code to upgrade).
2. **MINOR version** changes when you add features in a backward-compatible manner (users can upgrade without breaking their code).
3. **PATCH version** changes when you apply backward-compatible bug fixes.

### Pre-releases:
If you want to tag a test version before an official launch, append a hyphen and a label (e.g., `v1.0.0-alpha.1`, `v1.0.0-beta.3`, `v1.0.0-rc.2`).

---

## 5. GitHub Releases

On GitHub, a **Release** builds on top of a Git tag. It provides a user-friendly interface where users can view release notes and download software packages.

```
       Git Tag (e.g., v1.0.0)
                │
                ▼ (Created in Git)
         GitHub Release
     ┌────────────────────────┐
     │  Release Notes         │
     │  - Changelog           │
     │  - Contributors        │
     │  Assets (Downloadable) │
     │  - source_code.zip     │
     │  - installer.exe       │
     └────────────────────────┘
```

### How to Create a Release on GitHub:
1. Push your tag to GitHub: `git push origin v1.0.0`.
2. Go to your repository on GitHub and click **Releases** on the right sidebar.
3. Click **Draft a new release**.
4. Select the tag you just pushed (or create a new tag on the fly).
5. Write a **Release title** (e.g., `v1.0.0 - Authentication Release`).
6. Write a markdown description detailing the changelog (features added, bugs fixed). GitHub can auto-generate this list by clicking **Generate release notes**.
7. (Optional) Drag and drop binary assets (like compiled `.exe`, `.dmg`, or `.zip` application bundles) into the upload box.
8. If this is a test version, check the **Set as a pre-release** box.
9. Click **Publish release**.

---

## Tags & Releases Command Quick Reference

| Command | Action |
|---|---|
| `git tag` | List all local tags. |
| `git tag -a <tag> -m "msg"` | Create a new local annotated tag. |
| `git tag <tag>` | Create a new local lightweight tag. |
| `git show <tag>` | Inspect tag details and metadata. |
| `git tag -d <tag>` | Delete a tag locally. |
| `git push origin <tag>` | Push a specific tag to remote. |
| `git push origin --tags` | Push all local tags to remote. |
| `git push origin --delete <tag>` | Delete a tag on remote. |

---

## What Comes Next

Tagging and releases establish stable versions of your codebase. However, there are always files generated during local development (like dependency downloads, temporary compile caches, or secrets) that should never make it into your Git history or tags.

In **Chapter 13 — Git Ignore**, we will explore how to configure `.gitignore` files to prevent unwanted files from cluttering your repository.
