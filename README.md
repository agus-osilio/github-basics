# Git & GitHub Basics

> [!WARNING]
> Always review commands before running them. Never execute a command you don't understand — especially those involving `--force`, `sudo`, or history rewrites, as they can cause irreversible changes.

> [!IMPORTANT]
> This guide covers how to use Git and GitHub entirely from the terminal — no GUI, no browser beyond the one-time authentication step. It is written for both **Windows** (PowerShell) and **Ubuntu** (bash), with platform-specific instructions clearly separated where they differ.

> [!NOTE]
> All `git` and `gh` commands work the same in PowerShell and bash — use whichever terminal you prefer. The platform-specific parts are the installation steps (1.1–1.3) plus a few credential-helper details (the Windows-only command in 1.4, the Ubuntu-only `gh auth setup-git` in 1.5, and all of 1.6, which is Windows-only); these are clearly marked where they appear. Commands marked with `--global` are set **once per machine** and never need to be repeated.


---

## Index

1. [First-time Setup](#1-first-time-setup)
   - [Install Git](#11-install-git)
   - [Install GitHub CLI on Windows](#12-install-github-cli-on-windows)
   - [Install GitHub CLI on Ubuntu](#13-install-github-cli-on-ubuntu)
   - [Global Git Configuration](#14-global-git-configuration)
   - [Authentication](#15-authentication)
   - [Migrating to Git Credential Manager (GCM)](#16-migrating-to-git-credential-manager-gcm)
2. [Working with Repositories](#2-working-with-repositories)
   - [Clone an existing repo](#21-clone-an-existing-repo)
   - [Pull an existing remote repo into a local folder](#22-pull-an-existing-remote-repo-into-a-local-folder)
   - [Push a local project to a new repo](#23-push-a-local-project-to-a-new-repo)
   - [Create a repo from the terminal](#24-create-a-repo-from-the-terminal)
3. [Daily Workflow](#3-daily-workflow)
4. [Ignoring Files](#4-ignoring-files)
5. [History Management](#5-history-management)
   - [Reset branch history](#51-reset-branch-history)
   - [Delete all previous commits](#52-delete-all-previous-commits)
6. [Quick Command Reference](#6-quick-command-reference)

---

## 1. First-time Setup

### 1.1 Install Git

**What is Git?**  
Git is a version control system — a tool that tracks every change you make to your files over time. It lets you save snapshots of your project (commits), go back to previous states, and collaborate with others without overwriting each other's work. Every `git` command in this guide runs on top of it, so it needs to be installed first.

**What is GitHub, and how does it relate to Git?**  
Git runs entirely on your local machine. GitHub is a platform that hosts your Git repositories in the cloud, making them accessible from anywhere and giving you a place to back up and share your work. Git is the tool; GitHub is the service built around it.

**Why do you also need GitHub CLI?**  
GitHub CLI (`gh`) is covered in the next steps, but it's worth understanding upfront: while Git handles version control locally, `gh` is what lets you interact with GitHub from the terminal — creating repos, opening pull requests, checking authentication status, and more. Without it, you'd need to use the GitHub website for most account-level actions. Note that on Windows, avoiding password prompts on `git push`/`pull` is handled by Git Credential Manager (GCM), not by `gh` — they serve different roles and are both needed.

**Windows**

```powershell
winget install --id Git.Git -e --source winget
```

**Ubuntu**

Git usually comes pre-installed on Ubuntu. Check first:

```bash
git --version
```

If the command is not found, install it:

```bash
sudo apt update
sudo apt install git -y
```

---

### 1.2 Install GitHub CLI on Windows

**What is GitHub CLI?**  
GitHub CLI (`gh`) is an official command-line tool that lets you interact with GitHub directly from your terminal — create repos, authenticate, manage pull requests, and more — without opening a browser for every action. On Windows, credential storage for `git push`/`pull` is handled by Git Credential Manager (GCM), which is bundled with Git for Windows — `gh` and GCM serve different roles and work independently.

```powershell
winget install --id GitHub.cli -e --source winget
```

**Check your version and update:**

```powershell
gh --version
winget upgrade --id GitHub.cli -e --source winget
```

---

### 1.3 Install GitHub CLI on Ubuntu

> Do **not** use `sudo snap install gh` — the snap version is outdated and has compatibility issues.

Install from the official GitHub repository (first time only):

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
    && sudo mkdir -p -m 755 /etc/apt/keyrings \
    && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
    && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
    && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && sudo mkdir -p -m 755 /etc/apt/sources.list.d \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && sudo apt update \
    && sudo apt install gh -y
```

What each line does:

1. **`(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y))`** — Checks whether `wget` is already installed. If it is, nothing happens. If it isn't, updates the package list and installs it automatically. This prevents the rest of the command from failing due to a missing dependency.
2. **`sudo mkdir -p -m 755 /etc/apt/keyrings`** — Creates the folder where trusted GPG keys are stored. GPG keys are used to verify that the packages you download are genuine and haven't been tampered with.
3. **`out=$(mktemp) && wget -nv -O$out ...githubcli-archive-keyring.gpg`** — Creates a secure temporary file and downloads GitHub's official GPG key directly into it. Using `mktemp` is safer than a fixed path like `/tmp/file` because it generates a unique filename that can't be predicted or hijacked.
4. **`cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null`** — Copies the downloaded key into the trusted keys folder. `tee` is used because writing to `/etc/apt/keyrings/` requires root privileges — piping through it is the standard way to redirect output with `sudo`. The `> /dev/null` suppresses terminal output.
5. **`sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg`** — Gives read permission on the key file to other system processes that will need it during installation.
6. **`sudo mkdir -p -m 755 /etc/apt/sources.list.d`** — Creates the directory for package source lists if it doesn't already exist.
7. **`echo "deb ..." | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null`** — Registers GitHub's package repository as a source for `apt`, the Ubuntu package manager. This is what allows `apt install gh` to find the package. Output is suppressed.
8. **`sudo apt update`** — Refreshes the list of available packages, now including GitHub's repository.
9. **`sudo apt install gh -y`** — Installs `gh`. The `-y` flag auto-confirms the installation prompt.

**Why does Ubuntu require all these steps?**

Ubuntu's package manager (`apt`) only installs software from sources it explicitly trusts. By default, it only knows about Ubuntu's own official repositories — and GitHub CLI is not included there. So before you can install `gh`, you need to do two things: tell `apt` where to find it (step 7), and prove that source is trustworthy (steps 2–5).

The GPG key is the trust mechanism. When you download a package, `apt` uses that key to verify the package actually came from GitHub and wasn't modified in transit. Without it, `apt` would refuse to install anything from that source.

On Windows, `winget` works differently — it has its own curated catalog of verified apps, so GitHub CLI is already listed there and you can install it in one command. Ubuntu's model is more manual but gives you full control over which external sources your system trusts.

**Check your version and update:**

Once the repository is added (steps above), keeping `gh` current only takes two commands:

```bash
gh --version
sudo apt update && sudo apt install gh
```

---

### 1.4 Global Git Configuration

With Git installed, run these once per machine to set your identity. They apply to all repositories. From here on, all commands work the same in PowerShell and bash — run them from whichever terminal you prefer.

```bash
git config --global user.name "MY-USERNAME"
git config --global user.email "84744827+MY-USERNAME@users.noreply.github.com"
git config --global init.defaultBranch main
```

The last line sets `main` as the default branch name for every new repository you create with `git init`. Without it, Git defaults to `master` — which won't match the branch name GitHub expects when you push.

**On Windows only**, also configure the credential manager:

```bash
git config --global credential.helper manager
```

Tells Git to use **Git Credential Manager (GCM)** — the cross-platform credential helper maintained at [git-ecosystem/git-credential-manager](https://github.com/git-ecosystem/git-credential-manager), bundled with Git for Windows since version 2.28. GCM stores your credentials securely in the Windows Credential Manager and opens a browser window to complete OAuth authentication when no token is cached yet. Without it, Git may prompt for a username and password on every push or pull. GCM operates independently from `gh` — it handles Git credentials on its own and does not require `gh auth login` to function.

> [!IMPORTANT]
> The older **Git Credential Manager for Windows** (`microsoft/Git-Credential-Manager-for-Windows`) was archived in July 2023 and can no longer generate new GitHub tokens. If you have it installed, see [section 1.6](#16-migrating-to-git-credential-manager-gcm) to migrate.

**`--global` vs no flag**

The `--global` flag saves the setting to your user profile (`~/.gitconfig`), so it applies to every repository on your machine. Without it, the setting is saved only inside the current repo's `.git/config` and does not affect any other project.

Use `--global` for identity settings (name, email). Drop the flag if you need to override them for one specific project — for example, using a work email in a single repo while keeping your personal email everywhere else.

> Use your GitHub no-reply email to avoid exposing your personal address.  
> Find it at: GitHub → Settings → Emails → "Keep my email address private".

---

### 1.5 Authentication

Authentication is how you prove to GitHub that you are who you say you are. Without it, you can read public repos but cannot push changes, create repos, or access anything private. GitHub CLI handles this by storing a token on your machine that is sent automatically with every request — you authenticate once and never need to enter your password again.

Run on both Windows and Ubuntu:

```bash
gh auth login
```

Select the following options in the wizard:
- `GitHub.com`
- `HTTPS`
- `Login with a web browser`

An 8-character code will appear. Press Enter, the browser opens — paste the code and authorize the app.

**On Ubuntu only**, also run this to configure git to use `gh` as the credential manager:

```bash
gh auth setup-git
```

This replaces any previous credential helper and allows `git push`/`git pull` without prompting for a password.

**Verify authentication worked:**

```bash
gh auth status
# Expected: Logged in to github.com account YOUR-USERNAME
```

---

### 1.6 Migrating to Git Credential Manager (GCM)

> [!NOTE]
> This section applies to **Windows only**. On Ubuntu, `gh auth setup-git` (section 1.5) is sufficient — the old GCM for Windows was not Linux compatible.

> [!IMPORTANT]
> If you installed **Git for Windows 2.29 or later** — which covers most users on Windows 10 or 11 who installed Git in the last few years — the new GCM is already bundled and set as the default credential helper. Before going through the steps below, run these two commands to check:
>
> ```powershell
> git config --global credential.helper   # should return: manager
> git credential-manager --version        # should return: 2.x.x
> ```
>
> If both match, you already have the new GCM active and can skip this section entirely.

**Why this matters**

The original **Git Credential Manager for Windows** (`microsoft/Git-Credential-Manager-for-Windows`) was archived in July 2023 and no longer receives updates. GitHub has since disabled the password-based authentication APIs it relied on, so any machine still running the old GCM will fail to authenticate on new sessions.

The replacement is **Git Credential Manager** (`git-ecosystem/git-credential-manager`). It is cross-platform, uses OAuth-based authentication, and has been **bundled with Git for Windows since version 2.28** — meaning if you installed or updated Git for Windows on Windows 10 or 11 after mid-2020, you almost certainly already have it. Windows itself does not ship GCM; it comes through the Git for Windows installer.

**Is the bundled GCM safe and actively maintained?**  
Yes. Microsoft maintains the core platform under the `git-ecosystem` organization, and GitHub, Azure DevOps, and other providers each maintain their own authentication plugins within it. It uses the Windows Credential Manager (the native OS keychain) to store tokens securely.

**When do you need to do anything?**

| Situation | Action needed |
|---|---|
| Installed Git for Windows recently and never installed the old GCM separately | None — you already have the new GCM |
| Had the old GCM installed as a standalone program | Follow Steps 1 and 2 below |
| Unsure | Run the check command in Step 1 |

---

**Step 1 — Check whether the old GCM is present**

Run both commands to understand your current state:

```powershell
# Check if the old GCM is installed as a standalone program
Get-Package -Name "Git Credential Manager for Windows" -ErrorAction SilentlyContinue

# Check what credential manager is active and its version
git credential-manager --version
```

- If `Get-Package` returns **nothing** and `git credential-manager --version` shows `2.x.x` → you have the new GCM, skip to Step 2.
- If `Get-Package` returns a result → the old GCM is installed, continue below.

To uninstall the old GCM from the command line:

```powershell
git credential-manager uninstall
```

If that fails, force removal with:

```powershell
git credential-manager remove --passive --force
```

---

**Step 2 — Update Git for Windows**

The new GCM is bundled with Git for Windows 2.28 and later. Updating Git is enough to get it:

```powershell
winget upgrade --id Git.Git -e --source winget
```

When the installer asks which credential helper to use, leave the default — **Git Credential Manager** is selected by default.

---

**Step 3 — Set the credential helper**

Confirm Git is pointing to the new GCM:

```bash
git config --global credential.helper manager
```

Run `git config --global credential.helper` to verify the output is `manager`. If it shows a path to the old binary (e.g., something containing `Git-Credential-Manager-for-Windows`), the command above overwrites it.

---

**Step 4 — Clear cached GitHub credentials**

Tokens issued by the old GCM need to be removed before the new one can authenticate cleanly.

Open **Credential Manager** via the Windows Start menu → go to the **Windows Credentials** tab → find and delete any entries that reference `github.com` (they typically appear as `git:https://github.com`).

---

**Step 5 — Re-authenticate with GitHub via HTTPS**

Run the GitHub CLI login wizard:

```bash
gh auth login
```

Select the following options:
- `GitHub.com`
- `HTTPS`
- `Login with a web browser`

An 8-character code will appear in the terminal. Press Enter — your default browser opens. Paste the code and authorize the app. The new GCM securely stores the resulting token and reuses it automatically for every subsequent `git push` and `git pull`.

If you don't use `gh`, you can skip this step and just run `git push` on any repo. The new GCM will detect the missing token and open a browser window on the first unauthenticated operation.

---

**Verify everything works:**

```bash
gh auth status
# Expected: Logged in to github.com account YOUR-USERNAME (keyring)
```

---

## 2. Working with Repositories

**What is a repository?**  
A repository (repo) is the folder where Git tracks your project. It contains all your files plus the complete history of every change ever made to them. Repos can live locally on your machine, remotely on GitHub, or both — with Git keeping them in sync.

**Push, Pull, and Merge**

| Operation | Direction | What it does |
|---|---|---|
| **Push** | Local → Remote | Uploads your local commits to GitHub |
| **Pull** | Remote → Local | Downloads remote changes and applies them to your local copy |
| **Merge** | Branch → Branch | Combines the history of two branches into one |

Pull is actually a shortcut for two operations: it fetches the remote changes and then automatically merges them into your current branch.

---

### 2.1 Clone an existing repo

Cloning is the most common way to get a remote repo onto your machine. It downloads the entire project — files and full history — into a new folder, and automatically configures the remote so you can push and pull right away.

```bash
git clone https://github.com/USER/REPO-NAME.git
```

This creates a folder named `REPO-NAME` in your current directory with everything ready to use.

---

### 2.2 Pull an existing remote repo into a local folder

Use this when you already have a local folder with files and want to link it to an existing remote repo, rather than starting fresh from a clone.

> [!IMPORTANT]
> `git init` creates your initial branch using the name from `init.defaultBranch` (set in [1.4](#14-global-git-configuration)). If you skipped that step, run `git config --global init.defaultBranch main` **before** `git init` — otherwise your branch will be called `master` and the push to `main` in the last step will fail.

```bash
git init
git remote add origin https://github.com/USER/REPO-NAME.git
git pull origin main --allow-unrelated-histories

git add .
git commit -m "first commit"
git push -u origin main
```

What each command does:

| Command | What it does |
|---|---|
| `git init` | Starts Git tracking in the current folder, creating a hidden `.git` directory |
| `git remote add origin <url>` | Links your local repo to the remote GitHub repo. `origin` is the conventional name for the main remote — a saved shortcut to the URL |
| `git pull origin main --allow-unrelated-histories` | Downloads commits from the remote and merges them into your local history. The flag is explained below |
| `git add .` | Stages all files in the current folder for the next commit |
| `git commit -m "first commit"` | Saves a snapshot of the merged state locally |
| `git push -u origin main` | Uploads your commits to GitHub. `-u` sets `origin main` as the default upstream so future pushes only need `git push` |

**What `--allow-unrelated-histories` does**

Git refuses to merge two branches that share no common commit — it treats them as completely separate projects. This happens here because your local folder was initialized with `git init` (brand new history) while the remote repo already has its own commits (e.g. a README or `.gitignore` that GitHub created automatically). The flag tells Git to merge them anyway, ignoring the lack of a shared ancestor. You only need it once — after this pull, both histories are joined and everything works normally.

**When you don't need it**

Skip the flag if the remote repo is completely empty — no README, no `.gitignore`, no initial commit of any kind. In that case both sides have nothing to reconcile and a plain `git pull origin main` will work, or you can go straight to `git push -u origin main` without pulling at all.

---

### 2.3 Push a local project to a new repo

**When to use this:** You have local code and you already created the GitHub repository manually — for example through the GitHub website. You now want to upload your local project to it.

> [!IMPORTANT]
> The GitHub repo must be **completely empty** — no README, no `.gitignore`, no license. If it has any content, the push will be rejected because the two histories don't share a common commit. In that case, use [2.2](#22-pull-an-existing-remote-repo-into-a-local-folder) instead.

> [!IMPORTANT]
> `git init` creates your initial branch using the name from `init.defaultBranch` (set in [1.4](#14-global-git-configuration)). If you skipped that step, run `git config --global init.defaultBranch main` **before** `git init` — otherwise your branch will be called `master` and the push to `main` in the last step will fail.

```bash
git init
git remote add origin https://github.com/MY-USERNAME/REPO-NAME.git

git add .
git commit -m "first commit"
git push -u origin main
```

What each command does:

| Command | What it does |
|---|---|
| `git init` | Starts Git tracking in the current folder, creating a hidden `.git` directory |
| `git remote add origin <url>` | Tells your local repo where the GitHub repo lives. `origin` is the conventional name for the main remote — you can think of it as a saved shortcut to the URL |
| `git add .` | Stages all files in the current folder for the next commit |
| `git commit -m "first commit"` | Saves the first snapshot of your project locally |
| `git push -u origin main` | Uploads your commits to GitHub. The `-u` flag sets `origin main` as the default upstream so future pushes only need `git push` |

---

### 2.4 Create a repo from the terminal

**When to use this:** You have local code and the GitHub repository doesn't exist yet. The difference with 2.3 is that here you create the GitHub repo yourself from the terminal using `gh repo create` — no browser needed.

**Step 1 — Create the empty repo on GitHub:**

```bash
gh repo create REPO-NAME --private
```

This creates an empty repo on GitHub and prints its URL. You'll use that URL in the next step.

**Step 2 — Initialize your local folder, link it to the new repo, and push:**

> [!IMPORTANT]
> `git init` creates your initial branch using the name from `init.defaultBranch` (set in [1.4](#14-global-git-configuration)). Run `git config --global init.defaultBranch main` **before** `git init` if you haven't already.

```bash
git init
git remote add origin https://github.com/MY-USERNAME/REPO-NAME.git
git add .
git commit -m "first commit"
git push -u origin main
```

What each command does:

| Command | What it does |
|---|---|
| `git init` | Starts Git tracking in the current folder |
| `git remote add origin <url>` | Links your local repo to the GitHub repo created in step 1. Use the URL printed by `gh repo create` |
| `git add .` | Stages all files for the first commit |
| `git commit -m "first commit"` | Saves the first local snapshot |
| `git push -u origin main` | Uploads your commits to GitHub. `-u` sets the default upstream so future pushes only need `git push` |

**Shortcut — steps 1 and 2 in a single command:**

If you prefer to do everything at once, run `git init`, `git add .`, and `git commit` first, then let `gh repo create` handle the GitHub repo creation, remote setup, and push in one go:

```bash
git init
git add .
git commit -m "first commit"
gh repo create REPO-NAME --private --source=. --remote=origin --push
```

| Flag | What it does |
|---|---|
| `--source=.` | Uses the current folder as the source — replaces `git remote add origin` |
| `--remote=origin` | Names the remote connection `origin` |
| `--push` | Immediately pushes your commits — replaces `git push -u origin main` |

**2.3 vs 2.4 at a glance:**

| | 2.3 | 2.4 |
|---|---|---|
| GitHub repo | Already exists (created in browser) | Doesn't exist yet |
| How it's created | Manually, on GitHub.com | `gh repo create` from the terminal |
| Connection to local | `git remote add origin <url>` | `git remote add origin <url>` (step 2) |
| Push | `git push -u origin main` | `git push -u origin main` (step 2) |

---

## 3. Daily Workflow

Once your repo is set up and linked to GitHub (see section 2), this is the cycle you repeat every time you make changes.

**What is a commit?**  
A commit is a saved snapshot of your project at a specific point in time. Every commit records exactly which lines changed, who made the change, when, and why (via the commit message). Commits are permanent and form a chain — the full history of your project.

**The commit message**  
The string you pass with `-m` is the commit message. It should describe *what changed and why*, not *how*. Good messages make it possible to understand the history of a project months later without reading every diff.

Examples of good commit messages:
- `"fix login redirect loop on session expiry"`
- `"add dark mode toggle to settings page"`
- `"remove unused analytics dependency"`

**Checking what you're about to commit**  
Before staging and committing, use `git status` to see which files have been modified, which are already staged, and which are untracked. It's a good habit to run it before every commit.

```bash
git status

git add .
git commit -m "your message"
git push

# Pull latest changes from remote
git pull origin main
```

---

## 4. Ignoring Files

**What is `.gitignore`?**  
`.gitignore` is a plain text file in the root of your repo that tells Git which files and folders to never track. Anything listed there will never be staged, committed, or pushed — no matter how many times you run `git add .`. It is typically used for secrets, API keys, build output, editor config, and OS-generated files.

To ignore something, create a `.gitignore` file in the root of your repo (if it doesn't exist yet) and add one entry per line:

```
.claude/
build/
secrets.env
*.log
```

That's it — no special syntax required to create or edit the file, just open it in any text editor and write the paths you want to ignore.

**Pattern syntax**

| Pattern | What it matches |
|---|---|
| `secrets.env` | A file named exactly `secrets.env` at any directory level in the repo |
| `*.log` | All files ending in `.log` anywhere in the repo |
| `build/` | The entire `build` folder and everything inside it |
| `docs/*.pdf` | All `.pdf` files directly inside the `docs/` folder |
| `!important.log` | Exception — do **not** ignore this file even if a rule above would match it |

---

## 5. History Management

### 5.1 Reset branch history

Replaces the entire commit history with a single initial commit, while keeping all current files.

```bash
git checkout --orphan clean_branch
git add -A
git commit -m "first commit"
git branch -D main
git branch -m main
git push --force origin main
```

What each command does:

| Command | What it does |
|---|---|
| `git checkout --orphan clean_branch` | Creates a new branch with no parent commit — completely disconnected from the existing history. Files are preserved but no commit has been made yet |
| `git add -A` | Stages all changes, including deletions. Broader than `git add .`: also picks up files that were removed from the working tree |
| `git commit -m "first commit"` | Creates a single new commit containing all current files — this becomes the only commit in the new history |
| `git branch -D main` | Force-deletes the `main` branch. `-D` skips the safety check that `-d` applies (which blocks deletion of branches with unmerged changes) |
| `git branch -m main` | Renames the current branch (`clean_branch`) to `main` |
| `git push --force origin main` | Overwrites the remote history with the new single-commit history. Without `--force`, GitHub would reject the push since the histories no longer match |

---

### 5.2 Delete all previous commits

Wipes the commit history and leaves only the current state as the initial commit.

```bash
git update-ref -d HEAD
git add .
git commit -m "Initial commit"
git push origin <branch-name> --force
```

What each command does:

| Command | What it does |
|---|---|
| `git update-ref -d HEAD` | Removes the `HEAD` reference — Git's internal pointer to the current commit. This erases the entire commit history from Git's perspective while leaving all your files untouched on disk |
| `git add .` | Stages all current files to be included in the new initial commit |
| `git commit -m "Initial commit"` | Creates a brand-new first commit with all current files — no history attached |
| `git push origin <branch-name> --force` | Overwrites the remote history with this single-commit history. `--force` is required because the remote still has the old commits and would otherwise reject the push |

> Both approaches rewrite history permanently. Anyone with a local clone will need to re-clone the repo.

---

## 6. Quick Command Reference

| Action | Command |
|---|---|
| Clone a remote repo locally | `git clone <url>` |
| Initialize local repo | `git init` |
| Set default branch name | `git config --global init.defaultBranch main` |
| Set global username | `git config --global user.name "..."` |
| Set global email | `git config --global user.email "..."` |
| Link remote origin | `git remote add origin <url>` |
| Check file status | `git status` |
| Stage all files | `git add .` |
| Commit changes | `git commit -m "message"` |
| Push changes | `git push -u origin main` |
| Pull remote changes | `git pull origin main` |
| Ignore a file or folder | Add the path to `.gitignore`, one entry per line |
| Create private repo (CLI) | `gh repo create NAME --private` |
| Force push | `git push --force origin main` |
| Check authentication status | `gh auth status` |

---

*Personal reference guide.*
