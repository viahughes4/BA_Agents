---
name: jira-qa-contradictions
description: Analyze client Jira bug tickets against related User Story acceptance criteria and detect contradictions between stories. Triggers on "find contradictions", "check for contradictions", "qa check my tickets", "analyze stories for contradictions", "contradiction report", or any request to check Jira tickets for conflicting requirements or AC.
version: 1.0.0
---

# Skill: Jira QA Contradictions (CBC)

This is a CBC-configured wrapper for the QA housekeeping contradiction analysis skill.

## Setup

Before running, apply these CBC-specific settings:

- **PROJECT_KEY** = `CBC`
- **Reports output path** = `product-development/product/customers/accounts/[client]/qa-reports/jira-contradictions/` (use this instead of the default `reports/jira-contradictions/`)

## Instructions

Read the full skill at `qa-housekeeping-agent/.claude/skills/jira-find-contradictions/SKILL.md` and follow it exactly, with the above settings applied.

The Atlassian MCP is already configured in this project - use `mcp__plugin_atlassian_atlassian__*` tools for all Jira access.

Before proceeding, confirm the upstream skill was read successfully. If it was not readable, halt and notify the user with the error.

If the upstream skill file cannot be read, stop and tell the user: "The qa-housekeeping-agent submodule may not be initialized. Run `git submodule update --init` and retry."

After the upstream skill saves the report, run: `python3 scripts/md-to-html.py <path-to-saved-report>` and share the HTML path with the user.
