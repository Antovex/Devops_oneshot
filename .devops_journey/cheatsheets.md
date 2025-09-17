# DevOps Cheatsheets

This markdown file will contain concise cheatsheets for each DevOps topic you revise. Cheatsheets will be organized by topic and updated as you progress.

---

## Cheatsheet Index

- [Git Fundamentals](#git-fundamentals) ✅ **MASTERED**

---

## Git Fundamentals ✅ **MASTERED**

### Essential Commands
```bash
# Setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Local Workflow
git init                    # Initialize repository
git status                  # Check status
git add .                   # Stage all files
git commit -m "message"     # Commit changes
git log --oneline           # View history

# Branching
git branch feature-name     # Create branch
git checkout feature-name   # Switch to branch
git checkout -b new-feature # Create and switch to branch

# Remote Repository
git branch -M main          # Rename branch to main
git remote add origin <url> # Link to remote repository
git push -u origin main     # Push and set upstream
git push                    # Subsequent pushes
git pull                    # Pull latest changes

# Navigation
git checkout <commit-hash>  # Go to specific commit
git checkout main           # Return to main branch
```

### Feature Branch Workflow
```bash
git pull                           # Get latest changes
git checkout -b feature-login      # Create feature branch
git add .                          # Stage changes
git commit -m "add login feature"  # Commit with good message
git push -u origin feature-login   # Push feature branch
```

### Commit Message Convention
**Rule:** "If applied to the codebase, this commit will ___"

Examples:
- `git commit -m "add user authentication"`
- `git commit -m "fix navigation menu bug"`
- `git commit -m "update API documentation"`

### Workflow Summary
- **Local:** `init` → `add` → `commit` → `log`
- **Branching:** `checkout -b` → develop → `add` → `commit` → `push -u`
- **Remote:** `branch -M main` → `remote add origin` → `push -u origin main`

### Quick Reference
- **Working Directory** → `git add` → **Staging Area** → `git commit` → **Local Repository** → `git push` → **Remote Repository**
- Always run `git status` before committing
- Pull before push to avoid conflicts
- Use descriptive branch names (`feature-login`, `fix-payment`)
- Write commit messages that complete: "If applied, this commit will ___"

### 🎯 **Mastery Status: COMPLETE**
You've mastered all fundamental Git concepts including local workflows, branching, remote repositories, and team collaboration patterns. Ready for production use!

---

## 🚀 Next Topic Preparation

Ready to add your next DevOps topic here! Common next topics in DevOps learning paths:
- Linux/Unix Command Line
- Docker & Containerization
- CI/CD Pipelines
- Infrastructure as Code
- Monitoring & Logging
- Cloud Platforms (AWS/Azure/GCP)

---

