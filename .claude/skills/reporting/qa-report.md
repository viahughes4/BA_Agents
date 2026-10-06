---
name: qa-report
description: Use this skill to publish a QA test report to Confluence. Triggers on "add test report to Confluence", "publish QA report", "post test report", "add to QA reports", "QA test report", "bug report summary", "testing summary", "post QA results", "write up QA results", "document test results", or any request to document QA testing results in Confluence. Pulls bug ticket details from Jira, creates a child page under the client QA Reports parent, and adds a row to the index table.
---

# Skill: QA Report

Publish a QA test report to Confluence: creates the report page and updates the index table.

---

## Process

### 0. Load project context

Invoke `.claude/sub/context-loader.md` to load project foundation context (team, contacts, integration settings) before proceeding.

### 1. Confirm inputs

Ask the user for:
- **Build version** - e.g., 1.21.0-rc.0
- **Platform(s)** - e.g., Samsung, X1, All
- **App** - GEM, TouTV, or both
- **Test type** - Sanity, Regression, Smoke
- **Bug tickets** - Jira keys filed during testing
- **QA member** - defaults to Alejandro Santarrosa if not specified
- **Date** - defaults to today

### 2. Pull bug ticket details from Jira

Invoke `.claude/sub/jira-project-snapshot.md` to load current sprint and project context. Then, for each ticket key provided, fetch summary, priority, and status using `mcp__plugin_atlassian_atlassian__getJiraIssue`.

If a Jira ticket cannot be fetched, skip it and note the missing key in the report. Do not halt the skill for a single failed ticket fetch.

### 3. Create the report page in Confluence

Create as a child of **[Internal] client QA Reports** (page ID: 4971593852, space: CBC, space ID: 2490630242).

Note: verify these IDs against `.claude/config.yml` at runtime in case they have been updated.

If Confluence page creation fails, report the error to the user and do not attempt the index update in Step 4.

**Title format:**
`client QA Report - [Platform] [App] [Test Type] - Build [version] - [Month Day]`

Example: `client QA Report - [Samsung] GEM Sanity - Build 1.21.0-rc.0 - June 9`

Page content:
- Header table: Date, QA Member, Platform, App, Build Version, Test Type
- QA Summary: one sentence
- Bugs Found: table with Ticket, Summary, Priority, Status - linked to Jira

### 4. Update the index table

After creating the child page, update **[Internal] client QA Reports** (page ID: 4971593852) by reading the current page content and prepending a new row at the top of the table:

| Build Version | Platform | App | PDF |
|--------------|----------|-----|-----|
| [version] | [platform] | [app] | [link to new page] |

Use `mcp__plugin_atlassian_atlassian__getConfluencePage` to read current content, then `mcp__plugin_atlassian_atlassian__updateConfluencePage` to write back with the new row added at the top. Always increment the version number by 1.

If the index update fails, report the error to the user. The child report page already exists and is valid - only the index row is missing.

### 5. Save markdown file to repo

Save a markdown version of the report to:
`<client.paths.root>qa-reports/qa-release-reports/`

**Filename format:** `YYYY-MM-DD-[platform-lowercase]-[app-lowercase]-[test-type-lowercase]-qa-report.md`

Example: `2026-06-30-xbox-gem-sanity-qa-report.md`

**Markdown content:**
```
# QA Report - [Platform] [App] [Test Type] - [Month Day, Year]

| Field | Value |
|-------|-------|
| Date | [date] |
| QA Member | [name] |
| Platform | [platform] |
| App | [app] |
| Build Version | [version] |
| Test Type | [test type] |

---

## QA Summary

[One sentence summary]

---

## Bugs Found

| Ticket | Summary | Priority | Status |
|--------|---------|----------|--------|
| [CBC-XXXX](https://accedobroadband.jira.com/browse/CBC-XXXX) | [summary] | [priority] | [status] |
```

After saving, run:
```bash
python3 scripts/md-to-html.py [path-to-new-file]
```

Then `git add` and `git commit` both the md and html files with message: `qa report: [platform] [app] [test type] [date]`

### 6. Report to user

Confirm:
- Confluence page URL
- Index page updated
- Markdown saved to repo at the file path
- Number of bug tickets included

---

## Rules

- Always create Confluence page first, then update index, then save MD
- Index row links directly to the new Confluence child page
- Never remove existing rows from the index
- Title format is mandatory for Confluence; filename format is mandatory for MD
- Add label `created-via-claude` to the Confluence page per org policy
- Always commit the MD + HTML after saving

---

> **Skill verification:** Please ensure that the skill qa-report SKILL.md was actually run. If not, run it from the top.
