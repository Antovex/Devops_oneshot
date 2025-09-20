# GitHub Actions Deep Dive

A hands-on guide to CI/CD with GitHub Actions.

## What is GitHub Actions?
- Native CI/CD for GitHub repositories
- Automate workflows for build, test, deploy

# GitHub Actions Deep Dive

A hands-on guide to CI/CD with GitHub Actions.

## What is GitHub Actions?
- Native CI/CD for GitHub repositories
- Automate workflows for build, test, deploy

## Key Concepts
- **Workflow:** Automated process triggered by events
- **Job:** Set of steps run on a runner
- **Step:** Individual command or action
- **Action:** Reusable automation (official or community)

## Example: Node.js Workflow
```yaml
name: Node.js CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
```

## Best Practices
- Use official actions where possible
- Keep workflows in `.github/workflows/`
- Use secrets for sensitive data
- Cache dependencies for speed

## Troubleshooting
- Check Actions tab for logs
- Validate YAML syntax
- Ensure correct event triggers

## Exercises & Next Steps
- Exercise: Create the Node.js workflow in a test repo and push a change to verify the run.
- Next: Add a job that builds a Docker image and pushes it to a registry using secrets.

## Resources
- [GitHub Actions Docs](https://docs.github.com/en/actions)