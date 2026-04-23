# GitHub Workflows and Actions

This directory contains GitHub Actions workflows, agent prompts, and related configuration.

## Workflows

### Check Workflow

The `check.yml` workflow runs on pull requests, pushes, and a weekly schedule to
ensure code quality and correctness.

Jobs:

- **actionlint**: Validates GitHub Actions workflow files
- **link-checker**: Checks for broken links in Markdown files using Lychee
- **pre-commit**: Runs pre-commit hooks for code formatting and linting

#### Link Checker

The link checker job uses [Lychee](https://github.com/lycheeverse/lychee) to
scan all Markdown files for broken links. It includes caching to avoid rate
limits and can be configured via `.lycheeignore` at the repository root to
exclude specific URLs or patterns.

**Local Testing**: You can test links locally with the configured
`markdown-link-check` pre-commit hook:

```bash
# Install from requirements.txt
pip install -r .devcontainer/requirements.txt

# Check a single file
pre-commit run markdown-link-check --files path/to/file.md

# Check all Markdown files
pre-commit run markdown-link-check -a
```

The hook uses `.markdown-link-check.json` and checks both local file references
and remote URLs before you push changes.

#### Using Check as a Reusable Workflow

You can use the Check workflow in your repository by referencing it via
`workflow_call`. Note that the `master` branch is required as it is the default
branch for the `EA31337/.github` repository:

```yaml
---
name: Check
on:
  pull_request:
  push:
  schedule:
    - cron: 0 0 * * 1  # Run every Monday at 00:00 UTC
  workflow_dispatch:
jobs:
  check:
    uses: EA31337/.github/.github/workflows/check.yml@master
    with:
      submodules: 'false'  # Set to 'true' or 'recursive' if repository uses submodules
```

### Cogni AI Agent Workflow

The `cogni-ai-agent.yml` workflow provides an integration with the Cogni AI Agent for autonomous issue
resolution and PR review.

It triggers on issue and pull request comments, as well as `workflow_dispatch`. It utilizes the
`Cogni-AI-OU/cogni-ai-agent-action` and allows specifying various AI models to fulfill requests.

*Note: Requires `OPENCODE_API_KEY` secret to be set in repository settings.*

### Copilot Setup Steps Workflow

The `copilot-setup-steps.yml` workflow automates the installation and caching of Python dependencies
required for the development environment and GitHub Copilot agents.

It reads from `.devcontainer/requirements.txt`, caching the Python user site (`~/.local`) to speed up
subsequent runs, and ensures local binaries are added to the GitHub Actions `PATH`.

### Development Containers (CI) Workflow

The `devcontainer-ci.yml` workflow automates the building and testing of Development Containers.

It checks out the repository, logs into the GitHub Container Registry (GHCR), and uses the
`devcontainers/ci` action to build the container image, checking against an existing cache.
Additionally, it tests the container by verifying that required command-line tools and Python
packages are correctly installed.

#### Using Devcontainer CI as a Reusable Workflow

You can use the Devcontainer CI workflow in your repository by referencing it via `workflow_call`.
It accepts inputs for `required_commands` and `required_python_packages` to customize the validation step.

```yaml
jobs:
  devcontainer:
    uses: EA31337/.github/.github/workflows/devcontainer-ci.yml@master
    permissions:
      contents: read
      packages: write  # Required for pushing to GitHub Container Registry
```

## Problem Matchers

GitHub Actions problem matchers automatically annotate files with errors and
warnings in pull requests, making it easier to identify and fix issues.

### Available Matchers

- **actionlint-matcher.json**: Captures errors from actionlint workflow linting
- **pre-commit-matcher.json**: Captures errors from pre-commit hooks

### Pre-commit Problem Matcher

The pre-commit problem matcher supports two output formats:

1. **Generic format** (`file:line:col: message`): Used by flake8, actionlint,
   and other tools that provide column information
2. **No-column format** (`file:line message`): Used by markdownlint and other
   tools that only provide line numbers

Note: Some hooks like yamllint and ansible-lint already output GitHub Actions
annotations directly and don't need the problem matcher.

### Configuration

Problem matchers are registered in the `.github/workflows/check.yml` workflow
before running the corresponding tools.

### Using Matchers in Reusable Workflows

When using the `check.yml` workflow as a reusable workflow (via `workflow_call`),
the matcher files are automatically provided from this repository. You don't need
to copy the matcher files to your repository.

If you want to use custom matcher files, you can specify them using the inputs:

```yaml
jobs:
  check:
    uses: EA31337/.github/.github/workflows/check.yml@master
    with:
      actionlint-matcher-path: .github/custom-actionlint-matcher.json
      pre-commit-matcher-path: .github/custom-pre-commit-matcher.json
```

If these inputs are not provided, the workflow will automatically use the default
matcher files from this repository.
