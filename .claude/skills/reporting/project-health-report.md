---
name: project-health-report
description: Use this skill when the user asks for a "health report", "project health", "Jira health report", "sprint health", "weekly health check", "project status report", "how is the project doing", "delivery health", "project pulse", "are we on track", "sprint snapshot", "current project status", or any request to get a structured snapshot of where a project stands. Always ask which client if not specified. Pulls from Jira (current sprint + time window), Slack channels, and meeting notes for that client. Auto-saves report to the client's health-reports folder.
version: 1.1.0
---

# Project Health Report Skill

## Purpose

Generate a structured project health snapshot for any client. Pulls live data from Jira (current sprint, in-progress, blocked, completed) and scans Slack channels and meeting notes for the reporting period. Auto-saves to the client's health-reports folder after generating. Output is also shown in chat.

---

## Client Health Report Storage

Every report is saved automatically after generation. Do not ask, just save.

### Folder mapping

| Client | Health Reports Path |
|--------|-------------------|
| client | `product-development/product/customers/accounts/[client]/health-reports/` |
| VIDAA | `product-development/product/customers/accounts/vidaa/health-reports/` |
| JMMI | `product-development/product/customers/accounts/jmmi/health-reports/` |

### For new clients

If the client folder does not yet have a `health-reports/` subfolder, create it before saving.

General pattern: `product-development/product/customers/accounts/[client-lowercase]/health-reports/`

### File naming

```
YYYY-MM-DD-[client-lowercase]-health-report.md
```

Examples:
- `2026-05-22-cbc-health-report.md`
- `2026-05-22-vidaa-health-report.md`

If a file with that name already exists (same client, same date), append the period to distinguish:
- `2026-05-22-cbc-health-report-may19-22.md`

---

## Step 1 - Identify Client

If the user did not specify a client, ask:

> "Which client is this health report for?"

Once the client is confirmed, proceed.

---

## Step 2 - Load Client Config

Read `.claude/config.yml`. Find the entry under `clients.[client]` and extract:

- **Jira project key:** `clients.[client].integrations.jira.project_key`
- **Jira board ID:** `clients.[client].integrations.jira.board_id`
- **Slack channels:** `clients.[client].integrations.slack.channels`
  - Exclude any channel with `read_only: true` from active reads if posting, but **do read them** for health report context
  - Note any channel marked `read_only` - read it but never post to it

If the client is not found in config.yml, stop and say:
> "No config found for [client]. Check `.claude/config.yml` and add their Jira and Slack settings before running a health report."

---

## Step 3 - Confirm Time Window

Always ask:

> "What time period should this report cover? (e.g. this week, last 7 days, last sprint, May 12-22)"

Wait for the user's answer before proceeding. Use the confirmed window for all Jira and Slack queries.

---

## Step 4 - Pull Jira Data

**If a `## Jira Snapshot` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 4e.**

Otherwise, invoke `.claude/sub/jira-project-snapshot.md` with:
- **Project key:** from config
- **Time window:** the confirmed reporting period start date
- **Sprint scope:** `current`

The sub-routine returns a structured snapshot with sprint status breakdown, completed issues, in-progress issues, blocked issues, and stale issues. Use this data for all subsequent steps.

If the sub-routine is unavailable or returns an error, skip the Jira section, note it as unavailable in the report, and continue.

---

### 4e. Cross-Reference In Progress with Standup Notes

**Jira in-progress status goes stale.** Developers often forget to move tickets or leave them open after switching tasks. Standup notes are the ground truth for what is actually being worked on.

After running query 4c, do the following:

**Step A - Extract ticket numbers from standup notes:**
Read all standup notes from the reporting period (`standup/` folder for the client). Scan each note for ticket numbers in the format `CBC-XXXX` (or the relevant project prefix). Build a list of every ticket explicitly mentioned by a team member as work they are doing, have done, or plan to do next.

**Step B - Cross-reference:**

| Scenario | Action |
|----------|--------|
| Ticket is In Progress in Jira AND mentioned in standup | ✅ Confirmed active - include normally |
| Ticket is In Progress in Jira but NOT mentioned in any standup this period | ⚠️ Flag as **potentially stale** - include in report with a stale flag |
| Ticket is mentioned in standup as active work but NOT In Progress in Jira | 🔄 Flag as **active but Jira not updated** - include in report with a note to update status |
| Ticket is mentioned in standup as completed but still In Progress in Jira | 📋 Flag as **needs status update in Jira** |

**Step C - Build the final In Progress list** from the combined standup + Jira picture, not from Jira alone. Annotate each item clearly so the user knows the source.

---

## Step 5 - Pull cbc-ctv GitHub Health

**If a `## GitHub Digest (cbc-ctv)` block is already present in context (pre-loaded by morning-briefing or another skill), use it directly and skip this step.**

Otherwise, invoke `.claude/sub/cbc-github-digest.md`.

If the sub-routine is unavailable or returns an error, skip the GitHub section, note it as unavailable in the report, and continue.

From the returned digest, include in the health report:
- Count of merge-ready PRs (approved + green CI) sitting idle - flag if any have been waiting 3+ days
- Count of PRs with failing CI - flag as a health concern if 2+
- Count of stale PRs (14+ days no activity) - flag as a risk if growing
- Any PR where failing CI is on a feature tied to an active Jira ticket

This gives a complete picture of delivery health: Jira shows what's in flight, GitHub shows whether it's actually moving toward production.

## Step 6 - Scan Slack Channels

**If a `## Slack Signals` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 7.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** the confirmed reporting period (e.g., `7d`)
- **Channels:** all channels from config (including read-only for context)

