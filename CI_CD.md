# CI/CD Pipeline Configuration

This guide outlines our organization's standards for Continuous Integration and Continuous Deployment (CI/CD) using GitHub Actions.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start

1. Create a `.github/workflows` directory in your repository
2. Add workflow files for CI and CD
3. Configure GitHub repository secrets
4. Push your changes to trigger the workflows

## Overview

Our CI/CD process follows this workflow:

1. Developers push code to feature branches
2. GitHub Actions runs CI checks on pull requests
3. After PR approval and merge to `development`, CD pipeline deploys to staging
4. After validation in staging, a PR from `development` to `main` is created
5. After PR approval and merge to `main`, CD pipeline deploys to production

## Workflow Files

Each repository should have at least two workflow files:

- `ci.yml` - For continuous integration (linting, testing, etc.)
- `cd.yml` - For continuous deployment (deployment to environments)

### Directory Structure

```
.github/
└── workflows/
    ├── ci.yml
    └── cd.yml
```

## CI Workflow Configuration

### Python Projects

Create `.github/workflows/ci.yml`:

```yaml
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

         - name: Set up Python
           uses: actions/setup-python@v5
           with:
              python-version: '3.12'
              cache: 'pip'

         - name: Install Poetry
           run: |
              curl -sSL https://install.python-poetry.org | python3 -
              echo "$HOME/.local/bin" >> $GITHUB_PATH

         - name: Install dependencies
           run: |
              poetry install

         - name: Run linters
           run: |
              poetry run black --check .
              poetry run isort --check-only --profile black .
              poetry run flake8 .
              poetry run mypy .

   test:
      name: Test
      runs-on: ubuntu-latest
      needs: lint
      steps:
         - uses: actions/checkout@v4

         - name: Set up Python
           uses: actions/setup-python@v5
           with:
              python-version: '3.12'
              cache: 'pip'

         - name: Install Poetry
           run: |
              curl -sSL https://install.python-poetry.org | python3 -
              echo "$HOME/.local/bin" >> $GITHUB_PATH

         - name: Install dependencies
           run: |
              poetry install

         - name: Run tests
           run: |
              poetry run pytest --cov=app --cov-report=xml

         - name: Upload coverage to Codecov
           uses: codecov/codecov-action@v4
           with:
              file: ./coverage.xml
              fail_ci_if_error: true
```

### Next.js Projects

Create `.github/workflows/ci.yml`:

```yaml
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
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linters
        run: |
          npm run lint
  
  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
```

### Express Projects

Create `.github/workflows/ci.yml`:

```yaml
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
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linters
        run: |
          npm run lint
  
  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
```

## CD Workflow Configuration

### Python (FastAPI) Projects

Create `.github/workflows/cd.yml`:

```yaml
name: CD

on:
  push:
    branches:
      - development
      - main

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set environment variables
        run: |
          if [[ $GITHUB_REF == 'refs/heads/main' ]]; then
            echo "ENVIRONMENT=prod" >> $GITHUB_ENV
            echo "ECR_REPOSITORY=${{ secrets.PROD_ECR_REPOSITORY }}" >> $GITHUB_ENV
          else
            echo "ENVIRONMENT=dev" >> $GITHUB_ENV
            echo "ECR_REPOSITORY=${{ secrets.DEV_ECR_REPOSITORY }}" >> $GITHUB_ENV
          fi
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.5
      
      - name: Terraform Init
        run: |
          cd infrastructure/environments/$ENVIRONMENT
          terraform init
      
      - name: Terraform Apply
        run: |
          cd infrastructure/environments/$ENVIRONMENT
          terraform apply -auto-approve -var="container_image=${{ steps.build-image.outputs.image }}"
      
      - name: Send notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          SLACK_CHANNEL: deployments
          SLACK_COLOR: ${{ job.status }}
          SLACK_TITLE: Deployment to ${{ env.ENVIRONMENT }}
          SLACK_MESSAGE: ${{ job.status == 'success' && 'Deployment succeeded! 🚀' || 'Deployment failed! 🔥' }}
```

### Next.js Projects

Create `.github/workflows/cd.yml`:

```yaml
name: CD

on:
  push:
    branches:
      - development
      - main

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set environment
        run: |
          if [[ $GITHUB_REF == 'refs/heads/main' ]]; then
            echo "ENVIRONMENT=production" >> $GITHUB_ENV
          else
            echo "ENVIRONMENT=preview" >> $GITHUB_ENV
          fi
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--${{ env.ENVIRONMENT }}'
          working-directory: ./
      
      - name: Send notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          SLACK_CHANNEL: deployments
          SLACK_COLOR: ${{ job.status }}
          SLACK_TITLE: Deployment to ${{ env.ENVIRONMENT }}
          SLACK_MESSAGE: ${{ job.status == 'success' && 'Deployment succeeded! 🚀' || 'Deployment failed! 🔥' }}
```

### Express Projects

Create `.github/workflows/cd.yml`:

