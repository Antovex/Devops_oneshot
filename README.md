<div align="center">

# DevOps Onesho## 📦 What's Inside
- `revising_git/` — Git Ess## 🌟 Featured Cheatsheets

### Git Essentials (Mastered ✅)tials Cheat Sheet → see [`revising_git/git.md`](revising_git/git.md)
- `revising_linux/` — Linux/Unix Commands Reference → see [`revising_linux/linux_commands.md`](revising_linux/linux_commands.md)
- `.devops_journey/` — Learning artifacts:
  - [`cheatsheets.md`](.devops_journey/cheatsheets.md) — consolidated topic cheatsheets
  - [`journey_summary.md`](.devops_journey/journey_summary.md) — high-level progress
  - [`progress_log.md`](.devops_journey/progress_log.md) — day-by-day notes
- `.github/prompts/` — Assistant prompt specs:
  - [`devops_assistant.prompt.md`](.github/prompts/devops_assistant.prompt.md)
  - [`cheetsheet.prompt.md`](.github/prompts/cheetsheet.prompt.md)
  - [`progress.prompt.md`](.github/prompts/progress.prompt.md)ng Journey 🚀

Comprehensive, beautifully crafted notes and cheatsheets from a focused DevOps mastery sprint.

<a href="#-progress--status"><img alt="Status" src="https://img.shields.io/badge/status-active-brightgreen?style=for-the-badge"></a>
<a href="#-whats-inside"><img alt="Topics" src="https://img.shields.io/badge/topics-2_mastered-blue?style=for-the-badge"></a>
<a href="#-progress--status"><img alt="Last Updated" src="https://img.shields.io/badge/last_update-2025--09--18-purple?style=for-the-badge"></a>
<a href="#-how-i-work"><img alt="Conventions" src="https://img.shields.io/badge/commits-conventional-ff69b4?style=for-the-badge"></a>

</div>

---

A curated repository to document and reinforce DevOps concepts through concise cheatsheets, guided prompts, and a running progress log. Built for clarity, speed, and consistency.


## ✨ Highlights
- **Cheatsheets that matter:** Minimal, actionable, copy-paste friendly.
- **Guided assistant prompts:** Purpose-built prompts in `.github/prompts/` to streamline learning.
- **Transparent progress:** Daily log and summary in `.devops_journey/`.
- **Polished design:** Clear structure, badges, and modern formatting for quick scanning.


## 🧭 Table of Contents
- [Highlights](#-highlights)
- [What’s Inside](#-whats-inside)
- [Getting Started](#-getting-started)
- [Progress & Status](#-progress--status)
- [Featured Cheatsheet](#-featured-cheatsheet)
- [Topics Roadmap](#-topics-roadmap)
- [How I Work](#-how-i-work)
- [Prompts (for Copilot Chat)](#-prompts-for-copilot-chat)
- [Contributing](#-contributing)
- [Resources](#-resources)
- [License](#-license)


## 📦 What’s Inside
- `revising_git/` — Git Essentials Cheat Sheet → see [`revising_git/git.md`](revising_git/git.md)
- `.devops_journey/` — Learning artifacts:
  - [`cheatsheets.md`](.devops_journey/cheatsheets.md) — consolidated topic cheatsheets
  - [`journey_summary.md`](.devops_journey/journey_summary.md) — high-level progress
  - [`progress_log.md`](.devops_journey/progress_log.md) — day-by-day notes
- `.github/prompts/` — Assistant prompt specs:
  - [`devops_assistant.prompt.md`](.github/prompts/devops_assistant.prompt.md)
  - [`cheetsheet.prompt.md`](.github/prompts/cheetsheet.prompt.md)
  - [`progress.prompt.md`](.github/prompts/progress.prompt.md)


## 🚀 Getting Started
Use this repo as a fast reference and structured learning log.

- Browse cheatsheets: start with [`revising_git/git.md`](revising_git/git.md) or [`revising_linux/linux_commands.md`](revising_linux/linux_commands.md)
- Keep progress flowing in [`.devops_journey/progress_log.md`](.devops_journey/progress_log.md)
- Summarize milestones in [`.devops_journey/journey_summary.md`](.devops_journey/journey_summary.md)

If you’re using Copilot Chat, these lightweight commands keep momentum:

```text
Log: <short summary>
Cheatsheet: <topic>
Suggest next
```


## 📈 Progress & Status
- Status: 🟢 Active Learning
- Mastered Topics: 2
- Currently Learning: 0
- Latest Completed: Linux/Unix Commands — ✅ MASTERED (Sept 18, 2025)
- Previous: Git Fundamentals — ✅ MASTERED (Sept 17, 2025)
- Overview: see [`journey_summary.md`](.devops_journey/journey_summary.md)
- Daily notes: see [`progress_log.md`](.devops_journey/progress_log.md)


## 🌟 Featured Cheatsheet
A taste of what’s inside the Git essentials file.

```bash
# One-time setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Local workflow
git init
git status
git add .
git commit -m "message"
git log --oneline

# Branching
git checkout -b feature-name

# Remote
git branch -M main
git remote add origin <url>
git push -u origin main
```

### Linux/Unix Commands (Mastered ✅)
```bash
# System Information
free -h                     # Memory usage
df -h                       # Disk space
ps aux                      # Running processes
top                         # Real-time monitor

# File Operations
ls -la                      # List files detailed
chmod 755 file              # Set permissions
cp -r source dest           # Copy recursively
grep -r "pattern" dir       # Search in files

# Network
ping -c 4 google.com        # Test connectivity
curl -I URL                 # Get HTTP headers
netstat -tuln               # Show ports
```

Full references: [`revising_git/git.md`](revising_git/git.md) | [`revising_linux/linux_commands.md`](revising_linux/linux_commands.md)


## 🗺️ Topics Roadmap
- [x] Git Fundamentals
- [x] Linux/Unix Command Line
- [ ] Docker & Containerization
- [ ] CI/CD Fundamentals
- [ ] Infrastructure as Code (Terraform)
- [ ] Configuration Management (Ansible)
- [ ] Monitoring & Logging
- [ ] Kubernetes & Orchestration
- [ ] Cloud Platforms (AWS/Azure/GCP)
- [ ] Security in DevOps (DevSecOps)


## 🧪 How I Work
- Commit messages: follow the rule — “If applied, this commit will …”
- Conventional-types encouraged: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Branch names: `feature/<name>`, `fix/<name>`
- Workflow: small commits, pull before push, review with `git diff`

Example messages:
```bash
git commit -m "feat: add user profile page"
git commit -m "fix: resolve login button alignment"
```


## 🧩 Prompts (for Copilot Chat)
<details>
<summary>Click to view prompt specs</summary>

- Assistant role: [`.github/prompts/devops_assistant.prompt.md`](.github/prompts/devops_assistant.prompt.md)
- Cheatsheet generator: [`.github/prompts/cheetsheet.prompt.md`](.github/prompts/cheetsheet.prompt.md)
- Progress logger: [`.github/prompts/progress.prompt.md`](.github/prompts/progress.prompt.md)

</details>


## 🤝 Contributing
This is primarily a personal learning repository. Suggestions and improvements are welcome via issues/PRs, but the scope stays focused on concise learning artifacts.
