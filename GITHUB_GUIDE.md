# How to Share Your Project on GitHub - Step by Step Guide

This guide walks you through sharing your SonarQube Docker setup on GitHub for the first time.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Part 1: GitHub Account Setup](#part-1-github-account-setup)
3. [Part 2: Local Git Configuration](#part-2-local-git-configuration)
4. [Part 3: Initialize Your Repository](#part-3-initialize-your-repository)
5. [Part 4: Create Your First Commit](#part-4-create-your-first-commit)
6. [Part 5: Create Repository on GitHub](#part-5-create-repository-on-github)
7. [Part 6: Push to GitHub](#part-6-push-to-github)
8. [Part 7: Verify on GitHub](#part-7-verify-on-github)

---

## Prerequisites

You need:
- A GitHub account (free)
- Git installed on your computer
- Your SonarQube project files ready

### Check if Git is Installed

Open terminal/command prompt and run:

```bash
git --version
```

Should show something like: `git version 2.39.0`

If not installed, download from: https://git-scm.com/

---

## Part 1: GitHub Account Setup

### Step 1.1: Create GitHub Account

1. Go to https://github.com
2. Click **"Sign up"** (top right)
3. Enter your email address
4. Click **"Continue"**
5. Create a password
6. Choose a username (e.g., `your-username` - can't contain spaces)
7. Verify your email address (GitHub will send you an email)

**Tips for username:**
- Use lowercase letters and hyphens (e.g., `john-doe`, `devops-pro`)
- Keep it professional - this is your developer identity
- You can change it later if needed

### Step 1.2: Setup SSH Key (Recommended)

GitHub can authenticate via SSH key instead of password. This is more secure.

**Generate SSH Key:**

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Press Enter for all prompts (or customize if you know SSH):

```
Enter file in which to save the key: [Press Enter]
Enter passphrase: [Press Enter - or create one]
Enter same passphrase again: [Press Enter]
```

This creates two files:
- `~/.ssh/id_ed25519` (private key - keep secret!)
- `~/.ssh/id_ed25519.pub` (public key - share on GitHub)

**Add SSH Key to GitHub:**

1. Copy your public key:

```bash
# On Mac/Linux:
cat ~/.ssh/id_ed25519.pub

# On Windows (PowerShell):
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

2. Go to GitHub → Settings (top right profile) → **"SSH and GPG keys"**
3. Click **"New SSH key"**
4. Paste your public key
5. Click **"Add SSH key"**

---

## Part 2: Local Git Configuration

Git needs to know who you are. Configure it with your GitHub details.

### Step 2.1: Configure Your Name

```bash
git config --global user.name "Your Name"
```

Example:
```bash
git config --global user.name "John Doe"
```

### Step 2.2: Configure Your Email

```bash
git config --global user.email "your-email@example.com"
```

Use the same email as your GitHub account.

### Step 2.3: Verify Configuration

```bash
git config --list
```

Look for:
```
user.name=Your Name
user.email=your-email@example.com
```

---

## Part 3: Initialize Your Repository

A repository is a project folder tracked by Git. GitHub hosts it.

### Step 3.1: Navigate to Your Project

Open terminal and go to your SonarQube folder:

```bash
cd /Users/lin/gitlab-sonarqube
```

### Step 3.2: Initialize Git

```bash
git init
```

This creates a hidden `.git` folder. You now have a local Git repository!

### Step 3.3: Check Git Status

```bash
git status
```

Expected output shows your files in red (not tracked yet):
```
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .env.example
        .gitignore
        README.md
        docker-compose.yml
        ...
```

---

## Part 4: Create Your First Commit

A commit is like a snapshot of your project at a moment in time.

### Step 4.1: Stage All Files

```bash
git add .
```

The dot (`.`) means "add all files". The `.gitignore` file automatically excludes files listed in it (like `.env`).

### Step 4.2: Verify Staged Files

```bash
git status
```

Now files should be green (staged for commit):
```
On branch master

No commits yet

Changes to be committed:
  new file:   .env.example
  new file:   .gitignore
  new file:   README.md
  new file:   docker-compose.yml
```

### Step 4.3: Create Commit

```bash
git commit -m "Initial commit: Add SonarQube Docker Compose setup"
```

The `-m` flag adds a message describing what you're committing.

**Good commit messages:**
- ✅ "Initial commit: Add SonarQube Docker setup"
- ✅ "Add configuration files and documentation"
- ❌ "update" (too vague)
- ❌ "asdf" (not descriptive)

### Step 4.4: Verify Commit

```bash
git log
```

Shows your commit history:
```
commit 3a4b5c6d7e8f9g0h1i2j3k4l5m6n7o8p [HEAD -> master]
Author: Your Name <your-email@example.com>
Date:   Mon Nov 10 10:30:45 2024 +0000

    Initial commit: Add SonarQube Docker Compose setup
```

---

## Part 5: Create Repository on GitHub

### Step 5.1: Go to GitHub

1. Log into https://github.com
2. Click the **"+"** icon (top right)
3. Select **"New repository"**

### Step 5.2: Fill in Repository Details

| Field | Value | Notes |
|-------|-------|-------|
| **Repository name** | `sonarqube-docker` | Use lowercase, hyphens, no spaces |
| **Description** | Optional | "Production-ready SonarQube Docker setup" |
| **Public/Private** | Public | So others can see it (recommended for sharing) |
| **Initialize repository** | ❌ No | Don't check any boxes (you have local files) |

### Step 5.3: Create Repository

Click **"Create repository"** button.

### Step 5.4: Copy the Commands

GitHub shows commands to link your local repo to GitHub. Look for this section:

```
…or push an existing repository from the command line

git remote add origin git@github.com:your-username/sonarqube-docker.git
git branch -m master main
git push -u origin main
```

**Copy these commands** - you'll use them next!

---

## Part 6: Push to GitHub

Pushing sends your local commits to GitHub servers.

### Step 6.1: Add Remote Repository

This tells Git where your GitHub repository is.

```bash
git remote add origin git@github.com:your-username/sonarqube-docker.git
```

Replace `your-username` with your actual GitHub username.

**If you don't have SSH set up**, use HTTPS instead:
```bash
git remote add origin https://github.com/your-username/sonarqube-docker.git
```

### Step 6.2: Verify Remote Added

```bash
git remote -v
```

Should show:
```
origin  git@github.com:your-username/sonarqube-docker.git (fetch)
origin  git@github.com:your-username/sonarqube-docker.git (push)
```

### Step 6.3: Rename Branch (Optional but Recommended)

Modern GitHub uses `main` instead of `master`. You can skip this if you prefer `master`.

```bash
git branch -m master main
```

### Step 6.4: Push to GitHub

```bash
git push -u origin main
```

Or if you kept `master`:
```bash
git push -u origin master
```

The `-u` flag sets this as your default branch.

**First time only:** You might be asked to authenticate. Follow the prompts.

---

## Part 7: Verify on GitHub

### Step 7.1: Check GitHub Website

1. Go to https://github.com/your-username/sonarqube-docker
2. You should see your files!

### Step 7.2: Check Your Files Are There

Look for:
- ✅ `README.md` - Shown at the bottom
- ✅ `docker-compose.yml`
- ✅ `.env.example`
- ✅ `.gitignore`

### Step 7.3: Verify .env NOT Uploaded

⚠️ **Important**: Make sure `.env` is **NOT** in the file list. This means `.gitignore` worked!

If `.env` is there, follow the [Fix It](#fix-accidentally-committed-env) section below.

---

## Common Tasks After Uploading

### Make Changes Locally

After your initial upload, you can make changes:

```bash
# Make edits to files
nano README.md

# Check what changed
git status

# Stage changes
git add .

# Commit changes
git commit -m "Update README with more examples"

# Push to GitHub
git push
```

### Clone Your Repository

Clone means download a copy from GitHub:

```bash
git clone https://github.com/your-username/sonarqube-docker.git
cd sonarqube-docker
```

### Add Collaborators

1. Go to your repo → **"Settings"** tab
2. Click **"Collaborators"** (left sidebar)
3. Click **"Add people"**
4. Enter their GitHub username
5. Click **"Add"**

---

## Troubleshooting

### "Repository already exists"

You may have already initialized. Delete the `.git` folder and try again:

```bash
rm -rf .git
git init
```

### "Permission denied (publickey)"

SSH authentication failed. Either:

**Option 1: Use HTTPS instead**

```bash
git remote remove origin
git remote add origin https://github.com/your-username/sonarqube-docker.git
git push -u origin main
```

**Option 2: Set up SSH properly**

Follow Part 1.2 again.

### "fatal: remote origin already exists"

Your remote is already set. Remove it first:

```bash
git remote remove origin
git remote add origin git@github.com:your-username/sonarqube-docker.git
```

### Files not uploading

Check what Git sees:

```bash
git status
```

Stage everything:

```bash
git add .
git commit -m "Update"
git push
```

### .env file was accidentally uploaded

This is a security issue! Fix it immediately:

```bash
# Remove the file from Git history
git rm --cached .env

# Update .gitignore to include it
echo ".env" >> .gitignore

# Commit the fix
git add .gitignore
git commit -m "Remove .env from tracking"
git push
```

Then consider resetting your credentials (if passwords were exposed).

---

## Verify Everything Works

### Test Your Repository

1. Go to your GitHub repo page
2. Click the green **"<> Code"** button
3. Copy the clone URL
4. In a different folder, run:

```bash
git clone <paste-url-here>
cd sonarqube-docker
```

5. Verify the project loads correctly

---

## Next Steps

After uploading to GitHub:

1. **Add a license** - Click "Add license" on the repo page
2. **Enable GitHub Pages** - Share documentation automatically
3. **Create releases** - Mark stable versions
4. **Add GitHub Actions** - Automate testing/deployments
5. **Promote your project**:
   - Share on Twitter/LinkedIn with the GitHub link
   - Add to dev.to portfolio
   - Include in your resume

---

## Quick Reference

### Essential Commands

```bash
# Initial setup
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

# Create local repo
git init

# Create first commit
git add .
git commit -m "Initial commit"

# Connect to GitHub
git remote add origin https://github.com/username/repo-name.git

# Push to GitHub
git push -u origin main
```

### After First Upload

```bash
# Check status
git status

# Add changes
git add .

# Commit
git commit -m "Your message"

# Push
git push
```

---

## Need Help?

- GitHub Docs: https://docs.github.com
- Git Basics: https://git-scm.com/book/en/v2/Getting-Started-Git-Basics
- GitHub Community: https://github.com/orgs/community/discussions

---

**You're ready! 🚀 Follow these steps and you'll have your project on GitHub in minutes!**