```yaml
name: CD

on:
  push:
    branches:
      - development
      - main

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set environment variables
        run: |
          if [[ $GITHUB_REF == 'refs/heads/main' ]]; then
            echo "ENVIRONMENT=prod" >> $GITHUB_ENV
            echo "ECR_REPOSITORY=${{ secrets.PROD_ECR_REPOSITORY }}" >> $GITHUB_ENV
          else
            echo "ENVIRONMENT=dev" >> $GITHUB_ENV
            echo "ECR_REPOSITORY=${{ secrets.DEV_ECR_REPOSITORY }}" >> $GITHUB_ENV
          fi
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: infrastructure/task-definition.json
          container-name: app
          image: ${{ steps.build-image.outputs.image }}
          service: ${{ env.ENVIRONMENT }}-service
          cluster: ${{ env.ENVIRONMENT }}-cluster
      
      - name: Send notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          SLACK_CHANNEL: deployments
          SLACK_COLOR: ${{ job.status }}
          SLACK_TITLE: Deployment to ${{ env.ENVIRONMENT }}
          SLACK_MESSAGE: ${{ job.status == 'success' && 'Deployment succeeded! 🚀' || 'Deployment failed! 🔥' }}
```

## GitHub Repository Secrets

Set up the following secrets in your GitHub repository:

### AWS Deployment

- `AWS_ACCESS_KEY_ID` - AWS access key ID
- `AWS_SECRET_ACCESS_KEY` - AWS secret access key
- `DEV_ECR_REPOSITORY` - Development ECR repository name
- `PROD_ECR_REPOSITORY` - Production ECR repository name

### Vercel Deployment

- `VERCEL_TOKEN` - Vercel API token
- `VERCEL_ORG_ID` - Vercel organization ID
- `VERCEL_PROJECT_ID` - Vercel project ID

### Notifications

- `SLACK_WEBHOOK` - Slack webhook URL for notifications

## CI/CD Best Practices

### Incremental Deployment

1. **Development Environment**:
   - Automatically deployed on every merge to `development`
   - Used for testing new features and bug fixes

2. **Staging Environment**:
   - Deployed after QA approval in development
   - Mirrors production environment for final testing

3. **Production Environment**:
   - Deployed after approval and merge to `main`
   - Requires manual approval for critical changes

### Environment Variables

Store environment-specific variables as GitHub repository secrets:

1. Create environment variables in GitHub:
   - Go to repository settings > "Secrets and variables" > "Actions"
   - Add new repository secrets

2. Reference in workflows:
   ```yaml
   env:
     API_URL: ${{ secrets.API_URL }}
   ```

### Caching Dependencies

Cache dependencies to speed up builds:

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

## Continuous Integration Practices

### Code Quality Checks

1. **Linting**:
   - Run linters to enforce code style
   - Fail the build on linting errors

2. **Testing**:
   - Run unit tests for all code changes
   - Run integration tests for API changes

3. **Code Coverage**:
   - Set minimum code coverage requirements
   - Upload coverage reports to Codecov or SonarQube

### Security Scanning

1. **Dependency Scanning**:
   ```yaml
   - name: Run dependency security scan
     uses: snyk/actions/node@master
     with:
       args: --severity-threshold=high
     env:
       SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
   ```

2. **SAST (Static Application Security Testing)**:
   ```yaml
   - name: Run SAST scan
     uses: github/codeql-action/analyze@v2
   ```

## Continuous Deployment Practices

### Deployment Strategies

1. **Blue-Green Deployment**:
   - Deploy new version alongside old version
   - Switch traffic after validation

2. **Canary Deployment**:
   - Gradually roll out to a small percentage of users
   - Increase percentage as confidence grows

3. **Feature Flags**:
   - Deploy features behind toggles
   - Control feature availability without redeployment

### Rollback Procedures

1. **Automated Rollback**:
   ```yaml
   - name: Rollback on failure
     if: failure()
     run: |
       aws ecs update-service --cluster $CLUSTER --service $SERVICE --task-definition $PREVIOUS_TASK_DEF
   ```

2. **Manual Rollback**:
   - Provide clear instructions for manual rollback
   - Train team members on rollback procedures

## Monitoring Deployments

### Health Checks

```yaml
- name: Verify deployment
  run: |
    curl --retry 10 --retry-delay 5 --retry-connrefused https://api.example.com/health
```

### Deployment Notifications

```yaml
- name: Send deployment notification
  uses: rtCamp/action-slack-notify@v2
  env:
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
    SLACK_CHANNEL: deployments
    SLACK_COLOR: ${{ job.status }}
    SLACK_TITLE: Deployment to ${{ env.ENVIRONMENT }}
    SLACK_MESSAGE: ${{ job.status == 'success' && 'Deployment succeeded! 🚀' || 'Deployment failed! 🔥' }}
```

## Troubleshooting

### Common CI Issues

1. **Build Failures**:
   - Check dependency issues
   - Verify environment configuration
   - Review test failures

2. **Timeout Issues**:
   - Optimize build steps
   - Use caching for dependencies

3. **Permission Issues**:
   - Check GitHub token permissions
   - Verify AWS IAM permissions

### Common CD Issues

1. **Deployment Failures**:
   - Check environment variables
   - Verify infrastructure state
   - Review application logs

2. **Configuration Issues**:
   - Ensure environment-specific configuration is correct
   - Check secret management

3. **Rollback Failures**:
   - Test rollback procedures regularly
   - Document rollback steps

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS ECS Deployment Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-external.html)
- [Vercel Deployment Documentation](https://vercel.com/docs/concepts/deployments/overview)

## Version Management

GitHub Actions version updates are handled as follows:
- We maintain compatibility with the latest stable GitHub Actions releases
- New projects should always use the current standardized versions
- Existing projects should update to new GitHub Actions versions within 3 months of their release
- We regularly check for available updates to GitHub Actions