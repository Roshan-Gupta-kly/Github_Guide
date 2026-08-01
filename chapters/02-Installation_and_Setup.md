# Chapter 02 — Installation and Setup

## Introduction

Every journey with Git begins the same way: getting it installed correctly and configured properly on your machine. This sounds trivial, but a surprising number of downstream problems — broken line endings, mismatched author names in commit history, authentication failures, "safe directory" errors — trace back to a rushed or incomplete setup process.

This chapter treats installation and configuration as a serious, professional step, not an afterthought. We will install Git across all three major operating systems, verify the installation, and then configure Git properly: identity, default branch name, line-ending behavior, editor, credential handling, and useful aliases. By the end of this chapter, your machine will be set up the way a professional engineer's machine is set up — not just "able to run `git status`," but correctly and safely configured for real collaborative work.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Install Git on Windows, macOS, and Linux
- Verify that Git is installed correctly and understand version output
- Configure your Git identity (name and email) at global and local scope
- Understand Git's three configuration levels: system, global, and local
- Configure line-ending behavior correctly for your operating system
- Set a default branch name and default editor
- Set up SSH authentication with GitHub
- Set up HTTPS authentication with a credential manager
- Troubleshoot the most common installation and configuration problems

## Prerequisites

- A computer running Windows, macOS, or Linux, with administrator/sudo access
- Basic comfort using a terminal, command prompt, or PowerShell
- A GitHub account (free) — if you do not have one yet, create one at github.com before continuing

---

## Theory: Why Configuration Matters Before You Ever Commit

Git attaches metadata to every single commit you make — most importantly, **who made it** (name and email) and **when**. This metadata is baked permanently into the commit's identity (we will explore exactly how in Chapter 16, Git Internals). If your identity is misconfigured, incorrect, or inconsistent, you end up with commit history that is confusing, unprofessional, or even impossible to attribute correctly — a serious problem in team environments and open-source projects where trust and accountability matter.

Similarly, line-ending configuration might seem like a minor detail, but it is one of the most common sources of frustrating, noisy diffs and false merge conflicts across teams that mix Windows and Unix-based systems. Getting this right *before* your first commit saves significant pain later.

```
 Correct Setup                         Incorrect Setup
 ───────────────                       ─────────────────
 Consistent author identity            "unknown" or wrong author on commits
 Clean, meaningful diffs               Noisy diffs full of line-ending changes
 Smooth cross-platform collaboration   Constant false merge conflicts
 Secure, convenient authentication     Repeated password prompts or failures
```

---

## Installing Git

### Installing Git on Windows

