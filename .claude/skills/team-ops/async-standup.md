---
name: async-standup
description: Use this skill to write and send a standup update to the team Slack channel. Triggers on "write my standup", "send my standup", "post my standup", "draft standup", "write my update for the team", "I need to send my standup", "async standup", or any request to compose a standup message ready to post. Pulls from today's tracker, standup notes, and Slack signals. Posts to #prj-cbc-int or shows the draft for review first.
---

# Skill: Async Standup

Composes a standup update in Yesterday / Today / Blockers format from today's tracker, standup notes, and Slack signals. Shows the draft for PM review, then either posts to Slack or saves for you to post.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, team roster, and folder paths.

---

## Process

### 1. Gather standup data

**If standup-prep data is already present in context (pre-loaded by batch-runner or run earlier in this session), use it directly.**

Otherwise, gather the data that feeds standup-prep:

**1a. Load the most recent standup notes** from `product-development/product/meetings/[Client]/meeting-notes/standup/`. Extract per-person work items, completed items, and blockers. If no standup notes are found for today, note that standup notes were unavailable and proceed with tracker data only.

**1b. Scan Slack for overnight signals**

**If a `## Slack Signals` block is already present in context, use it directly.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** `1d`
- **Channels:** all channels from config

If the Slack signal scan returns no results or is unavailable, note that no Slack signals were found and continue with available data.

**1c. Scan Gmail for overnight emails**

**If a `## Gmail Digest` block is already present in context, use it directly.**

Otherwise, invoke `.claude/sub/gmail-cbc-digest.md` with:
- **Time window:** `1d`
- **Confluence lookback:** `1d`
- **Include unread only:** `false`

If the Gmail digest is unavailable or returns no results, note that Gmail data was unavailable and continue with available data.

**1d. Load the weekly tracker** from `product-development/product/customers/accounts/[client]/weekly-trackers/` for today's priorities.

### 2. Compose the standup message

Build a formatted standup update from the PM's perspective:

```
**Yesterday:**
- [What the PM accomplished yesterday - from tracker completed items, meeting notes, Slack activity]
- [Specific deliverables, meetings attended, decisions made]

**Today:**
- [What the PM plans to work on today - from tracker active items, follow-ups due, Week Ahead]
- [Meetings scheduled, deliverables due]

**Blockers:**
- [Anything preventing progress - from tracker blocked items, unanswered emails, waiting-on-client items]
- [Or: "None"]

**Notes:**
- [Any relevant context - team member availability, upcoming deadlines, client signals]
```

### 3. Present for PM review

Show the composed standup and ask:

> **Here's your standup draft. Want me to post it to #prj-cbc-int, or copy it for you to post yourself?**

Wait for the PM to approve or request edits.

### 4. Post or copy

If the PM says post it: send to Slack channel #prj-cbc-int using the Slack MCP (channel ID is loaded from config via context-loader.md).
If the PM wants to post themselves: just show the formatted text cleanly so they can copy it.
Never send an email. Never create a Gmail draft.

### 5. Report to user

Confirm:
- Posted to Slack or ready to copy
- Summary of what was included (X yesterday items, Y today items, Z blockers)

---

## Rules

- **One interactive gate.** Step 3 is the only confirmation point. Data gathering is autonomous.
- **PM perspective only.** This is Vanessa's standup update - do not include per-developer updates. Those go in the standup-prep file.
- **Yesterday items must be real.** Pull from tracker completed items, meeting notes, and Slack activity. Do not fabricate accomplishments.
- **Today items must be actionable.** Pull from tracker active items, follow-ups due today, and Week Ahead. Each item should be specific enough to verify at EOD.
- **Blockers are honest.** If the PM is waiting on the client, say so. If nothing is blocked, say "None."
- **Slack only.** Post to #prj-cbc-int or show copy-ready text. Never create a Gmail draft. Never send email.
- **Keep it scannable.** Standup messages should be readable in 30 seconds. 2-4 bullets per section max.

---

> **Skill verification:** Please ensure that the skill async-standup SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
