---
name: confluence-email-digest
description: Use this skill to get a digest of recent client Confluence changes from email notifications. Triggers on "Confluence changes", "what changed in Confluence", "client Confluence digest", "Confluence updates", "daily digest", "action items for today", "what's on my plate today", or any request to see what the client team has changed in Confluence recently. This skill runs automatically during session start and as part of the morning daily digest.
---

# Skill: Confluence Email Digest

Pulls Confluence notification emails from Gmail, extracts all the client's Jira project changes, and presents a daily-grouped digest of what the client team has updated.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, client contacts, and folder paths.

---

## Process

### 1. Invoke the Gmail client digest sub-routine

**If a `## Gmail Digest` block is already present in context (pre-loaded by batch-runner or another skill), use the Confluence Changes section directly and skip to Step 2.**

Otherwise, invoke `.claude/sub/gmail-cbc-digest.md` with:
- **Time window:** `5d` (for client emails, though this skill focuses on Confluence)
- **Confluence lookback:** `5d` (always 5 business days)
- **Include unread only:** `false`

The sub-routine returns a `### Confluence Changes` section with daily-grouped changes, highlights, referenced tickets, and items needing review. Use this data for all subsequent steps.

If the sub-routine returns no Confluence changes or the Gmail connection is unavailable, output a brief inline note: "Confluence digest unavailable - check Gmail MCP connection and retry." Do not proceed to Steps 2-4.

### 2. Enrich with Jira ticket status (if connected)

If Jira is connected and the Gmail digest returned referenced client tickets, invoke `.claude/sub/jira-project-snapshot.md` passing the specific ticket keys as a filter to pull current status and summary for each referenced ticket.

If jira-project-snapshot does not support per-key enrichment, use a direct Jira tool call to fetch each ticket by key rather than embedding a JQL template inline.

If Jira is not connected or returns no results for the referenced tickets, skip this step and note in the output: "Jira enrichment unavailable - ticket statuses not shown."

This reveals which tickets are being actively discussed or spec'd on the client side.

### 3. Identify patterns and highlights

Using the data from the sub-routine, add a brief analysis section:

- **Most active pages** - which pages are getting the most edits (signals active work or review cycles)
- **New pages created** - entirely new documentation or specs added
- **Active contributors** - which client team members are making changes (useful for knowing who is engaged)
- **Pages relevant to current sprint** - cross-reference page titles against known active tickets or features if possible
- **Tickets being discussed** - client tickets referenced in Confluence changes

### 4. Surface actionable items

Flag any changes that may require PM attention:
- New specs or requirements pages the team hasn't reviewed
- Changes to pages that relate to in-progress work
- Comments that may contain questions or decisions needing response
- Changes to release notes, test plans, or certification docs

Mark these as:
> **Needs review:** [Page title] - [why it matters]

---

## Output Format

```markdown
# client Confluence Changes: [Date Range]

> [X] changes across [X] pages by [X] contributors

## [Most Recent Day]

| Page | Change | By | Summary |
|------|--------|----|---------|
| ... | ... | ... | ... |

## [Previous Day]
...

---

## Highlights

- **Most active pages:** [list]
- **New pages:** [list or "none"]
- **Active contributors:** [names]

## client Tickets Referenced

| Ticket | Summary | Status | Referenced In |
|--------|---------|--------|---------------|
| CBC-XXXX | [summary from Jira if available] | [status] | [Page title where it was mentioned] |

[If no tickets found: "No client tickets referenced in Confluence changes."]

## Needs Review

- [Page title] - [reason for PM attention]
- [Page title] - [reason]

[If no items need review: "No items requiring immediate PM attention."]
```

---

## When This Skill Runs

- **Session start** - runs automatically as step 10 of the auto-sync routine
- **Daily digest** - runs when the user asks for "daily digest", "action items for today", or "what's on my plate"
- **On demand** - runs when the user asks about Confluence changes directly

When running as part of session start or daily digest, output the digest inline in the chat. Do not save to a file unless the user asks.

If the user requests the digest be saved to a file, save it under `<client.paths.root>` with the naming format `YYYY-MM-DD-confluence-digest.md`, then run:
```
python3 scripts/md-to-html.py <path-to-saved-file>
```

---

## Quality Rules

- **Extract from emails only** - do not call Confluence directly. The email notifications are the source of truth for this skill.
- **5 business days** - always go back 5 business days. Do not shorten the window.
- **Do not fabricate changes.** If an email is unclear about what changed, note it as "details not available in notification."
- **Group by calendar day**, not by email arrival time.
- **De-duplicate.** If the same page edit appears in multiple notification emails, show it once.
- **Keep it scannable.** The digest should be readable in 30 seconds. Save detail for the "Needs Review" section.

---

> **Skill verification:** Please ensure that the skill confluence-email-digest SKILL.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
