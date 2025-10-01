<div align="center">

# DevOps Learning Path — Complete Guide 🚀

**A structured, hands-on approach to mastering DevOps fundamentals**

Learn DevOps the right way with comprehensive cheatsheets, practical examples, and a proven learning framework.

<a href="#-learning-path"><img alt="Learning Path" src="https://img.shields.io/badge/learning_path-structured-brightgreen?style=for-the-badge"></a>
<a href="#-topics-covered"><img alt="Topics" src="https://img.shields.io/badge/topics-10+_planned-blue?style=for-the-badge"></a>
<a href="#-progress-tracking"><img alt="Progress" src="https://img.shields.io/badge/progress-trackable-purple?style=for-the-badge"></a>
<a href="#-get-started"><img alt="Beginner Friendly" src="https://img.shields.io/badge/level-beginner_friendly-ff69b4?style=for-the-badge"></a>

</div>

---

## 🎯 What This Repository Offers

This repository provides a **complete, structured learning path** for DevOps fundamentals. Whether you're starting from scratch or looking to solidify your knowledge, you'll find:

- **📚 Comprehensive Guides**: Detailed explanations with practical examples
- **⚡ Quick Reference Cheatsheets**: Essential commands at your fingertips
- **🛤️ Structured Learning Path**: Follow a proven sequence of topics
- **📊 Progress Tracking**: Monitor your learning journey
- **🎯 Hands-On Focus**: Real-world examples and best practices

