---
name: release-notes
description: Use this skill to create release notes for a client CTV release. Triggers on "create release notes", "write release notes", "release notes for version", "release note", or any request to document a release. Pulls ticket details from Jira, generates a structured release note in the standard client format, saves it locally, and publishes to Confluence under the client CTV Release Notes parent page.
---

# Skill: Release Notes

Create a structured release note for a client CTV version, save it locally, and publish to Confluence.

---

## Process

### 0. Load project context

Invoke `.claude/sub/context-loader.md` to load the project foundation. This provides: team roster, platform list, Confluence cloudId, spaceId, parentId, and other project config values. Use these loaded values throughout the skill instead of hardcoded IDs.

### 1. Confirm inputs

Ask the user for:
- **Version number** - e.g., 1.19.3
- **Platform(s)** - e.g., X1, Samsung, All, LG, Xbox, FireTV, SSTV
- **Ticket(s)** - Jira ticket keys to include (e.g., <jira.project_key>-5264, or "all tickets in this sprint")
- **Date** - defaults to today if not specified

### 2. Pull all tickets from Jira for this version

Invoke `.claude/sub/jira-project-snapshot.md` with the version and project parameters (project: CBC, version: [user-provided version]) to retrieve the consolidated ticket list for this release. Use the snapshot output as the basis for all ticket data in subsequent steps.

**Step 2a: Find all matching release versions**

Using the snapshot output, identify all tickets across every release candidate for the version. For example, if the version is `1.20.0`, the snapshot should cover tickets fixed in any version matching that base:

```
project = <jira.project_key> AND fixVersion IN ("1.20.0", "1.20.0-RC.0", "1.20.0-RC.1", "1.20.0-rc.0", "1.20.0-rc.1", "1.20.0-rc.2", "1.20.0-rc.3", "1.20.0-rc.4", "1.20.0-rc.5") ORDER BY issuetype ASC
```

Try broad first - include all likely RC suffixes. Only versions that exist in Jira will return results.

If the snapshot returns 0 tickets for all RC variants, surface the following message and stop:
> No tickets found for version [version] or any RC variant. Please verify the version string and check that fixVersion is set correctly in Jira before continuing.

**Step 2b: Add any tickets the user explicitly provided**

If the user gave specific ticket keys, fetch those too with `mcp__plugin_atlassian_atlassian__getJiraIssue` and merge into the full list. Deduplicate.

**Step 2c: Filter and fetch full details**

For each ticket found, fetch:
- Summary
- Issue type (Bug, Feature, Improvement, Task, Story)
- Assignee
- Fix Version(s)

Exclude: Epic-level tickets, tickets with status "Won't Do" or "Duplicate", and any ticket whose summary contains "DO NOT MERGE".

**Step 2d: Confirm with the user before generating**

Show the user a summary of what was found:
> Found [N] tickets across versions: [list of RC versions]. Here is the breakdown:
> - New Features: [N]
> - Improvements: [N]
> - Bug Fixes: [N]
> - [list of ticket keys + summaries]
>
> Any tickets to add or remove before I generate the release note?

Wait for confirmation or adjustments before proceeding.

Group final ticket list by type:
- New Features
- Improvements to existing features
- Bug Fixes

### 3. Generate the release note

Use the structure below exactly. Never deviate from this structure.

### 4. Save locally

Save to:
`product-development/product/customers/accounts/[client]/release-notes/[version]-release-notes.md`

Run `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/release-notes/[version]-release-notes.md`

If the HTML export script fails, note the error and continue - report the failure in the final summary.

### 5. Publish to Confluence

Publish as a child page under: **client CTV Release Notes** (use cloudId, spaceId, and parentId loaded from context-loader in Step 0)

**Title format - always use this exact pattern:**
`[Platform] Release Notes - client - [version] - [Month Day]`

Examples:
- `[X1] Release Notes - client - 1.19.3 - June 9`
- `[Samsung] Release Notes - client - 1.20.0 - June 10`
- `[All] Release Notes - client - 1.20.1 - June 15`
- `[X1][LG][FireTV] Release Notes - client - 1.20.0 - June 10`

Use `mcp__plugin_atlassian_atlassian__createConfluencePage` with the cloudId, spaceId, and parentId values loaded from context-loader (do not hardcode these IDs inline).

If the Confluence publish fails, save the release note locally and report:
> Confluence publish failed: [error]. Release note saved locally at [local file path]. Please publish manually or retry.

Add the label `created-via-claude` per org policy.

### 6. Report to user

Confirm:
- Local file path
- HTML export path
- Confluence page URL
- Number of tickets included
- Platform(s) covered

---

## Structure - always use this exact format

```
[Header table]
- Release: [Jira link(s)]
- Date: [date]
- Version: [version]
- Description: [one-line summary]
- Contributors: [assignees from tickets]
- QA Report: [platform QA results]
- QA Sign Off: [GO / PENDING / NOT READY per platform]

# Summary
[2-3 sentences describing what this release contains and why it was released]

# New Features
[List of new features, grouped by feature area. If none: "None."]

# Improvements to existing features
[List of improvements. If none: "None."]

# Bug Fixes
[List of bug fixes, grouped by area - e.g., Long Press, CMP, General. If none: "None."]
```

---

## Rules

- Title format is mandatory: `[Platform] Release Notes - client - [version] - [Month Day]`
- Structure must always match the template above - same sections, same order
- Never skip a section - if empty, write "None."
- Use ticket keys in bug fix entries - e.g., **<jira.project_key>-5264** - description
- Contributors are the assignees of included tickets
- Group bug fixes by feature area where possible
- QA Sign Off uses: GO (green), PENDING (yellow), NOT READY (red)
- Always publish to Confluence under the client CTV Release Notes parent page
- Add label `created-via-claude` to the Confluence page

---

> **Skill verification:** Please ensure that the skill release-notes SKILL.md was actually run. If not, run it again from the top.
