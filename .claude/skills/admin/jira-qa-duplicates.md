---
name: jira-qa-duplicates
description: Find duplicate Jira bug tickets and User Story tickets in the client's Jira project. Triggers on "find duplicates", "check for duplicate tickets", "duplicate stories", "duplicate bugs", "qa duplicates", or any request to detect repeated or overlapping Jira tickets in the client's Jira project.
version: 1.0.0
---

# Skill: Jira QA Duplicates (CBC)

This is a CBC-configured wrapper for the QA housekeeping duplicate detection skill.

## Setup

Before running, apply these CBC-specific settings:

- **PROJECT_KEY** = `CBC`
- **Reports output path** = `product-development/product/customers/accounts/[client]/qa-reports/jira-duplicates/` (use this instead of the default `reports/jira-duplicates/`)

## Instructions

Read the full skill at `qa-housekeeping-agent/.claude/skills/jira-find-duplicates/SKILL.md` and follow it exactly, with the above settings applied.

If the file at `qa-housekeeping-agent/.claude/skills/jira-find-duplicates/SKILL.md` cannot be read, stop and notify the user: "The underlying jira-find-duplicates skill could not be loaded. Check that the qa-housekeeping-agent submodule is present and try again."

The Atlassian MCP is already configured in this project - use `mcp__plugin_atlassian_atlassian__*` tools for all Jira access.

Before proceeding, confirm that the full skill at the path above was successfully read. If it was not read or returned an error, stop here and notify the user rather than continuing with incomplete instructions.

Before writing the report, run `mkdir -p product-development/product/customers/accounts/[client]/qa-reports/jira-duplicates/` to ensure the output folder exists.

After the report is saved, run `python3 scripts/md-to-html.py <report-path>` to generate the HTML export. The HTML file will appear at `product-development/product/customers/accounts/[client]/qa-reports/jira-duplicates/html/YYYY-MM-DD-[Client].html`.