1. Go to [git-scm.com/download/win](https://git-scm.com/download/win). The download should start automatically with the recommended 64-bit installer.
2. Run the installer. During setup, the installer will present several configuration screens. For beginners, the defaults are generally safe, but the following choices are worth understanding:
   - **Adjusting your PATH environment**: Choose "Git from the command line and also from 3rd-party software" — this ensures Git is usable from Command Prompt, PowerShell, and other tools.
   - **Choosing the SSH executable**: Use the bundled OpenSSH unless you have a specific reason not to.
   - **Choosing the default editor for Git**: Beginners often prefer Notepad++ or VS Code if installed; Vim is the historical default but has a learning curve (we cover this later in this chapter).
   - **Line ending conversions**: Choose "Checkout Windows-style, commit Unix-style line endings" (this is `core.autocrlf=true`, discussed in detail below).
3. Complete the installation and open **Git Bash**, a Unix-like terminal emulator installed alongside Git on Windows. Most of this handbook's command examples will work identically in Git Bash, PowerShell, or Command Prompt.

### Installing Git on macOS

macOS offers several installation methods:

**Option 1 — Homebrew (recommended for most developers):**

```bash
brew install git
```

**Option 2 — Xcode Command Line Tools:**

```bash
git --version
```

If Git is not already installed, running this command on a fresh macOS install will prompt you to install the Xcode Command Line Tools, which include Git.

**Option 3 — Official installer:**

Download the `.dmg` package directly from [git-scm.com/download/mac](https://git-scm.com/download/mac) and follow the installer prompts.

> **Best Practice:** Prefer Homebrew where possible. It makes future Git upgrades a simple `brew upgrade git`, rather than requiring you to manually reinstall.

### Installing Git on Linux

Linux distributions typically ship Git through their native package managers.

**Debian / Ubuntu:**

```bash
sudo apt update
sudo apt install git
```

**Fedora:**

```bash
sudo dnf install git
```

**Arch Linux:**

```bash
sudo pacman -S git
```

> **Best Practice:** Package manager installations are generally preferred on Linux because they integrate with your system's update cycle and dependency management.

---

## Verifying Installation

Regardless of platform, verify the installation the same way:

```bash
git --version
```

**Purpose:** Confirms Git is installed and prints the installed version number.

**Expected output (example):**

```
git version 2.45.1
```

**Common issues:**

- `command not found: git` (macOS/Linux) or `'git' is not recognized...` (Windows) — Git is not installed, or not added to your system's PATH. Reinstall and ensure the PATH option is selected (Windows) or reopen your terminal after installation.
- An old version number — many Linux distributions ship older Git versions in their default repositories. Consider adding the official Git PPA (Ubuntu) or using Homebrew (macOS) for a more current release.

> **Best Practice:** Keep Git reasonably up to date. Newer versions include performance improvements, security patches, and quality-of-life features like improved conflict markers and `--force-with-lease` safety defaults.

---

## Understanding Git's Configuration Levels

Git configuration exists at **three levels**, each overriding the one before it in scope:

```
 System Level                Global Level                 Local Level
 ─────────────                ─────────────                ─────────────
 Applies to ALL users         Applies to YOUR user          Applies to ONE
 on the machine                account, all repos            specific repository
 
 --system                      --global                      --local (default)
 
 Lowest priority               Medium priority               Highest priority
```

**File locations:**

| Scope | Typical Location |
|---|---|
| System | `/etc/gitconfig` (Linux/macOS) or Git installation directory (Windows) |
| Global | `~/.gitconfig` or `~/.config/git/config` |
| Local | `.git/config` inside a specific repository |

**Why this matters:** You can set a global identity for personal projects, but override it locally for a specific repository — for example, using your work email only inside your company's repositories.

```bash
git config --list --show-origin
```

**Purpose:** Shows every active configuration value and which file it came from.
**When to use:** Whenever you are unsure why Git is behaving a certain way — this command reveals exactly which configuration level is responsible.

---

## Configuring Your Identity

This is the single most important configuration step. Run these two commands before making your first commit anywhere:

```bash
git config --global user.name "Your Full Name"
git config --global user.email "your.email@example.com"
```

**What it does:** Sets the name and email that will be permanently attached to every commit you make on this machine, unless overridden locally.

**Why we use it:** Git refuses to let commit metadata be blank or ambiguous — this identity is how collaborators, reviewers, and tools like GitHub attribute work to you. GitHub also uses this email to link your commits to your GitHub profile (if it matches a verified email on your account).

**When to use it:** Immediately after installation, and again with `--local` inside any repository where you need a different identity (e.g., a work repository using your company email).

**What happens internally:** These values are written into your `.gitconfig` file as plain text, and later embedded directly into each commit object's metadata when you run `git commit` (explored fully in Chapter 16).

**Example — setting a different identity for one repository:**

```bash
cd my-work-project
git config --local user.email "you@company.com"
```

> **Common Mistake:** Forgetting to set an identity before committing. Git will either block the commit with an error or, in some configurations, fall back to a guessed identity based on your system username and hostname — producing embarrassing commits like `runner@DESKTOP-8X2KQ1L`.

> **Best Practice:** Use the **same email** for your Git config and your GitHub account (or add it as a verified secondary email on GitHub) so your commits are correctly linked to your GitHub profile and contribution graph.

---

## Configuring the Default Branch Name

Modern Git (2.28+) allows you to set the name used for the first branch in any new repository. The historical default was `master`; the modern, widely adopted default across the industry — including GitHub itself — is `main`.

```bash
git config --global init.defaultBranch main
```

**What it does:** Any new repository created with `git init` from now on will start on a branch named `main` instead of `master`.

**Why we use it:** Aligning with GitHub's default avoids friction and confusion when your local default branch name does not match the remote's expected default.

**When to use it:** Once, immediately after installation.

---

## Configuring Line Endings (core.autocrlf)

Different operating systems historically use different characters to represent a line break in a text file:

```
Windows:        CRLF   (\r\n)
macOS / Linux:  LF     (\n)
```

When teams mix operating systems, this difference can cause Git to think entire files have changed when only line endings differ — producing enormous, meaningless diffs and false merge conflicts.

**Windows:**

```bash
git config --global core.autocrlf true
```

This converts LF to CRLF when checking files out, and back to LF when committing — ensuring the repository itself always stores the Unix-style LF standard.

**macOS / Linux:**

```bash
git config --global core.autocrlf input
```

This leaves line endings untouched on checkout but converts CRLF to LF on commit, in case a file with Windows line endings is introduced.

```
                 core.autocrlf
        ┌─────────────────────────────┐
        │                              │
   Working Directory  ◄──────────►  Repository
   (OS-native endings)             (always LF)
```

> **Common Mistake:** Leaving `core.autocrlf` unset on a team that mixes Windows and Unix-based machines. This is one of the most frequent causes of "why does this diff show the whole file changed?" confusion for beginners.

> **Best Practice:** Combine `core.autocrlf` with a `.gitattributes` file in the repository itself (covered in Chapter 19) for the most reliable, team-wide consistency — configuration alone is per-machine and easy for a teammate to forget.

---

## Configuring Your Default Editor

Certain Git operations — writing a commit message without `-m`, resolving a rebase, editing an interactive rebase plan — open a text editor. Git's historical default is **Vim**, which is powerful but unfamiliar to many beginners.

**Setting VS Code as your Git editor:**

```bash
git config --global core.editor "code --wait"
```

**Setting Nano (a simpler terminal editor):**

```bash
git config --global core.editor "nano"
```

**What it does:** Determines which program opens when Git needs you to write or edit a text file interactively.

**Why we use it:** Getting unexpectedly dropped into Vim, without knowing how to save and exit (`Esc`, then `:wq`, then Enter), is a famous source of beginner panic.

> **Troubleshooting:** If you ever find yourself stuck inside Vim unexpectedly, press `Esc`, then type `:wq` and press Enter to save and quit, or `:q!` and press Enter to quit without saving.

---

## Setting Up Authentication with GitHub

To push code to GitHub, you need to authenticate. There are two primary methods: **SSH** and **HTTPS with a credential manager**. SSH is the professional standard for most engineers; HTTPS with a credential helper is simpler for beginners and works well too.

### Method 1: SSH Authentication (Recommended)

**Step 1 — Generate an SSH key:**

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
```

**What it does:** Generates a new public/private key pair using the modern, secure Ed25519 algorithm. Press Enter to accept the default file location, and optionally set a passphrase for extra security.

**Step 2 — Start the SSH agent and add your key:**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Step 3 — Copy your public key:**

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output.

**Step 4 — Add the key to GitHub:**

Go to **GitHub → Settings → SSH and GPG keys → New SSH key**, paste the copied key, and save.

**Step 5 — Test the connection:**

```bash
ssh -T git@github.com
```

**Expected output:**

```
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

```
     Your Machine                              GitHub
   ┌───────────────┐         SSH Key         ┌──────────────┐
   │ Private Key    │ ──────────────────────► │ Public Key    │
   │ (never shared) │        Handshake        │ (stored on    │
   │                │ ◄────────────────────── │  your account)│
   └───────────────┘                          └──────────────┘
```

> **Best Practice:** Never share your private key. Only the public key (`.pub` file) should ever be uploaded anywhere, including GitHub.

### Method 2: HTTPS with a Credential Manager

If SSH feels intimidating at first, HTTPS is a perfectly valid alternative:

```bash
git config --global credential.helper manager
```

**What it does:** Configures Git to use your operating system's credential manager (Git Credential Manager on Windows, Keychain on macOS, or `libsecret` on Linux) to securely cache your GitHub credentials after your first successful authentication, so you are not prompted every time.

**Note:** As of 2021, GitHub no longer accepts account passwords for Git operations over HTTPS. You must use a **Personal Access Token (PAT)** in place of a password, generated from **GitHub → Settings → Developer settings → Personal access tokens**.

> **Common Mistake:** Trying to authenticate over HTTPS with your GitHub account password and receiving a confusing authentication failure. This almost always means a Personal Access Token is required instead.

---

## Useful Global Configuration Additions

```bash
git config --global color.ui auto
git config --global core.pager "less -FRX"
git config --global pull.rebase false
```

- `color.ui auto` — enables colored output for commands like `status`, `diff`, and `log`, making terminal output significantly easier to read.
- `core.pager` — controls how long output (like `git log`) is displayed; this setting improves default scrolling behavior.
- `pull.rebase false` — explicitly sets the default strategy for `git pull` to merge rather than rebase, avoiding an interactive Git warning on newer versions. (We explore the merge-vs-rebase trade-off fully in Chapter 08.)

### Setting Up Aliases

Git allows you to create shortcuts for frequently used commands:

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
git config --global alias.lg "log --oneline --graph --all"
```

**What it does:** Creates shorter, custom command names — `git st` now behaves identically to `git status`.

**Why we use it:** Reduces typing friction for commands used dozens of times per day, especially useful once you are comfortable with the base commands (Chapter 22, the Cheat Sheet, includes a curated list of professional-grade aliases).

> **Best Practice:** Don't over-alias too early. Learn the full command names first so you genuinely understand what each alias is shortening — aliases are an efficiency layer, not a replacement for understanding.

---

## Verifying Your Full Setup

Run this checklist after completing setup:

```bash
git --version
git config --global user.name
git config --global user.email
git config --global init.defaultBranch
ssh -T git@github.com
```

If each command returns a sensible value (a version number, your name, your email, `main`, and a successful GitHub authentication message), your machine is fully and correctly set up for professional Git and GitHub work.

---

## Best Practices

- Always configure `user.name` and `user.email` immediately after installing Git, before your first commit.
- Prefer SSH authentication for daily professional work; it is more secure and does not require re-entering tokens.
- Set `init.defaultBranch` to `main` to match GitHub's convention and avoid confusion when pushing new repositories.
- Configure `core.autocrlf` correctly for your operating system before working on any cross-platform team.
- Keep Git updated to a reasonably recent version to benefit from security fixes and UX improvements.
- Document your team's expected Git configuration (editor, line endings, hooks) in your project's README or CONTRIBUTING guide.

## Common Mistakes

- Committing before setting `user.name` and `user.email`, resulting in incorrectly attributed or rejected commits.
- Mixing personal and work email addresses without using local (`--local`) configuration overrides per repository.
- Ignoring line-ending configuration on a mixed Windows/Unix team, leading to noisy diffs and false conflicts.
- Attempting HTTPS authentication with a GitHub account password instead of a Personal Access Token.
- Getting stuck in Vim with no idea how to exit, due to never configuring a preferred editor.

## Troubleshooting

**Problem:** `git: command not found` after installation.
**Fix:** Reopen your terminal (PATH changes often require a fresh shell session), or reinstall ensuring the "Add to PATH" option was selected.

**Problem:** `Permission denied (publickey)` when pushing to GitHub over SSH.
**Fix:** Confirm your SSH key was added to the SSH agent (`ssh-add -l` to list loaded keys) and that the matching public key is uploaded to your GitHub account settings.

**Problem:** Every `git pull` or `git push` prompts for a username and password, and the password is rejected.
**Fix:** GitHub requires a Personal Access Token instead of your account password for HTTPS authentication — generate one from GitHub's Developer Settings and use it in place of your password.

**Problem:** Diffs show entire files as changed even though you only edited one line.
**Fix:** This is almost always a line-ending mismatch. Set `core.autocrlf` correctly for your OS and consider adding a `.gitattributes` file (Chapter 19) to enforce consistency across the whole team.

---

## Summary

- Git can be installed on Windows, macOS, and Linux through official installers, Homebrew, or native package managers, and verified with `git --version`.
- Git configuration operates at three levels — system, global, and local — with local taking the highest priority.
- Setting `user.name` and `user.email` is the most critical first configuration step, since this metadata is permanently attached to every commit.
- `init.defaultBranch main` aligns your local defaults with GitHub's convention.
- `core.autocrlf` must be configured correctly for your operating system to avoid line-ending chaos on cross-platform teams.
- SSH authentication is the professional standard for connecting to GitHub; HTTPS with a Personal Access Token is a valid alternative.
- A properly configured editor, sensible aliases, and color output all reduce daily friction once you begin working with Git regularly.

---

## Interview Questions

**Beginner**

1. How do you verify that Git has been installed correctly?
2. What two configuration values should you set immediately after installing Git, and why?
3. What is the difference between global and local Git configuration?

**Intermediate**

4. Why does line-ending configuration matter, and what does `core.autocrlf` do?
5. What is the difference between SSH and HTTPS authentication methods for GitHub?
6. Why can't you use your GitHub account password for HTTPS Git operations anymore?

**Advanced**

7. Explain what happens internally when you run `git config --global user.email`. Where is this value stored, and how is it later used inside a commit object?
8. How would you configure a single machine to use two different Git identities for personal and work repositories?

**Scenario-Based**

9. A teammate on Windows reports that every file in a pull request shows as fully changed, even though they only modified a few lines. What is the likely cause, and how would you fix it for the whole team going forward?
10. You just cloned a new company repository and need to push using your work email instead of your personal one, without changing your global configuration. What commands would you run?

---

## Exercises

1. Install Git on your machine and run `git --version` to confirm success.
2. Configure your global `user.name` and `user.email`, then run `git config --list --show-origin` to confirm the values and their source file.
3. Set your `init.defaultBranch` to `main`.
4. Generate a new SSH key, add it to your GitHub account, and verify the connection with `ssh -T git@github.com`.
5. Create at least two custom aliases of your own choosing and test that they work correctly.
6. Intentionally set `user.email` to a second, different value using `--local` inside a test repository, then confirm with `git config --local user.email` that it overrides your global setting within that folder only.

---

## References

- Official Git Downloads — git-scm.com/downloads
- Pro Git Book, Chapter 1.6 "Getting Started - First-Time Git Setup" — git-scm.com/book
- GitHub Docs — "Connecting to GitHub with SSH" — docs.github.com
- GitHub Docs — "Managing your personal access tokens" — docs.github.com

---

**Previous Chapter:** [01 - Introduction](./01-Introduction.md)
**Next Chapter:** [03 - Git Basics](./03-Git-Basics.md)