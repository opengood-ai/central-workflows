# OpenGood GitHub Actions Reusable Workflows

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/opengood-ai/central-workflows/main/LICENSE)

Centralized, reusable GitHub Actions workflows for OpenGood CI/CD pipelines. This repository provides standardized workflow definitions for building, testing, releasing, and deploying projects across multiple languages and platforms.

## Overview

This repository serves as the single source of truth for CI/CD workflows used across the OpenGood organization. It provides:

- **Reusable Workflows**: Centralized workflow definitions in `.github/workflows/` that can be called from any repository
- **Copy-Paste Templates**: Ready-to-use workflow files in `workflows/` organized by platform and project type
- **Standardized Processes**: Consistent build, test, release, and deployment patterns across all projects
- **Reduced Duplication**: Changes to workflows propagate to all consuming repositories automatically

## Architecture

### Reusable Workflow Implementations (`.github/workflows/`)

These are the core workflow definitions that perform the actual CI/CD work. They use the `workflow_call` trigger and accept inputs and secrets as parameters.

**Key workflows:**
- `gradle-lib-build.yml` - Builds Gradle libraries with Java setup, ktlint validation, and optional code coverage
- `gradle-maven-central-release.yml` - Releases Gradle artifacts to Maven Central with GPG signing
- `python-package-build.yml` - Builds Python packages with pytest and ruff linting
- `python-py-pi-release.yml` - Publishes Python packages to PyPI
- `gradle-plugin-release.yml` - Publishes Gradle plugins to Gradle Plugin Portal
- `gradle-azure-app-service-deploy.yml` - Deploys applications to Azure App Service
- `java-kotlin-code-coverage.yml` - Generates and uploads code coverage reports
- `workflow-cancel.yml` - Cancels in-progress workflows
- `workflow-update-license-copyright.yml` - Updates license copyright headers

### Copy-Paste Templates (`workflows/`)

Thin wrapper workflows organized by language/platform and project type. These serve as starting points for consuming repositories and reference the reusable workflows.

**Supported platforms and project types:**

| Platform | Project Types |
|----------|---------------|
| **Bash** | `bats` - Projects using BATS testing framework |
| **Gradle** | `lib` - Java/Kotlin libraries<br>`lib-postgres` - Libraries with PostgreSQL<br>`app-postgres` - Applications with PostgreSQL<br>`app-web-mvc` - Web MVC applications<br>`cli` - Command-line applications<br>`plugin` - Gradle plugins |
| **Python** | `package` - Python packages |

## Usage

### Using in Your Repository

1. **Copy the appropriate workflow templates** from `workflows/{platform}/{project-type}/` to your repository's `.github/workflows/` directory

2. **Configure required secrets and variables** in your repository settings

3. **The workflows will reference** the centralized implementations using:

```yaml
jobs:
  build:
    uses: opengood-aio/central-workflows/.github/workflows/gradle-lib-build.yml@main
    with:
      run-code-coverage: true
      run-gradle-validation: true
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
```

### Example: Setting Up a Gradle Library

1. Copy workflows from `workflows/gradle/lib/`:
   - `build.yml` - Build and test on every push
   - `release.yml` - Release to Maven Central on PR merge with 'release' label
   - `coverage.yml` - Code coverage reporting
   - `codeql.yml` - Security analysis
   - `cancel.yml` - Cancel in-progress workflows

2. Configure repository secrets:
   - `CODECOV_TOKEN` - Codecov integration
   - `GPG_SIGNING_KEY_ID` - GPG key ID for Maven Central
   - `GPG_SIGNING_PASSWORD` - GPG key password
   - `GPG_SIGNING_PRIVATE_KEY` - GPG private key
   - `MAVEN_CENTRAL_REPO_USERNAME` - Maven Central username
   - `MAVEN_CENTRAL_REPO_PASSWORD` - Maven Central password
   - `WORKFLOW_TOKEN` - GitHub token with workflow permissions

3. Configure repository variables:
   - `GIT_EMAIL` - Git user email for automated commits
   - `GIT_USER` - Git username for automated commits

### Example: Setting Up a Python Package

1. Copy workflows from `workflows/python/package/`:
   - `build.yml` - Build and test with pytest
   - `release.yml` - Publish to PyPI
   - `coverage.yml` - Code coverage
   - `codeql.yml` - Security analysis

2. Configure repository secrets:
   - `CODECOV_TOKEN` - Codecov integration
   - `PYPI_API_TOKEN` - PyPI publishing token
   - `WORKFLOW_TOKEN` - GitHub token with workflow permissions

## Common Workflows

### Build Workflows
Triggered on all branches except `main`, these run tests and validation:
- Gradle: Runs `./gradlew clean build` with ktlint code style checks
- Python: Runs `pytest` with ruff linting and formatting checks
- Bash: Runs BATS test suite

### Release Workflows
Triggered when a PR is merged to `main` with the `release` label:
- Extracts version from `gradle.properties` or `pyproject.toml`
- Builds and publishes artifacts to Maven Central, PyPI, or Gradle Plugin Portal
- Creates a GitHub release with the version tag

### Deploy Workflows
Triggered when a PR is merged to `main` with a `deploy {env}` label:
- Downloads release artifacts from GitHub Packages
- Deploys to Azure App Service for the specified environment
- Configures app settings from repository secrets

### Coverage Workflows
Run code coverage analysis and upload results to Codecov:
- Gradle: Uses JaCoCo for Java/Kotlin coverage
- Python: Uses pytest-cov for Python coverage
- PostgreSQL projects: Runs coverage with database integration tests

### CodeQL Workflows
Run GitHub CodeQL security analysis on a schedule and for every push

## Default Configurations

| Tool | Default Version | Configurable |
|------|----------------|--------------|
| Java | 21 (Temurin) | Yes, via inputs |
| Python | 3.13 | Yes, via inputs |
| ktlint | 1.2.1 | No |
| ruff | Latest | No |

## Modifying Workflows

### Updating Reusable Workflows (`.github/workflows/`)

Changes to these files affect all consuming repositories:
1. Test changes thoroughly before merging
2. Maintain backward compatibility with existing inputs/secrets
3. Document any breaking changes
4. Consider versioning workflow calls using tags instead of `@main`

### Creating New Project Type Templates

1. Create a new directory: `workflows/{platform}/{project-type}/`
2. Copy workflows from a similar project type as a starting point
3. Update the `uses:` references to the appropriate reusable workflows
4. Add required inputs and secrets for the workflow type
5. Follow naming conventions: `{workflow-type}.yml`

## Contributing

When contributing to this repository:
1. Ensure workflows are tested in a sandbox repository before merging
2. Update this README if adding new workflow types
3. Follow existing patterns for consistency
4. Document all required secrets and variables

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
