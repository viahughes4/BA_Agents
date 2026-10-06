---
name: end-of-day
description: Use this skill to wrap up the PM's day. Triggers on "end of day", "eod", "wrap up", "close out my day", "carry forward", "what's left today", or any request to close out the current workday. Loads the weekly tracker, asks what's done vs. carry forward, flags stale items, scans Slack for unanswered messages, builds tomorrow's priorities, and updates the tracker.
---

# Skill: End of Day

Wraps up the PM's day by reviewing what was completed, carrying forward incomplete items, flagging stale work, surfacing unanswered messages, and building tomorrow's priority list. Updates the weekly tracker and generates HTML.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, team roster, and folder paths.

---

## Process

### 1. Load the current weekly tracker

Navigate to `<client.paths.root>weekly-trackers/` and find the tracker for the current week. If none exists, tell the user and offer to create one.

If the tracker file cannot be read or is malformed, surface the error to the PM and halt rather than proceeding with incomplete data.

Read the full tracker. Extract:
- All items in the **Daily To-Do** section for today
- All **Active Items** and their current status
- The **Follow-Up Schedule** - any follow-ups due today or overdue
- The **Week Ahead** section

### 2. Load all notes from today

**This step is mandatory - do not skip it.** Load every note file dated today before presenting the checklist. The tracker alone does not capture what actually happened during the day.

Search all of the following locations for files whose name starts with today's date (`YYYY-MM-DD`):

| Location | What it contains |
|----------|-----------------|
| `<client.paths.meetings_root>standup-prep/` | Standup prep compiled this morning |
| `<client.paths.meeting_notes>standup/` | Real standup notes from the call |
| `<client.paths.meeting_notes>weekly-client-sync/` | client weekly sync notes |
| `<client.paths.meeting_notes>team-bi-weekly/` | Team planning notes |
| `<client.paths.meeting_notes>team-meetings/` | Any other team meeting notes |
| `<client.paths.meeting_notes>po-sync-prep/` | PO sync prep if run today |

Use `find product-development/product/meetings/client -name "$(date +%Y-%m-%d)*.md" 2>/dev/null` to locate all matching files in one pass.

For each file found, read it and extract:
- **Completed work:** anything confirmed done today (per person)
- **New blockers:** anything that surfaced as blocked today
- **Decisions made:** any direction agreed in meetings
- **Action items for Vanessa:** anything assigned to PM today
- **New risks:** anything flagged as a risk or concern

Compile all extracted data into a **Daily Notes Summary** block that feeds into Step 3's checklist. Label each item with its source file so Vanessa knows where it came from.

If no files are found for today, note "No meeting notes found for today" and continue with tracker data only. Do NOT stop or ask for input at this step.

### 3. Present today's checklist for review

Show the PM their Daily To-Do items for today in a checklist format. If standup notes were loaded in Step 2, include a "From today's standup" section showing any completed work or new blockers surfaced there.

```
## End of Day Review - [Day, Month DD]

### Today's Items
- [ ] Item 1
- [ ] Item 2
- [x] Item 3 (already marked done)
...

### From Today's Standup Notes
- [Per-person work completed - from standup notes if loaded]
- [Blockers surfaced - from standup notes if loaded]

### Follow-ups Due Today
- [Follow-up 1]: [status]
- [Follow-up 2]: [status]
```

Ask the PM:

> **Which items are done? Which carry forward? Anything to add for tomorrow?**
>
> You can reply with numbers (e.g., "1, 3 done; 2, 4 carry forward") or describe in natural language.

### 4. Process the PM's response

Parse the response and categorize each item:
- **Done** → mark as `[x]` in today's checklist, move to Completed Items if it's an Active Item
- **Carry forward** → keep as `[ ]`, add to tomorrow's checklist with a carry-forward marker
- **New items for tomorrow** → add to tomorrow's checklist

### 5. Flag stale carry-forward items

Check all carry-forward items. If any item has been carried forward for **3 or more days**:
- Flag it prominently: `[CARRIED 3+ DAYS]`
- Ask the PM: "This has been carried for [N] days. Should I keep it, deprioritize it, or escalate it?"

### 6. Scan Slack for unanswered messages

**If a `## Slack Signals` block is already present in context (pre-loaded by batch-runner), use it directly.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** `1d`
- **Channels:** all channels from config

If the Slack scan is unavailable or returns an error, skip and note that Slack signals could not be retrieved. Continue with remaining data.

From the returned signals, identify:
- **Unanswered Questions:** messages directed at Vanessa or the team that haven't been responded to
- **Client Signals:** any client messages from today that may need attention tomorrow
- **Brain Captures:** any brain emoji captures from today

Surface these as a "Needs Attention Tomorrow" list.

### 7. Build tomorrow's priorities

Compile tomorrow's priority list from:
1. **Carried-forward items:** sorted by how many days carried (oldest first)
2. **Follow-ups due tomorrow:** from the Follow-Up Schedule
3. **Week Ahead items for tomorrow:** from the Week Ahead section
4. **Unanswered messages:** from Step 6
5. **Action items from standup notes:** any new PMs actions surfaced in today's standup (from Step 2)
6. **New items:** anything the PM added in Step 3

Present as a numbered priority list, top 3 highlighted.

### 8. Update the tracker

Update the weekly tracker with ALL of the following:

**From today's review:**
- Today's checklist items marked `[x]` done or `[ ]` carry-forward
- Active Items updated (completed items moved to Completed, statuses changed)
- Follow-Up Schedule updated (done items removed, new follow-ups added)
- Week Ahead refreshed (remove today's items, add any new ones)

**Tomorrow's Daily To-Do section - build and write automatically:**
Create or update tomorrow's Daily To-Do with every open item from:
1. Carried-forward items from today (oldest first)
2. Follow-ups due tomorrow from the Follow-Up Schedule
3. Week Ahead entries for tomorrow
4. Unanswered Slack messages from Step 6 that need PM action
5. Action items from standup notes (Step 2) not already in the tracker
6. Any new items the PM added in Step 3

Every item surfaced in the EOD review must appear in the tracker. Nothing should stay in the summary output only; if it needs PM attention tomorrow, it must be written to tomorrow's Daily To-Do.

Save the tracker to its existing path.

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```

Then sync the updated tracker to Google Drive using `mcp__claude_ai_Google_Drive__create_file` with title "client Weekly Tracker - CURRENT (Auto-synced)", parent folder ID `1ficiL4i0DJTKwpb770YoE8h1RtNEkaHe`, and the full updated markdown content.

### 9. Summary

Present a compact summary:

```
## EOD Summary - [Day, Month DD]

**Completed:** [X]/[Y] items
**Carry forward:** [N] items ([M] flagged as 3+ days stale)
**Needs attention tomorrow:** [N] unanswered messages
**Standup notes loaded:** [Yes - N items surfaced | No standup notes found]

### Top 3 Tomorrow
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

Tracker updated: [file path]
HTML: [html path]
```

---

## Rules

- **One interactive gate only.** Step 3 is the only point where the PM provides input. Everything else is autonomous.
- **Never auto-complete items.** Only mark items done when the PM confirms.
- **Carry-forward is the default.** If the PM doesn't mention an item, it carries forward; never drop it.
- **Stale items are surfaced, not hidden.** 3+ day carry-forwards are a signal. Always flag them.
- **Gmail drafts are not created by this skill.** This is a tracker-only skill.
- **No Jira or Slack writes.** This skill reads only.
- **Preserve all tracker content.** When updating, append and modify; never overwrite existing sections.

---

> **Skill verification:** Please ensure that the skill end-of-day SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
