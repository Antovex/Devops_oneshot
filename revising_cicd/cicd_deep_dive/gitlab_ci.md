# GitLab CI Deep Dive

A practical guide to GitLab CI/CD pipelines.

## What is GitLab CI?
- Built-in CI/CD for GitLab repositories
- Automate build, test, and deploy with `.gitlab-ci.yml`

## Key Concepts
- **Pipeline:** Sequence of jobs
- **Job:** Single task (build, test, deploy)
- **Stage:** Group of jobs (run in order)
- **Runner:** Executes jobs (shared or specific)

## Example: Simple Pipeline
```yaml
stages:
  - build
  - test
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

deploy-job:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

## Best Practices
- Use environment variables for secrets
- Keep jobs small and focused
- Use `only`/`except` to control when jobs run
- Store pipeline config in repo root

## Troubleshooting
- Check pipeline logs for errors
- Validate `.gitlab-ci.yml` syntax
- Ensure runners are online

## Exercises & Next Steps
- Exercise: Create a `.gitlab-ci.yml` in a test project and verify the pipeline runs on push.
- Next: Add caching and deploy to a staging environment using environment variables.

## Resources
- [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
# GitLab CI Deep Dive

A practical guide to GitLab CI/CD pipelines.

## What is GitLab CI?
- Built-in CI/CD for GitLab repositories
- Automate build, test, and deploy with `.gitlab-ci.yml`

## Key Concepts
- **Pipeline:** Sequence of jobs
- **Job:** Single task (build, test, deploy)
- **Stage:** Group of jobs (run in order)
- **Runner:** Executes jobs (shared or specific)

## Example: Simple Pipeline
```yaml
stages:
  - build
  - test
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

deploy-job:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

## Best Practices
- Use environment variables for secrets
- Keep jobs small and focused
- Use `only`/`except` to control when jobs run
- Store pipeline config in repo root

## Troubleshooting
- Check pipeline logs for errors
- Validate `.gitlab-ci.yml` syntax
- Ensure runners are online

## Resources
- [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)