# Test Strategy Adviser

This repository contains a GitHub Actions workflow that analyzes Jira tickets and recommends the correct test automation layer. It uses Claude Code via the `anthropics/claude-code-action` to examine ticket details, apply a structured decision framework, and output a structured recommendation.

To set up the workflow, add an API key for Anthropic and any Jira credentials as repository secrets. See `.github/workflows/automation-layer-adviser.yml` for full configuration details.
