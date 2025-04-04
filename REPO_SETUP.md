# GitHub Repository Setup

This guide outlines our organization's standards for setting up and managing GitHub repositories. These standards apply to all projects regardless of technology stack.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Repository Creation and Structure

### Creating a New Repository

1. Use our [repository template](https://github.com/organization/template) as a starting point
2. Follow our naming convention: `<project-name>-<component>` (e.g., `customer-portal-api`)
3. Add appropriate repository topics for discoverability
4. Enable "Issues" and "Pull requests" features

### Essential Files

Every repository must include:

- `README.md` - Project overview and setup instructions
- `.gitignore` - Appropriate for the tech stack
- `LICENSE` - Our standard license
- `CONTRIBUTING.md` - Contribution guidelines
- `.github/` folder with issue/PR templates

Example `.github/PULL_REQUEST_TEMPLATE.md`:
```markdown
## Description
<!-- Briefly describe the changes -->

## Related Issues
<!-- Link to related issues: Closes #123 -->

## Testing
<!-- How were these changes tested? -->

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] CI checks pass
```

## Branching Strategy

We follow a modified GitFlow workflow:

```
main (production)
  ↑
development (integration)
  ↑
feature/*, bugfix/*, hotfix/*, release/*
```

### Branch Types

- `main` - Production code only, deployed to production
- `development` - Integration branch, deployed to staging
- `feature/<name>` - New features (e.g., `feature/user-authentication`)
- `bugfix/<name>` - Non-critical fixes (e.g., `bugfix/form-validation`)
- `hotfix/<name>` - Critical production fixes (e.g., `hotfix/security-patch`)
- `release/<version>` - Release preparation (e.g., `release/v1.2.0`)

### Branch Naming Convention

- Use lowercase and hyphens for readability
- Include a descriptive name: `feature/add-user-profile`
- For features related to tickets/issues, include the number: `feature/user-profile-GH-123`

## Branch Protection

All repositories must have the following branch protection rules:

### For `main` branch:
- ✅ Require pull request before merging
- ✅ Require at least 1 approval
- ✅ Dismiss stale pull request approvals when new commits are pushed
- ✅ Require status checks to pass (CI workflows)
- ✅ Require branches to be up to date
- ✅ Include administrators in these restrictions

### For `development` branch:
- ✅ Require pull request before merging
- ✅ Require status checks to pass
- ❌ No direct commits allowed

## Setting Up Branch Protection

1. Go to repository settings
2. Click on "Branches" in the left sidebar
3. Click "Add branch protection rule"
4. Enter the branch name pattern (e.g., `main`)
5. Configure the required protection settings
6. Click "Create" or "Save changes"

## Repository Maintenance

### Archived Code
- Archive repositories that are no longer maintained
- Add an "ARCHIVED" notice at the top of the README
- Disable issues and pull requests

### Dependency Updates
- Enable Dependabot for automatic dependency updates
- Configure in `.github/dependabot.yml`:
  ```yaml
  version: 2
  updates:
    - package-ecosystem: "npm" # or "pip", etc.
      directory: "/"
      schedule:
        interval: "weekly"
  ```

## Common Git Workflows

### Starting a New Feature

```bash
# Update main branch
git checkout main
git pull

# Create a feature branch
git checkout -b feature/your-feature-name

# Make changes and commit
git add .
git commit -m "Descriptive commit message"

# Push to GitHub
git push -u origin feature/your-feature-name

# Then create a PR on GitHub
```

### Updating Your Branch with Latest Changes

```bash
# Get latest changes from main
git checkout main
git pull

# Switch back to your feature branch
git checkout feature/your-feature-name

# Merge main into your branch
git merge main

# Resolve any conflicts
# Then push your updated branch
git push
```