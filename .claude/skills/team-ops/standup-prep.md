---
name: standup-prep
description: Use this skill to prepare for a daily team standup. Triggers on "prep standup", "standup prep", "prepare standup", "set up today's standup", "get standup ready", "standup notes for today", or any request to get the team standup ready before the call. Pulls live project data per team member, surfaces blockers and changes, and produces a pre-populated standup notes template saved to the correct folder.
---

# Skill: Standup Prep

Prepare for today's team standup. Pulls live project state, surfaces relevant team updates, and generates a pre-populated notes file ready to update during the call.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths.

### 2. Confirm who this standup is for
If the user has not specified, ask:

> **Which project or client is this standup for?**

Use the answer to identify the correct Jira project key, team roster, and meeting notes folder.

### 3. Confirm the date
Use `currentDate` from context. This is the standup date.

### 4. Check for an existing file
Check the standup folder for a file dated today. If one already exists, tell the user and ask whether to overwrite or append.

### 5. Load previous standup notes

Read the most recent standup file for this project from the standup folder. This is the sole source of truth for per-person work items - do not query Jira for in-progress tickets; Jira statuses lag behind actual work and produce noise.

If no previous standup file exists, note this explicitly in each team member section: "No previous standup on file - check in directly." and skip the carryover action items section.

For each team member, extract:
- What they reported working on or finishing
- Any tickets they mentioned by key or name
- Any "next" items they committed to
- Any blockers or open questions they raised

Also extract for the full team:
- Action items that were assigned and not yet confirmed complete
- Any blockers that were noted and may still be active
- Any team member absences noted

### 6. Fetch cbc-ctv GitHub activity

**If a `## GitHub Digest (cbc-ctv)` block is already present in context (pre-loaded by batch-runner or morning-briefing), use it directly and skip this step.**

Otherwise, invoke `.claude/sub/cbc-github-digest.md`.

If the GitHub digest is unavailable or returns no results, note in the GitHub section: "GitHub digest unavailable - check manually." and continue.

From the returned digest, surface for standup:
- Any PRs merged since yesterday (check git log or recently-updated approved+merged PRs)
- Any new PRs opened by the team since yesterday
- Any PRs that newly have failing CI
- Any merge-ready PRs (approved + green) that have been sitting for 2+ days

This is the engineering team's work - standup is the right place to call out "PR X has been approved and green for 3 days, what's blocking the merge?"

### 6b. Scan the project Slack channel for overnight signals

**If a `## Slack Signals` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 6c.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** `1d`
- **Channels:** all channels from config

From the returned signals, focus on: Builds & PRs, Blockers, Decisions, Brain Captures - the categories most relevant for standup prep.

### 6c. Scan Gmail for overnight client emails

**If a `## Gmail Digest` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 7.**

Otherwise, invoke `.claude/sub/gmail-cbc-digest.md` with:
- **Time window:** `1d`
- **Confluence lookback:** `1d`
- **Include unread only:** `false`

From the returned digest, focus on: client emails needing response and Confluence changes that may affect today's work. Flag any unacknowledged client asks as needing attention at standup.

### 7. Filter for standup-relevant content

Standup surfaces: **general team updates, what was done, what's being worked on today, resourcing changes (time off, new joiners, handoffs), plan or ticket changes, and blockers.**

From the data collected, identify:
- **Completed since last standup** - what shipped or merged yesterday
- **In progress today** - what each developer is actively on
- **Blockers** - anything preventing forward progress
- **Plan or ticket changes** - scope changes, tickets added or removed, priority shifts noted in Jira or Slack
- **Resourcing updates** - anyone out, new team members, handoffs needed
- **Carryover action items** - items from the previous standup that haven't been addressed

Do NOT include in standup prep:
- Roadmap risks (save for PO sync or status meeting)
- Deep client dependency discussions
- Sprint-level delivery risk analysis

### 8. Build the per-person sections

For each developer on the team roster:
- List their current in-progress tickets (key + one-line summary)
- Flag if they have a blocked ticket
- Note anything they completed yesterday
- Note any carried-over action items from the previous standup

For the PM (Vanessa):
- List open PM action items from the previous standup
- Flag anything that is overdue or has a team dependency

### 9. Generate and save the prep file

Use the template below. Pre-populate each section. Leave blank lines where Vanessa will fill in during the call.

**IMPORTANT - Save location:**
Save to the standup PREP folder, NOT the standup notes folder.
Standup notes folder (`meeting-notes/standup/`) is **read-only** - only real meeting notes pulled from Google Drive go there. Never save prep files there.

Save prep files to:
`product-development/product/meetings/[Client]/standup-prep/YYYY-MM-DD-[client]-standup-prep.md`

Example: `product-development/product/meetings/[Client]/standup-prep/YYYY-MM-DD-[client]-standup-prep.md`

After saving, run:
```
python3 scripts/md-to-html.py product-development/product/meetings/[Client]/standup-prep/YYYY-MM-DD-[client]-standup-prep.md
```

### 10. Report to user
Confirm:
- File path where it was saved
- HTML export path
- Number of in-progress tickets surfaced
- Number of blocked tickets
- Any notable Slack signals
- Any overdue action items carried from previous standup

---

## Output Template

```markdown
# [Project / Client] - Daily Stand-up
**Date:** [Month DD, YYYY]
**Source:** To be updated after call

---

## Attendees

[Leave blank - fill in during call]

---

## Summary

[Leave blank - fill in after call]

---

## Team Updates

**[Developer 1]**
- [Ticket key: summary - in progress]
- [Ticket key: summary - completed yesterday, if any]
- Update:

**[Developer 2]**
- [Pre-populated from previous standup notes]
- Update:

**[Developer N]**
- [Pre-populated from previous standup notes]
- Update:

**[PM Name]**
- [Open action items from previous standup]
- Update:

---

## Blocked - [N] issues
[Key | Summary | Assignee | Blocker reason if known]

---

## GitHub (cbc-ctv)
[Pre-populated from cbc-github-digest - PRs merged yesterday, new PRs opened, merge-ready sitting idle, failing CI]

## Plan / Ticket Changes Since Last Standup
[Pre-populated from Jira activity + Slack signals - scope changes, tickets added/removed/reprioritized]

---

## Resourcing
[Pre-populated from previous standup notes - absences, new joiners, handoffs]

---

## Decisions

[Leave blank - fill in during call]

---

## Action Items

| Owner | Action |
|-------|--------|
| | |

---

## Notes

[Leave blank - fill in during call]

---

## Prep Notes *(remove before sharing)*
> Generated [timestamp] · Sources: Previous standup [date], Slack #prj-cbc-int, Gmail
> Slack signals: [summary or "none"]
> Email signals: [summary or "none"]
> Confluence changes overnight: [summary or "none"]
> Overdue action items carried from [previous date]: [list or "none"]
```

---

## Rules

- Never fabricate work items - use only what's in the previous standup notes or Slack
- If a team member had no updates in the previous standup, note it explicitly: "No updates from previous standup - check in"
- Blocked tickets always appear in the Blocked section AND in the relevant person's section
- Prep Notes are for Vanessa's reference only - remind her to remove before sharing
- Resourcing changes (vacation, new joiner) always get their own line - they affect the whole team's day

---

> **Skill verification:** Please ensure that the skill standup-prep SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
