# Jenkins Deep Dive

A practical guide to Jenkins for CI/CD automation.

## What is Jenkins?
- Open-source automation server for building, testing, and deploying software.
- Highly extensible with plugins.

# Jenkins Deep Dive

A practical guide to Jenkins for CI/CD automation.

## What is Jenkins?
- Open-source automation server for building, testing, and deploying software.
- Highly extensible with plugins.

## Key Concepts
- **Pipeline:** Sequence of automated steps (build, test, deploy)
- **Job:** A single automation task
- **Agent/Node:** Machine that runs jobs
- **Plugin:** Adds new features (e.g., Git, Docker)

## Example: Simple Jenkins Pipeline
```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps { sh 'npm run build' }
    }
    stage('Test') {
      steps { sh 'npm test' }
    }
    stage('Deploy') {
      steps { sh './deploy.sh' }
    }
  }
}
```

## Best Practices
- Use declarative pipelines
- Store pipeline as code (Jenkinsfile)
- Use credentials for secrets
- Keep jobs modular and reusable

## Troubleshooting
- Check build logs for errors
- Validate Jenkinsfile syntax
- Ensure agents have required tools

## Exercises & Next Steps
- Exercise: Run a local Jenkins instance (Docker image) and create a pipeline job using the example Jenkinsfile.
- Next: Integrate a Docker build stage and push the image to a registry using Jenkins credentials.

## Resources
- [Jenkins Documentation](https://www.jenkins.io/doc/)