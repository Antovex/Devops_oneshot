# CI/CD Fundamentals Guide ⚡

A comprehensive, hands-on guide to Continuous Integration and Continuous Delivery/Deployment (CI/CD) for DevOps.

**🎯 Status:** LEARNING IN PROGRESS - Started September 20, 2025

---

## 🔐 Best Practices & Security
- Keep pipelines simple and fast
- Run tests in parallel where possible
- Fail fast: stop pipeline on first error
- Use secrets/credentials managers for sensitive data
- Limit permissions for CI tokens (least privilege)
- Scan images and artifacts for vulnerabilities
- Store pipelines as code in the repository

---

## 🆘 Troubleshooting & Quick Reference
- **Build fails?** Check logs and reproduce locally
- **Tests fail?** Run tests locally with same environment
- **Deployment fails?** Validate credentials and target environment

### Quick Commands
```
# Trigger a pipeline locally (GitHub Actions runner / act)
act -P ubuntu-latest=nektos/act-environments-ubuntu:18.04

# View pipeline logs in provider UI (GitHub/GitLab/Jenkins)
# Build and test locally
npm install && npm test
```

---

## Exercises & Next Steps
- Exercise 1: Create the sample GitHub Actions workflow above in a test repo and push a change to trigger it.
- Exercise 2: Create a Jenkinsfile and run it against a local Jenkins (Docker image) to observe stage logs.
- Exercise 3: Extend the GitLab CI example to push a Docker image to a registry using protected variables.
- Next Steps: Dive into `revising_cicd/cicd_deep_dive/*` modules for tool-specific workflows and advanced patterns.

---

## 📚 Deep Dive Resources

- `revising_cicd/cicd_deep_dive/github_actions.md`
- `revising_cicd/cicd_deep_dive/jenkins.md`
- `revising_cicd/cicd_deep_dive/gitlab_ci.md`

```
Source → Build → Test → Package → Deploy → Monitor
  ↓        ↓       ↓        ↓         ↓
GitHub    Jenkins  Unit    Docker    Kubernetes
Actions   Pipelines  Tests  Images    Observability
```

## 📋 Quick Navigation
- [What is CI/CD?](#-what-is-cicd)
- [Core Concepts & Benefits](#-core-concepts--benefits)
- [CI/CD Pipeline Stages](#cicd-pipeline-stages)
- [Quick Start Example (GitHub Actions)](#-quick-start-example-github-actions)
- [Building a Simple Pipeline (Jenkins/GitLab)](#building-a-simple-pipeline-jenkinsgitlab)
- [Common Use Cases](#-common-use-cases)
- [Deep Dive Resources](#-deep-dive-resources)
- [Best Practices & Security](#-best-practices--security)
- [Troubleshooting & Quick Reference](#-troubleshooting--quick-reference)
- [Exercises & Next Steps](#exercises--next-steps)

---
---

## 📋 Table of Contents
- [What is CI/CD?](#what-is-cicd)
- [Core Concepts & Benefits](#core-concepts--benefits)
- [CI/CD Pipeline Stages](#cicd-pipeline-stages)
- [Popular Tools Overview](#popular-tools-overview)
- [Quick Start Example](#quick-start-example)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Quick Reference](#quick-reference)
- [Exercises & Next Steps](#exercises--next-steps)

---

## 🚦 What is CI/CD?

**Continuous Integration (CI):**
- Developers merge code changes frequently into a shared repository.
- Automated builds and tests run to catch issues early.

**Continuous Delivery/Deployment (CD):**
- Code is automatically prepared for release to production (delivery) or actually deployed (deployment).
- Ensures software is always in a deployable state.

---

## 💡 Core Concepts & Benefits
- **Automation:** Reduces manual work, increases reliability.
- **Feedback:** Fast feedback on code quality and integration issues.
- **Quality:** Automated testing and checks improve code quality.
- **Speed:** Faster releases, less time spent on manual deployment.
- **Collaboration:** Encourages frequent, small changes and teamwork.

---

## 🛠️ CI/CD Pipeline Stages

1. **Source:** Code pushed to version control (e.g., GitHub)
2. **Build:** Compile code, resolve dependencies
3. **Test:** Run automated tests (unit, integration, etc.)
4. **Package:** Bundle application for deployment
5. **Deploy:** Release to staging/production environments
6. **Monitor:** Track application health and feedback

---

## 🚀 Popular Tools Overview
- **Jenkins:** Open-source automation server
- **GitHub Actions:** Native CI/CD for GitHub repos
- **GitLab CI:** Integrated with GitLab
- **CircleCI, Travis CI, Azure Pipelines, etc.**

---

## ⚡ Quick Start Example: GitHub Actions

Create `.github/workflows/ci.yml` in your repo:

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
```

---

## Building a Simple Pipeline (Jenkins / GitLab)

### Jenkins (Jenkinsfile)
```groovy
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'npm run build' } }
    stage('Test')  { steps { sh 'npm test' } }
    stage('Package'){ steps { sh 'docker build -t myapp .' } }
    stage('Deploy') { steps { sh './deploy.sh' } }
  }
}
```

### GitLab CI (`.gitlab-ci.yml`)
```yaml
stages:
  - build
  - test
  - package
  - deploy

build-job:
  stage: build
  script:
    - npm install
    - npm run build

test-job:
  stage: test
  script:
    - npm test

package-job:
  stage: package
  script:
    - docker build -t registry.example.com/myapp:latest .

deploy-job:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

---

## 🎯 Common Use Cases

- **CI for Pull Requests:** Run tests and linters on PRs to prevent regressions.
- **CD to Staging:** Automatically deploy to staging after passing tests.
- **Canary/Blue-Green Deployments:** Reduce risk for production updates.
- **Build + Publish Artifacts:** Build binaries, container images, and push to registries.
- **Infrastructure Validation:** Run IaC plan/validate steps in pipelines.


## ✅ Best Practices
- Keep pipelines simple and fast
- Run tests in parallel where possible
- Fail fast: stop pipeline on first error
- Use secrets for sensitive data
- Version control your pipeline configs
- Monitor and review pipeline results

---

## 🆘 Troubleshooting
- **Build fails?** Check logs for missing dependencies or syntax errors
- **Tests fail?** Run tests locally to debug
- **Deployment fails?** Check environment configs and credentials
- **Pipeline slow?** Optimize steps, use caching

---

## 📝 Quick Reference

| Stage   | Example Tool/Command         |
|---------|-----------------------------|
| Source  | `git push`                  |
| Build   | `npm run build`             |
| Test    | `npm test`                  |
| Package | `docker build -t app .`     |
| Deploy  | `kubectl apply -f deploy/`  |
| Monitor | `Prometheus`, `Grafana`     |

---

## Exercises & Next Steps
- Exercise 1: Create the sample GitHub Actions workflow above in a test repo and push a change to trigger it.
- Exercise 2: Add a caching step to speed up dependency installation.
- Next Steps: Explore deep-dive modules for Jenkins, GitHub Actions, and GitLab CI. Track your progress in `.devops_journey/progress_log.md`.