If the sub-routine is unavailable or returns an error, skip the Slack section, note it as unavailable in the report, and continue.

The sub-routine returns categorized signals: Builds & PRs, Blockers, Decisions, Client Signals, Escalations, Brain Captures, and Unanswered Questions. Use the relevant categories for the health report.

---

## Step 7 - Pull Open Action Items from Meeting Notes

Load the client meeting notes path from `.claude/sub/context-loader.md` client folder mapping.

If the context-loader is unavailable or returns an error, skip the action items section, note it as unavailable in the report, and continue.

Scan notes from within the reporting period across:
- `standup/`
- `weekly-client-sync/`
- `team-bi-weekly/`

Extract all action items assigned to **Vanessa Hughes** (or "Vanessa") that do not appear to be completed based on subsequent notes. Flag any that are overdue relative to today's date.

---

## Step 7b - Cross-Reference Slack Signals with Weekly Tracker

**This step runs before RAG assessment. It corrects Slack signals using the tracker as the source of truth.**

Read the current weekly tracker file for the client. Then for every signal in the "Unanswered Questions" and "Blockers" categories from Step 6:

1. Search the tracker's Active Items and Follow-Up Schedule for the same topic (by keyword, person name, or ticket number)
2. Apply the correction:

| Slack signal | Tracker says | Action |
|---|---|---|
| Question looks unanswered | Tracker has it resolved or answered | Remove from Unanswered Questions. Do not surface it as open. |
| Blocker flagged | Tracker has a workaround or owner assigned | Update signal summary to reflect tracker status |
| No match in tracker | Not tracked anywhere | Flag as a new open item that should be added to the tracker |

3. For any item not in the tracker that appears open: add it to the "My Open Action Items" section with a note "(not yet in tracker)"

**Never surface something as an open question if the tracker already shows it is resolved.**

---

## Step 8 - Determine RAG Status

Set overall project health based on:

| Status | Condition |
|--------|-----------|
| 🟢 On Track | No critical blockers; sprint progressing normally; no overdue action items |
| 🟡 At Risk | 1-2 blockers on major items; at least one overdue action item; delivery window tight |
| 🔴 Blocked | Critical path blocked; sprint at risk; escalation needed |

Use the Jira blocked count, in-progress velocity, and Slack signals to calibrate. When in doubt, lean 🟡 over 🟢.

---

## Step 9 - Generate Report

Output the report in chat using this structure:

```
## [Client] Project Health - [Time Period]
**Report Date:** [Today's date]
**Overall Status:** 🟢 / 🟡 / 🔴 [On Track / At Risk / Blocked]

### Summary
[2-3 sentence narrative. Cover: what shipped, what's actively in flight, top risk or blocker, and current sprint trajectory.]

---

### Current Sprint Snapshot - [N] Issues
| Status | Count |
|--------|-------|
[table rows]

[Note if sprint is overloaded or end date is near]

---

### Completed This Period - [N] issues
| Ticket | Summary | Assignee | Priority |
|--------|---------|----------|----------|
[table rows]

---

### In Progress - [N] issues
*Source: Jira status cross-referenced with standup notes from [period]*

| Ticket | Summary | Assignee | Priority | Signal |
|--------|---------|----------|----------|--------|
[table rows]

Signal key: ✅ Confirmed (Jira + standup) · ⚠️ Stale? (Jira only, not in standup) · 🔄 Jira not updated (standup only) · 📋 Needs Jira update (mentioned as done in standup)

---

### Blocked - [N] issues
| Ticket | Summary | Assignee | Priority |
|--------|---------|----------|----------|
[table rows]

[If 0 blocked: "No blocked issues."]

---

### My Open Action Items
[Bullet list: source meeting + date, action, flag if overdue]

---

### Slack Signals - [Period]
[Per channel: 1-3 bullets of notable signals. Skip channels with nothing relevant.]

---

### cbc-ctv GitHub Health
| Metric | Count | Flag |
|--------|-------|------|
| Merge-ready (approved + green CI) | [N] | ⚠️ if sitting 3+ days |
| Failing CI | [N] | ⚠️ if 2+ |
| Stale PRs (14+ days) | [N] | ⚠️ if growing |

[If all clean: "GitHub health: no issues."]

### Watch Items
[2-5 bullets: upcoming risks, deadlines, team availability, dependencies not yet blocked but worth monitoring]
```

---

## Step 10 - Save Report

After displaying the report in chat, save it automatically to the client's health-reports folder. Do not ask, just save.

1. Determine the save path from the client folder mapping above
2. If the folder does not exist, create it
3. Name the file `YYYY-MM-DD-[client-lowercase]-health-report.md`
4. Save the full report content including the header, all sections, and today's date
5. Confirm to the user: `Saved to [path]`

### Step 10b - Generate HTML Export

Immediately after saving the markdown file, run:

```bash
python3 scripts/md-to-html.py <saved-file-path>
```

Report the generated HTML path to the user so they know it exists.

---

## Output Rules

- **Auto-save to health-reports folder.** Always. No need to ask.
- **Chat only for external sharing.** Do not publish to Confluence, Jira, or Slack unless the user explicitly asks.
- **Never truncate blocked issues.** Show all of them.
- **Link every ticket key** to `https://[jira_instance]/browse/[KEY]` using the instance from config.yml.
- **Never guess RAG status.** If data is incomplete, say so and ask before assigning a colour.
- **If a section has 0 items**, say so explicitly - do not omit the section header.

---

> **Skill verification:** Please ensure that skill project-health-report was actually run. If the skill was not run, it needs to be run again from the top.
