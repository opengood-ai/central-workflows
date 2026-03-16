# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains reusable GitHub Actions workflows for OpenGood CI/CD pipelines. It provides centralized workflow definitions that can be called from other repositories in the OpenGood organization.

## Repository Architecture

The repository has a two-layer structure:

### 1. Reusable Workflow Implementations (`.github/workflows/`)

These are the actual workflow definitions that do the work. They use `workflow_call` triggers and accept inputs/secrets as parameters. Examples:
- `gradle-lib-build.yml` - Builds Gradle libraries with Java setup, ktlint, and optional coverage
- `gradle-maven-central-release.yml` - Releases Gradle artifacts to Maven Central with GPG signing
- `python-package-build.yml` - Builds Python packages with pytest, ruff linting
- `gradle-azure-app-service-deploy.yml` - Deploys Gradle apps to Azure App Service
- `workflow-cancel.yml` - Cancels running workflows
- `workflow-update-license-copyright.yml` - Updates license copyright headers

### 2. Copy-Paste Templates (`workflows/`)

Organized by language/platform and project type, these are thin wrapper workflows that consuming repositories can copy. They reference the reusable workflows in `.github/workflows/` using the pattern:

```yaml
jobs:
  build:
    uses: opengood-ai/central-workflows/.github/workflows/<workflow-name>.yml@main
    with:
      # inputs
    secrets:
      # secrets
```

Directory structure:
- `workflows/bash/bats/` - Bash projects using BATS testing framework
- `workflows/gradle/lib/` - Gradle library projects
- `workflows/gradle/lib-postgres/` - Gradle library projects with PostgreSQL
- `workflows/gradle/app-postgres/` - Gradle application projects with PostgreSQL
- `workflows/gradle/app-web-mvc/` - Gradle web MVC application projects
- `workflows/gradle/cli/` - Gradle CLI projects
- `workflows/gradle/plugin/` - Gradle plugin projects
- `workflows/python/package/` - Python package projects

## Common Workflow Types

Each project type typically includes these workflow files:
- `build.yml` - Runs on all branches except main, performs build and tests
- `release.yml` - Triggered on PR merge to main with 'release' label
- `deploy-dev.yml` - Deploys to dev environment (apps only)
- `deploy-prod.yml` - Deploys to prod environment (apps only)
- `cancel.yml` - Cancels in-progress workflows
- `codeql.yml` - CodeQL security analysis
- `coverage.yml` - Code coverage reporting to Codecov
- `dependencies-submit.yml` - Submits dependency graph to GitHub
- `update-license-copyright.yml` - Updates copyright headers
- `clear-packages.yml` - Clears old GitHub packages (apps only)

## Key Patterns

### Default Tool Versions
- Java: 21 (Temurin distribution)
- Python: 3.13
- Configurable via workflow inputs

### Release Process
- Gradle releases use the `release.useAutomaticVersion=true` flag
- Release version extracted from `gradle.properties` by removing `-SNAPSHOT` suffix
- Releases require PR to be merged with 'release' label
- Gradle libraries publish to Maven Central (requires GPG signing secrets)
- Python packages publish to PyPI
- Gradle plugins publish to Gradle Plugin Portal

### Deployment Process
- Deploys triggered by PR merge with label `deploy {env}` (e.g., 'deploy dev')
- Artifacts downloaded from GitHub Maven packages before deployment
- Azure App Service deployments use publish profiles and app settings from secrets

### Code Quality
- Gradle projects use ktlint for Kotlin code style (version 1.2.1)
- Python projects use ruff for linting and formatting (target py313)
- All projects support optional Codecov integration

### PostgreSQL Projects
- Use separate admin and build database URIs
- Require PostgreSQL service credentials as secrets
- Run PostgreSQL containers during build for integration tests

## Modifying Workflows

When editing the reusable workflow definitions in `.github/workflows/`:
1. Changes affect all consuming repositories using that workflow
2. Test changes carefully as they have wide impact
3. Maintain backward compatibility with existing inputs/secrets where possible
4. Version workflow calls using `@main` or specific commit SHA/tag

When creating new project type templates in `workflows/`:
1. Copy an existing similar project type as a starting point
2. Update the `uses:` reference to point to the appropriate reusable workflow
3. Configure required inputs and secrets for that workflow type
4. Follow the naming convention: `workflows/{platform}/{project-type}/{workflow-type}.yml`
