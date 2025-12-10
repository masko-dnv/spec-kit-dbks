# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spec Kit is an open source toolkit for Spec-Driven Development (SDD) - a methodology where specifications become executable, directly generating working implementations. The project consists of:

- **Specify CLI** (`src/specify_cli/`): Python CLI tool that bootstraps projects with SDD templates
- **Templates** (`templates/`): Markdown templates and slash commands for AI coding agents
- **Scripts** (`scripts/`): Bash and PowerShell helper scripts for project setup
- **Documentation** (`docs/`): DocFX-powered documentation site

## Development Commands

### Running the CLI locally

```bash
# Install in development mode
uv pip install -e .

# Run directly
python -m specify_cli init <project-name>

# Or use uvx for one-off runs
uvx --from . specify init <project-name>
```

### Linting

```bash
# Markdown linting (CI uses markdownlint-cli2-action)
npx markdownlint-cli2 "**/*.md"
```

### Building Documentation

```bash
cd docs
dotnet tool install -g docfx
docfx docfx.json --serve
# View at http://localhost:8080
```

## Architecture

### CLI Structure

The entire CLI is in a single module `src/specify_cli/__init__.py`. Key components:

- **AGENT_CONFIG dict**: Defines supported AI agents with their folder paths, install URLs, and CLI requirements
- **download_template_from_github()**: Fetches release assets from GitHub API
- **download_and_extract_template()**: Downloads and extracts agent-specific template ZIPs
- **StepTracker class**: Renders hierarchical progress tree during initialization
- **select_with_arrows()**: Cross-platform interactive menu using readchar

### Template System

Templates are packaged as ZIP files per release (one per AI agent + script type combination). The release workflow packages templates from `templates/` directory with agent-specific configurations.

Key template files:
- `commands/*.md`: Slash command definitions for `/speckit.*` commands
- `*-template.md`: Templates for specs, plans, tasks, and checklists

### Script Variants

Two script variants exist for cross-platform support:
- `scripts/bash/`: POSIX shell scripts (sh)
- `scripts/powershell/`: PowerShell scripts (ps)

## Key Conventions

- Agent configuration folders follow pattern: `.{agent-name}/` (e.g., `.claude/`, `.gemini/`)
- CLI checks for Claude in `~/.claude/local/claude` first (post-migrate-installer location)
- GitHub token can be provided via `--github-token`, `GH_TOKEN`, or `GITHUB_TOKEN` env vars
- Templates are versioned via GitHub releases with pattern `spec-kit-template-{agent}-{script}.zip`