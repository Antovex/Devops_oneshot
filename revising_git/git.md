# Git Essential Commands Cheatsheet ✅ **REVISED**

A comprehensive guide to the most fundamental Git commands for version control.

**🏆 Status:** COMPLETE - Revision completed on September 17, 2025

---

## 📋 Table of Contents
- [Initial Setup](#initial-setup)
- [Repository Management](#repository-management)
- [Working with Files](#working-with-files)
- [Commit History](#commit-history)
- [Navigation](#navigation)
- [Branch Management](#branch-management)
- [Remote Repository Management](#remote-repository-management)
- [Collaboration Commands](#collaboration-commands)
- [Commit Message Best Practices](#commit-message-best-practices)
- [Complete Git Workflow](#complete-git-workflow)
- [Quick Command Reference](#quick-command-reference)

---

## 🔧 Initial Setup

### Check Git Version
```bash
git --version
```
**Description:** Displays the currently installed version of Git. Useful for troubleshooting compatibility issues.

### Configure Git (First Time Setup)
```bash
git config --global user.name '<Your name>'
git config --global user.email '<Your email>'
git config --global init.defaultBranch main
```
**Description:** 
- Sets your name and email for Git commits globally
- Configures the default branch name to be "main" (modern standard)
- These settings apply to all repositories on your system

---

## 📁 Repository Management

### Initialize a New Repository
```bash
git init
```
**Description:** Creates a new Git repository in the current directory. This adds a hidden `.git` folder that tracks all version control data.

### Check Repository Status
```bash
git status
```
**Description:** Shows the current state of your repository including:
- Modified files
- Staged files ready for commit
- Untracked files
- Current branch information

---

## 📝 Working with Files

### Stage Files for Commit
```bash
# Stage specific files
git add <filename1> <filename2>

# Stage all files in current directory
git add .

# Stage all files in repository
git add -A
```
**Description:** Moves files from the working directory to the staging area, preparing them for the next commit.

### Commit Changes
```bash
git commit -m "<Descriptive commit message>"
```
**Description:** Creates a snapshot of staged changes with a descriptive message. Good commit messages are:
- Clear and concise
- Written in present tense
- Explain what the commit does

---

## 📚 Commit History

### View Commit History
```bash
# Basic log
git log

# One line per commit
git log --oneline

# Graphical representation
git log --graph --oneline
```
**Description:** Displays the commit history showing:
- Commit hash (unique identifier)
- Author and date
- Commit message

---

## 🧭 Navigation

### Navigate to Previous Commits
```bash
# Go to specific commit
git checkout <commit-hash>

# Return to latest commit on current branch
git checkout main
```
**Description:** 
- Allows you to view your project at any previous point in time
- Creates a "detached HEAD" state when checking out old commits
- Use branch name to return to the latest version

---

## 🌿 Branch Management

### Create a New Branch
```bash
# Create branch (but stay on current branch)
git branch <branch_name>

# Create and switch to new branch
git checkout -b <new_branch_name>

# Create branch from specific source branch
git branch <new_branch_name> <source_branch_name>
```
**Description:** Branches allow parallel development without affecting the main codebase. Essential for feature development and collaboration.

### Switch Between Branches
```bash
# Switch to existing branch
git checkout <branch_name>

# Switch to main branch
git checkout main
```
**Description:** Allows you to move between different branches to work on different features or versions of your code.

### Branch Operations Workflow
```bash
1. git checkout -b feature-login    # Create and switch to feature branch
2. # Make changes to files
3. git add .                        # Stage changes
4. git commit -m "Add login feature" # Commit changes
5. git push -u origin feature-login # Push new branch to remote
```

---

## 🌐 Remote Repository Management

### Rename Current Branch to Main
```bash
git branch -M main
```
**Description:** Renames the current branch to "main". The `-M` flag forces the rename even if a branch named "main" already exists. This is commonly used to standardize branch naming.

### Add Remote Repository
```bash
git remote add origin <repository-url>
```
**Description:** Links your local repository to a remote repository (like GitHub, GitLab, etc.). The name "origin" is the conventional name for the primary remote repository.

**Example:**
```bash
git remote add origin https://github.com/username/repository-name.git
```

### Push to Remote Repository
```bash
# First push (sets upstream for new branch)
git push --set-upstream origin <branch_name>
git push -u origin <branch_name>

# Subsequent pushes (after upstream is set)
git push
```
**Description:** 
- Uploads your local commits to the remote repository
- `--set-upstream` or `-u` flag establishes tracking between local and remote branch
- After setting upstream, simple `git push` works for future pushes

### Additional Remote Commands
```bash
# View remote repositories
git remote -v

# Remove a remote
git remote remove origin

# Change remote URL
git remote set-url origin <new-url>
```

---

## 🤝 Collaboration Commands

### Pull Changes from Remote
```bash
git pull
```
**Description:** Downloads and merges changes from the remote repository into your current branch. Always pull before pushing to avoid conflicts.

### Common Collaboration Workflow
```bash
1. git pull                         # Get latest changes
2. git checkout -b feature-branch   # Create feature branch
3. # Make changes
4. git add .                        # Stage changes
5. git commit -m "descriptive msg"  # Commit with good message
6. git push -u origin feature-branch # Push feature branch
7. # Create Pull Request on GitHub/GitLab
```

---

## ✍️ Commit Message Best Practices

### The Golden Rule
**"If applied to the codebase, this commit will ___"**

Your commit message should complete this sentence.

### Good Examples
```bash
git commit -m "Add user authentication system"
git commit -m "Fix navigation menu styling issue"
git commit -m "Update API documentation"
git commit -m "Remove deprecated payment methods"
```

### Commit Message Structure
```
<type>: <description>

[optional body]

[optional footer]
```

### Common Types
- **feat:** New feature
- **fix:** Bug fix
- **docs:** Documentation changes
- **style:** Code formatting changes
- **refactor:** Code refactoring
- **test:** Adding or updating tests
- **chore:** Maintenance tasks

### Examples with Types
```bash
git commit -m "feat: add user profile page"
git commit -m "fix: resolve login button alignment"
git commit -m "docs: update installation instructions"
```

---

## 🔄 Complete Git Workflow

### Local Development Workflow
```bash
1. git init                     # Initialize repository
2. git add .                    # Stage files
3. git commit -m "message"      # Commit changes
4. git log --oneline           # Review history
```

### Feature Branch Workflow
```bash
1. git pull                           # Get latest changes
2. git checkout -b feature-name       # Create feature branch
3. git add .                          # Stage changes
4. git commit -m "descriptive message" # Commit with good message
5. git push -u origin feature-name    # Push feature branch
6. # Create Pull Request
7. git checkout main                  # Switch back to main
8. git pull                           # Update main branch
```

### Remote Repository Setup
```bash
1. git branch -M main          # Ensure main branch
2. git remote add origin <url> # Link to remote
3. git push -u origin main     # Push and set upstream
```

---

## 💡 Best Practices

### General Practices
1. **Commit Often:** Make small, logical commits
2. **Write Good Messages:** Follow the "If applied..." rule
3. **Check Status:** Always run `git status` before committing
4. **Stage Selectively:** Be intentional about what you commit

### Branching Practices
5. **Use Descriptive Names:** `feature-user-login`, `fix-payment-bug`
6. **Keep Branches Small:** Focus on single features or fixes
7. **Delete Merged Branches:** Clean up after merging

### Collaboration Practices
8. **Pull Before Push:** Always `git pull` before pushing
9. **Review Changes:** Use `git diff` to review before committing
10. **Communicate:** Use clear commit messages and PR descriptions

---

## 🚀 Quick Command Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `git init` | Initialize repo | `git init` |
| `git add` | Stage files | `git add .` |
| `git commit` | Save changes | `git commit -m "Add feature"` |
| `git branch` | Create branch | `git branch feature-name` |
| `git checkout` | Switch branch | `git checkout main` |
| `git checkout -b` | Create & switch | `git checkout -b new-feature` |
| `git push` | Upload to remote | `git push -u origin branch` |
| `git pull` | Download changes | `git pull` |
| `git status` | Check repo state | `git status` |
| `git log` | View history | `git log --oneline` |
| `git remote` | Manage remotes | `git remote -v` |