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

**Build workflows:**
- `bash-bats-build.yml` - Builds Bash projects with BATS testing framework
- `gradle-lib-build.yml` - Builds Gradle libraries with Java setup, ktlint validation, and optional code coverage
- `gradle-lib-postgres-build.yml` - Builds Gradle libraries with PostgreSQL integration
- `gradle-app-build.yml` - Builds Gradle applications
- `gradle-app-postgres-build.yml` - Builds Gradle applications with PostgreSQL integration
- `gradle-cli-build.yml` - Builds Gradle CLI applications
- `gradle-plugin-build.yml` - Builds Gradle plugins
- `python-package-build.yml` - Builds Python packages with pytest and ruff linting

**Release workflows:**
- `gradle-maven-central-release.yml` - Releases Gradle artifacts to Maven Central with GPG signing
- `gradle-plugin-release.yml` - Publishes Gradle plugins to Gradle Plugin Portal
- `gradle-github-release.yml` - Creates GitHub releases for Gradle projects
- `python-release-build.yml` - Builds Python package distributions for release
- `python-release.yml` - Creates GitHub releases for Python projects with version bumping

**Deployment workflows:**
- `gradle-azure-app-service-deploy.yml` - Deploys applications to Azure App Service

**Code quality workflows:**
- `java-kotlin-code-coverage.yml` - Generates and uploads code coverage reports for Java/Kotlin
- `java-kotlin-postgres-code-coverage.yml` - Generates code coverage for Java/Kotlin with PostgreSQL
- `python-code-coverage.yml` - Generates and uploads code coverage reports for Python
- `java-kotlin-codeql-analyze.yml` - CodeQL security analysis for Java/Kotlin
- `python-codeql-analyze.yml` - CodeQL security analysis for Python

**Utility workflows:**
- `workflow-cancel.yml` - Cancels in-progress workflows
- `workflow-update-license-copyright.yml` - Updates license copyright headers
- `workflow-update-actions.yml` - Updates GitHub Actions versions in workflows
- `workflow-clear-packages.yml` - Clears old GitHub packages
- `gradle-dependencies-submit.yml` - Submits Gradle dependency graph to GitHub
- `local-update-actions.yml` - Local workflow wrapper for updating GitHub Actions versions
- `local-update-license-copyright.yml` - Local workflow wrapper for updating license copyright

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
   - `build.yml` - Build and test with pytest and ruff linting
   - `release.yml` - Build package, publish to PyPI, and create GitHub release
   - `coverage.yml` - Code coverage reporting
   - `codeql.yml` - Security analysis
   - `cancel.yml` - Cancel in-progress workflows
   - `update-actions.yml` - Auto-update GitHub Actions versions
   - `update-license-copyright.yml` - Update copyright headers

2. Configure repository secrets:
   - `CODECOV_TOKEN` - Codecov integration
   - `WORKFLOW_TOKEN` - GitHub token with workflow permissions

3. Configure repository variables:
   - `GIT_EMAIL` - Git user email for automated commits
   - `GIT_USER` - Git username for automated commits

4. Set up PyPI trusted publishing:
   - Configure PyPI trusted publisher for your repository
   - No API token needed - uses OpenID Connect (OIDC)

## Common Workflow Templates

The following workflow templates are available in the `workflows/` directory for each project type:

### Standard Workflows (All Project Types)

- **`build.yml`** - Triggered on all branches except `main`, runs tests and validation
  - Gradle: Runs `./gradlew clean build` with ktlint code style checks
  - Python: Runs `pytest` with ruff linting and formatting checks
  - Bash: Runs BATS test suite

- **`release.yml`** - Triggered when a PR is merged to `main` with the `release` label
  - Gradle libraries: Publishes to Maven Central with GPG signing
  - Gradle plugins: Publishes to Gradle Plugin Portal
  - Python packages: Builds and publishes to PyPI with trusted publishing
  - Creates a GitHub release with version tag

- **`coverage.yml`** - Runs code coverage analysis and uploads to Codecov
  - Gradle: Uses JaCoCo for Java/Kotlin coverage
  - Python: Uses pytest-cov for Python coverage
  - PostgreSQL projects: Includes database integration test coverage

- **`codeql.yml`** - Runs GitHub CodeQL security analysis on schedule and push

- **`cancel.yml`** - Cancels in-progress workflows when new commits are pushed

- **`update-license-copyright.yml`** - Updates license copyright headers on schedule

- **`dependencies-submit.yml`** - Submits dependency graph to GitHub (Gradle projects only)

### Application-Specific Workflows

The following templates are available for application projects (`app-postgres`, `app-web-mvc`):

- **`deploy-dev.yml`** - Deploys to dev environment when PR merged to `main` with `deploy dev` label
- **`deploy-prod.yml`** - Deploys to prod environment when PR merged to `main` with `deploy prod` label
- **`clear-packages.yml`** - Clears old GitHub packages on schedule

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
