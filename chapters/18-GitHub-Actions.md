# Chapter 18 — GitHub Actions (CI/CD)

## Introduction

In modern software development, teams want to ensure that every code change is stable, builds correctly, passes quality tests, and is safely deployed to users. Doing these tasks manually—running tests on your laptop, compiling binaries, and uploading files to a server—is slow, prone to human error, and does not scale.

To automate this, developers use **CI/CD** pipelines. GitHub includes a built-in automation engine called **GitHub Actions**. This tool allows you to write workflow scripts that automatically execute build tasks, run test suites, and deploy applications in response to Git events like `push`, `pull_request`, or `release`.

This chapter introduces the concepts of Continuous Integration (CI) and Continuous Deployment (CD), details the core components of GitHub Actions, and guides you in writing your first automated testing workflow.

---

## Learning Objectives

By the end of this chapter, you will be able to:

- Explain the concepts and value of Continuous Integration (CI) and Continuous Deployment (CD)
- Identify the core components of GitHub Actions (Workflows, Events, Jobs, Steps, Runners)
- Create a YAML configuration file to automate project building and testing
- Trigger workflows in response to specific Git events (like pushes and pull requests)
- Inspect workflow results and debug pipeline failures using the GitHub UI

---

## What is CI/CD?

CI/CD represents a set of practices designed to automate the integration and delivery of software changes:

```
        Developer Commits Code
                  │
                  ▼ (Push / Pull Request)
   Continuous Integration (CI)
   ┌────────────────────────────────────────────────────────┐
   │ Check Out  ──►  Install Deps  ──►  Lint  ──►  Run Tests │
   └────────────────────────┬───────────────────────────────┘
                            │ (If all checks pass)
                            ▼
   Continuous Deployment (CD)
   ┌────────────────────────────────────────────────────────┐
   │ Compile Build  ──►  Upload Assets  ──►  Deploy to Prod │
   └────────────────────────────────────────────────────────┘
```

* **Continuous Integration (CI):** The practice of automating the integration of code changes from multiple contributors into a single project. Whenever code is pushed, automated builds, linters, and unit tests run to ensure the code remains stable and does not break existing features.
* **Continuous Deployment (CD):** The practice of automating the release of validated code to staging or production servers. Once CI passes, the pipeline automatically compiles the software, packages container images, and deploys it to servers.

---

## 1. Components of GitHub Actions

GitHub Actions uses five core concepts to structure automation:

1. **Workflow:** An automated process defined in a YAML configuration file. Workflows must live in the `.github/workflows/` directory in your repository.
2. **Event:** A specific Git activity that triggers a workflow. Examples include `push` (pushing commits), `pull_request` (opening or updating a PR), and `release` (publishing a tagged release).
3. **Job:** A set of steps that execute on the same virtual machine runner. A workflow can contain multiple jobs, which run in parallel by default but can be configured to run sequentially.
4. **Step:** An individual task within a job. A step can run a shell command (like `npm test` or `pytest`) or call a reusable Action (like checking out your repository code).
5. **Runner:** The virtual machine or container hosted by GitHub (or self-hosted) that executes the steps in your job. GitHub provides runners for Linux (Ubuntu), macOS, and Windows.

---

## 2. Writing Your First Workflow File

To set up an automated testing pipeline for a project, you must create a workflow file. The file must use the YAML syntax and be placed in a specific directory.

### Step 1: Create the Workflows Directory
In your project root, create the hidden `.github` folder and the `workflows` subfolder:

```bash
mkdir -p .github/workflows
```

### Step 2: Create the Workflow File
Create a new file named `ci.yml` (e.g., for a Python testing environment):

```bash
touch .github/workflows/ci.yml
```

### Step 3: Write the YAML Configuration
Open `.github/workflows/ci.yml` in your editor and add the following configuration:

```yaml
name: Continuous Integration

# 1. Define when the workflow should run
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# 2. Define the jobs to run
jobs:
  build-and-test:
    runs-on: ubuntu-latest # Specify the runner operating system

    # 3. Define the steps in the job
    steps:
    # Step A: Check out the repository code onto the runner VM
    - name: Check out repository
      uses: actions/checkout@v4

    # Step B: Install Python on the runner VM
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.11'

    # Step C: Install project dependencies
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest

    # Step D: Run the test suite
    - name: Run test suite
      run: pytest
```

---

## 3. Triggering and Inspecting Workflows

Once you have written your YAML file, save it, commit it, and push it to GitHub:

```bash
git add .github/workflows/ci.yml
git commit -m "chore: configure GitHub Actions testing workflow"
git push origin main
```

### A. Inspecting Runs on GitHub
1. Go to your repository on GitHub.
2. Click the **Actions** tab.
3. You will see a list of workflow runs. Click on the run named *"chore: configure GitHub Actions..."*.
4. Click on the **build-and-test** job in the sidebar to watch the runner boot up, check out your code, install Python, install dependencies, and execute your test command.

### B. Pull Request Status Checks
Once configured, any Pull Request opened against the `main` branch will automatically run this workflow. 
* If your tests pass, a green checkmark (`✔`) appears next to the merge button.
* If your tests fail (e.g., an assertion error), a red X (`✘`) appears, warning reviewers that the branch contains broken code.

---

## GitHub Actions Best Practices

* **Keep Secrets Secure:** Never hardcode passwords, database connection strings, or API tokens in your workflow YAML file. Instead, save them under **Settings > Secrets and variables > Actions** on GitHub. Reference them in your YAML file using context variables:
  ```yaml
  env:
    STRIPE_API_KEY: ${{ secrets.STRIPE_API_KEY }}
  ```
* **Use Caching:** Downloading dependencies (like npm packages or pip downloads) on every single commit slows down builds. Use caching actions (like `actions/cache`) or language-setup cache options (e.g., `cache: 'pip'` inside `setup-python`) to speed up builds.
* **Control Cost:** GitHub provides free minutes for public repositories but caps usage on private repositories. Limit triggers so workflows only run on important branches (`main`, `develop`) and pull requests, rather than on every minor feature commit.

---

## What Comes Next

Automated pipelines are the final layer of a professional, collaborative software development environment. You now have the skills to manage code, collaborate, resolve errors, and automate quality control.

In **Chapter 19 — Git Cheat Sheet**, we package the commands covered in this entire guide into a single, structured reference page for your daily development use.