## 🧭 Table of Contents
- [What This Repository Offers](#-what-this-repository-offers)
- [Learning Path](#-learning-path)
- [Get Started](#-get-started)
- [Repository Structure](#-repository-structure)
- [Topics Covered](#-topics-covered)
- [How to Use This Repository](#-how-to-use-this-repository)
- [Progress Tracking](#-progress-tracking)
- [Contributing](#-contributing)
- [Resources](#-resources)
- [License](#-license)

## 🛤️ Learning Path

Follow this structured path to build strong DevOps foundations:

### Phase 1: Foundations (Essential)
1. **[Git & Version Control](revising_git/git.md)** ✅ - Master collaboration and code management
2. **[Linux/Unix Commands](revising_linux/linux_commands.md)** ✅ - Command-line proficiency for operations

### Phase 2: Containerization & Automation
3. **[Docker & Containerization](revising_docker/docker_guide.md)** ✅ - Package applications consistently, master images, containers, volumes, networking, and Dockerfile creation.  
   - Deep-dive modules: [Images & Containers](revising_docker/docker_deep_dive/images_and_containers.md), [Volumes & Persistence](revising_docker/docker_deep_dive/volumes_and_persistence.md), [Networking](revising_docker/docker_deep_dive/networking.md), [Dockerfile Instructions](revising_docker/docker_deep_dive/dockerfile_instructions.md)
4. **[CI/CD Fundamentals](revising_cicd/cicd_guide.md)** ✅ - Automate testing and deployment
   - Deep-dive modules: [Jenkins](revising_cicd/cicd_deep_dive/jenkins.md), [GitHub Actions](revising_cicd/cicd_deep_dive/github_actions.md), [GitLab CI](revising_cicd/cicd_deep_dive/gitlab_ci.md)

### Phase 3: Infrastructure & Configuration
5. **[Infrastructure as Code](revising_iac/iac_guide.md)** ✅ - Manage infrastructure with code (Terraform, CloudFormation, Ansible)
   - Deep-dive modules: [Terraform](revising_iac/iac_deep_dive/terraform.md), [CloudFormation](revising_iac/iac_deep_dive/cloudformation.md), [Ansible](revising_iac/iac_deep_dive/ansible.md)
6. **[Configuration Management](revising_config_management/configuration_guide.md)** ⚡ - Automate system configuration (Ansible, Chef, Puppet)
   - Deep-dive modules: [Advanced Ansible](revising_config_management/config_deep_dive/ansible_advanced.md), [Chef](revising_config_management/config_deep_dive/chef.md), [Puppet](revising_config_management/config_deep_dive/puppet.md)

### Phase 4: Monitoring & Cloud
7. **Monitoring & Logging** 📋 - Observe system health and performance
8. **Cloud Platforms** 📋 - Work with AWS, Azure, or GCP

### Phase 5: Advanced Topics
9. **Kubernetes & Orchestration** 📋 - Container orchestration at scale
10. **Security in DevOps** 📋 - Integrate security practices

**Legend**: ✅ Complete | ⚡ In Progress | 📋 Planned

## 🚀 Get Started

### Option 1: Follow the Complete Path
1. **Clone this repository**
   ```bash
   git clone https://github.com/Antovex/Devops_oneshot.git
   cd Devops_oneshot
   ```

2. **Start with Git fundamentals**
   - Read [`revising_git/git.md`](revising_git/git.md)
   - Practice the commands in the examples
   - Complete the workflow exercises

3. **Move to Linux/Unix commands**
   - Study [`revising_linux/linux_commands.md`](revising_linux/linux_commands.md)
   - Practice on your system or a virtual machine
   - Try the hands-on scenarios

4. **Continue with Docker & Containerization**
   - Read [`revising_docker/docker_guide.md`](revising_docker/docker_guide.md) ✅
   - Explore deep-dive modules for advanced topics ✅
   - Use the [Docker cheatsheet](.devops_journey/cheatsheets.md#docker--containerization-✅-revised) for quick reference ✅

5. **Start CI/CD Fundamentals**
   - Read [`revising_cicd/cicd_guide.md`](revising_cicd/cicd_guide.md) ✅
   - Explore deep-dive modules for Jenkins, GitHub Actions, and GitLab CI ✅

6. **Begin Infrastructure as Code**
   - Read [`revising_iac/iac_guide.md`](revising_iac/iac_guide.md) ✅
   - Explore deep-dive modules for Terraform, CloudFormation, and Ansible ✅
   - Set up cloud provider accounts (AWS/Azure/GCP) ✅

7. **Start Configuration Management**
   - Read [`revising_config_management/configuration_guide.md`](revising_config_management/configuration_guide.md) ⚡
   - Explore deep-dive modules for Ansible, Chef, and Puppet ⚡
   - Practice automation with real servers ⚡

7. **Track your progress**
   - Use [`.devops_journey/progress_log.md`](.devops_journey/progress_log.md) as a template
   - Update your learning status as you progress

### Option 2: Use as Reference
- Jump to any topic you need
- Use the quick reference sections
- Copy-paste commands and examples

## 👣 How to Follow and Track Your Own Journey

Want to use this repo as your own DevOps learning log? Here’s how:

1. **Fork this repository** to your own GitHub account.
2. **Clone your fork locally** and create a new branch for your learning journey.
3. **As you study each topic:**
   - Read the main guide and cheatsheet for that topic.
   - Practice the commands and exercises on your own system.
   - Add your own notes/examples to the topic files or create a `notes/` folder for personal insights.
   - Log your daily or weekly progress in `.devops_journey/progress_log.md` using the provided template.
   - Mark topics as ✅ Done in your README as you finish them.
4. **Customize your journey:**
   - Add new topics, tools, or workflows you encounter.
   - Update the learning path to match your goals.
   - Share your progress with others or keep it private.
5. **(Optional) Contribute back:**
   - If you improve a guide or add a new topic, consider submitting a pull request to help others!

**Template for your progress log:**
```markdown
### 📅 Day X - [DATE]
**Topic:** [Topic Name]
**Status:** [⚡ In Progress | ✅ Done | 📋 Planned]
**Resource:** [Learning resource used]

**What I Learned:**
- Key concept 1
- Key concept 2

**Next Steps:**
- [ ] Practice exercise 1
- [ ] Review and reinforce
```

> **Tip:** You can use this approach for any technical learning path—just fork, track, and iterate!

## 📁 Repository Structure

```
DevOps_Learning_Path/
├── README.md                      # This guide
├── revising_git/                  # Git & Version Control
│   └── git.md                     # Complete Git reference
├── revising_linux/                # Linux/Unix Commands
│   └── linux_commands.md         # Complete Linux reference
├── revising_cicd/                 # CI/CD Fundamentals
│   ├── cicd_guide.md             # Main CI/CD guide with fundamentals
│   └── cicd_deep_dive/           # Specialized deep-dive modules
│       ├── jenkins.md            # Jenkins pipelines and automation
│       ├── github_actions.md     # GitHub Actions workflows
│       └── gitlab_ci.md          # GitLab CI/CD configuration
├── revising_iac/                 # Infrastructure as Code
│   ├── iac_guide.md             # Main IaC guide with fundamentals
│   └── iac_deep_dive/           # Specialized deep-dive modules
│       ├── terraform.md          # Terraform state management and modules
│       ├── cloudformation.md     # CloudFormation templates and stacks
│       └── ansible.md           # Ansible automation and configuration
├── revising_config_management/   # Configuration Management
│   ├── configuration_guide.md   # Main Config Management guide
│   └── config_deep_dive/        # Specialized deep-dive modules
│       ├── ansible_advanced.md   # Advanced Ansible patterns and enterprise
│       ├── chef.md              # Chef cookbooks and enterprise features
│       └── puppet.md            # Puppet manifests and enterprise management
├── .devops_journey/               # Learning tracking
│   ├── cheatsheets.md            # Consolidated quick reference (see Docker section)
│   ├── journey_summary.md        # Progress overview
│   └── progress_log.md           # Detailed learning log
└── .github/prompts/              # AI assistant prompts
    ├── devops_assistant.prompt.md
    ├── cheetsheet.prompt.md
    └── progress.prompt.md
```

## 📚 Topics Covered

### Completed Topics
- **[Git Fundamentals](revising_git/git.md)**: Version control, branching, collaboration, workflows ✅
- **[Linux/Unix Commands](revising_linux/linux_commands.md)**: System administration, file operations, process management, networking ✅
- **[Docker & Containerization](revising_docker/docker_guide.md)**: Containerization fundamentals, Docker images, volumes, networking, Dockerfile creation, and production best practices ✅
- **[CI/CD Fundamentals](revising_cicd/cicd_guide.md)**: Pipeline automation, Jenkins, GitHub Actions, GitLab CI, deployment strategies ✅
- **[Infrastructure as Code](revising_iac/iac_guide.md)**: Terraform, CloudFormation, and Ansible for infrastructure automation ✅

### Currently Learning
- **[Configuration Management](revising_config_management/configuration_guide.md)** ⚡: Learning Ansible, Chef, and Puppet for automated system configuration

### Upcoming Topics
- **Monitoring & Logging**: Prometheus, Grafana, ELK Stack
- **Cloud Platforms**: AWS, Azure, GCP fundamentals
- **Kubernetes**: Container orchestration, deployments, services
- **DevSecOps**: Security integration, vulnerability scanning

## 📖 How to Use This Repository

### For Learners
1. **Start with Phase 1** - Build strong foundations first
2. **Practice hands-on** - Don't just read, try the commands
3. **Track progress** - Use the progress tracking templates
4. **Move sequentially** - Each phase builds on previous knowledge

### For Instructors
1. **Use as curriculum** - Topics are logically sequenced
2. **Assign sections** - Each topic is self-contained
3. **Check progress** - Students can track their own learning
4. **Customize content** - Fork and modify for your needs

### For Teams
1. **Onboarding resource** - Help new team members learn DevOps
2. **Reference guide** - Quick command lookup during work
3. **Standardization** - Common knowledge base for the team
4. **Skill assessment** - Check team members' DevOps knowledge

## 📊 Progress Tracking

### Personal Learning Log
Use the templates in `.devops_journey/` to track your progress:
- **Daily entries** in `progress_log.md`
- **Milestone tracking** in `journey_summary.md`
- **Quick reference** in `cheatsheets.md`

### For Teams/Organizations
- Fork this repository for your team
- Customize the learning path for your needs
- Track team progress through individual logs
- Add company-specific tools and practices

## 🎯 Learning Philosophy

This repository follows these principles:
- **Hands-on learning** - Practice with real examples
- **Incremental progress** - Build knowledge step by step
- **Practical focus** - Real-world scenarios and best practices
- **Self-paced** - Learn at your own speed
- **Reference-friendly** - Easy to find information quickly

## 🤝 Contributing

We welcome contributions to improve this learning resource!

### How to Contribute
1. **Fork the repository**
2. **Create a feature branch**
3. **Add improvements** (new topics, better examples, corrections)
4. **Submit a pull request**

### Contribution Ideas
- New topic guides following the established format
- Better examples and practical scenarios  
- Corrections to existing content
- Translations to other languages
- Learning exercises and quizzes

## 📚 Resources

### Essential Tools
- **Git**: [Download](https://git-scm.com/) | [Documentation](https://git-scm.com/doc)
- **Linux VM**: [VirtualBox](https://www.virtualbox.org/) | [VMware](https://www.vmware.com/)
- **Cloud Platforms**: [AWS Free Tier](https://aws.amazon.com/free/) | [Azure Free Account](https://azure.microsoft.com/free/)

### External Learning Resources
- [DevOps Roadmap](https://roadmap.sh/devops) - Visual learning path
- [The Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/1942788290) - DevOps philosophy
- [Docker Documentation](https://docs.docker.com/) - Official Docker guides

### Video Resources
- Primary reference: [DevOps Course](https://www.youtube.com/watch?v=H5FAxTBuNM8)
- Additional tutorials linked in individual topic files

## 📝 License

This project is open source and available under the [MIT License](https://opensource.org/licenses/MIT).

Feel free to use this content for:
- Personal learning
- Educational purposes
- Training materials
- Team onboarding

---

<div align="center">

**Ready to start your DevOps journey?** 🚀

[Begin with Git Fundamentals](revising_git/git.md) | [Jump to Linux Commands](revising_linux/linux_commands.md) | [View Progress Tracking](.devops_journey/README.md)

*Last updated: October 1, 2025*

</div>