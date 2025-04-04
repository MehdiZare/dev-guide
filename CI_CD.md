# CI/CD Pipeline Configuration

This guide outlines our CI/CD standards using GitHub Actions with workspace-based configuration.

## Quick Start

1. Create `.github/workflows` directory in your repository
2. Add reusable workflow files for CI and CD
3. Configure GitHub repository secrets
4. Push changes to trigger workflows

## Overview

```
feature branch → PR → CI checks → merge to dev → deploy to staging → PR to main → deploy to production
```

## Workflow Files

Each repository should include:

- `.github/workflows/ci.yml` - Define CI pipeline
- `.github/workflows/cd.yml` - Define CD pipeline
- `.github/workflows/reusable.yml` - Reusable workflow components

## Workspace-Based Configuration

Instead of duplicating code for different environments, use workspace variables:

```yaml
# .github/workflows/reusable.yml
name: Reusable Workflows

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      action:
        required: true
        type: string

jobs:
  test:
    name: Test
    if: inputs.action == 'test'
    runs-on: ubuntu-latest
    steps:
      # Common test steps across environments
      # ...

  deploy:
    name: Deploy (${{ inputs.environment }})
    if: inputs.action == 'deploy'
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
      
      # Environment-specific configuration loaded based on workspace
      - name: Load environment config
        id: env_config
        run: |
          source .github/config/${{ inputs.environment }}.sh
          echo "ECR_REPOSITORY=$ECR_REPOSITORY" >> $GITHUB_OUTPUT
          
      # Rest of deployment steps
      # ...
```

## Usage Examples

### Python/FastAPI Projects

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [development, main]
  push:
    branches: [development, main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install poetry
      - run: poetry install
      - run: poetry run black --check .
      # Other linting steps...

  test:
    uses: ./.github/workflows/reusable.yml
    with:
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
      action: test
```

```yaml
# .github/workflows/cd.yml
name: CD

on:
  push:
    branches: [development, main]

jobs:
  deploy:
    uses: ./.github/workflows/reusable.yml
    with:
      environment: ${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
      action: deploy
    secrets: inherit
```

## Environment Configuration

Store environment-specific configuration in separate files:

```sh
# .github/config/dev.sh
export ECR_REPOSITORY=dev-repository
export AWS_REGION=us-east-1
export CLUSTER_NAME=dev-cluster
```

```sh
# .github/config/prod.sh
export ECR_REPOSITORY=prod-repository
export AWS_REGION=us-east-1
export CLUSTER_NAME=prod-cluster
```

## GitHub Repository Secrets

Configure secrets per environment:

1. Go to repository settings > "Environments" > Create environments for "dev" and "prod"
2. Add environment-specific secrets to each environment
3. Add general secrets at the repository level

## Database Migration Strategy

For zero-downtime database changes:

1. Only add new tables/columns in the first deployment
2. Deploy code that works with both old and new schema
3. Remove deprecated tables/columns after transition period

## CI Best Practices

- Run linting, security scanning, and tests in parallel
- Add coverage thresholds to enforce testing standards
- Implement branch protection rules to enforce checks

## CD Best Practices

- Use deployment concurrency limits to prevent conflicts
- Implement canary or blue-green deployments for critical systems
- Add post-deployment verification steps

## Secret Management

Export environment-specific secrets in workflows:

```yaml
steps:
  - name: Configure AWS credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
      aws-region: ${{ env.AWS_REGION }}
```

## Version Management

- Use explicit versions for all GitHub Actions
- Avoid using `@master` or `@main` tags
- Automate dependency updates with Dependabot