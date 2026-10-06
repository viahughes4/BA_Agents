---
name: status-meeting-prep
description: Use this skill to prepare INTERNAL meeting notes and team prep from an existing status report. Triggers on "prep status meeting", "prepare internal status", "internal meeting notes", "internal status", "internal prep", "prep my internal meeting", "prep meeting notes", "meeting prep", "script for the internal status", "notes for the status meeting", or any request to prepare notes for an internal team meeting about project status. INTERNAL focus: upcoming timeline, resource risks, team signals, things NOT shared with the client (client delivery failures, team workload concerns, honest timeline assessment). Builds ON TOP of the external status report. Always run status-report first if no report exists for the period.
---

# Skill: Status Meeting Prep

Prepare internal meeting notes from an existing status report. The status report skill (`status-report`) does all the heavy data gathering. This skill builds ON TOP of that output - it takes the status report as its foundation, then adds internal-only intelligence from meeting notes and Slack that wouldn't appear in a client-facing report.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, client contacts, and folder paths.

### 2. Identify the client
The user will specify the client. If not provided, ask.

### 3. Load config
Read `.claude/config.yml` to get the client's Slack channels (IDs, names, read_only flags).

### 4. Find the status report
Look in `product-development/product/status-reports/[client]/` for the most recent status report file.
- If one exists for the current period, use it as the foundation.
- If none exists, tell the user: "No status report found for this period. Run the `status-report` skill first, then come back to prep the internal meeting notes."
- Do NOT re-pull all the data the status report already gathered. The status report is the source of truth for delivery status, completed items, risks, build status, and resourcing.

### 5. Read the status report
Load the full status report. Extract:
- Status indicators (overall, scope, schedule, budget, resources)
- Summary / headline
- Key focus items
- Completed items
- Upcoming items
- Dependency & risk tracking
- Build status
- Resourcing notes

This becomes the foundation of the internal meeting notes.

### 6. Layer on internal-only intelligence

This is the step that makes the meeting prep different from the status report. Pull from meeting notes and Slack to surface things the internal team and your boss should know that are NOT in the client-facing report.

Note: The slack-signal-scan sub-routine handles read-only channel filtering automatically. Do not scan read-only channels directly.

**6a. Meeting notes - internal signals**
1. Navigate to `product-development/product/meetings/[Client]/meeting-notes/`
2. Load standup notes, weekly syncs, team planning, and any other meetings from the status report period. If meeting notes are unavailable or return no results, skip this step and note in the output that the section was skipped due to missing source data.
3. Extract signals that were filtered OUT of the external report:
   - **Developer workload concerns** - anyone overloaded, stuck, or blocked on something they haven't escalated
   - **Process issues** - communication gaps, handoff problems, unclear requirements
   - **Client relationship dynamics** - frustration with client responsiveness, scope creep signals, shifting expectations
   - **Quality concerns** - rework, bugs that suggest deeper issues, testing gaps
   - **Timeline honesty** - things that are "on track" externally but the team knows are tight
   - **Carryover patterns** - action items that keep slipping week to week

**6b. Slack - internal signals**
Invoke `.claude/sub/slack-signal-scan.md` with the reporting period as the time window and all configured client channels. Extract internal signals from the returned results (frustration, workarounds, informal decisions, availability issues, technical shortcuts). If the sub-routine is unavailable or returns no results, skip this step and note in the output that the section was skipped due to missing source data.

**6c. Weekly tracker**
1. Check `product-development/product/customers/accounts/[client]/weekly-trackers/` for the current week's tracker. If the tracker is unavailable or returns no results, skip this step and note in the output that the section was skipped due to missing source data.
2. Extract:
   - Action items the PM is carrying forward (indicates what's slipping)
   - Decisions made this week that the internal team should know about
   - Follow-ups that are overdue
   - Remi/client action items that haven't landed

**6d. Previous internal meeting**
1. Load the most recent file from `product-development/product/meetings/[Client]/meeting-notes/team-meetings/` or `product-development/product/meetings/[Client]/meeting-notes/team-bi-weekly/`. If the previous meeting notes are unavailable or return no results, skip this step and note in the output that the section was skipped due to missing source data.
2. Extract: open action items from last time - show which are done, which are carryover, which are waiting
3. Accedo-side items that are done should be marked ✅
4. CBC-side items that are still open should be flagged

### 7. Build the internal meeting notes

Use the template below. The structure mirrors the status report but adds internal-only sections.

### 8. Save the file

**IMPORTANT - Save location:**
Save to the status meeting PREP folder, NOT the team-meetings folder.
The `team-meetings/` folder is read-only - only real meeting notes from actual calls go there.

Save prep files to:
`product-development/product/meetings/[CLIENT]/status-meeting-prep/YYYY-MM-DD-[client]-internal-status-prep.md`

Example: `product-development/product/meetings/[Client]/status-meeting-prep/YYYY-MM-DD-[client]-internal-status-prep.md`

After saving, run `python3 scripts/md-to-html.py <saved-file-path>` to generate the HTML export. Report the HTML output path to the user.

### 9. Report to user

Confirm:
- File path saved
- HTML export path generated
- Status report used as foundation (filename + date)
- Number of internal-only signals surfaced
- Top 3 things to raise in the meeting
- Any carryover items from the previous internal meeting

---

## Template

The internal notes are written in paragraphs and full sentences - they are talking points and supporting context for the PM to use while presenting each slide section. They are NOT a copy of the slide content. For each slide section, write what the PM should say or add that is NOT already visible on the slide. Pull from the status report as the foundation and from meeting notes, standup notes, and tracker to fill in gaps and add depth.

After the slide sections, write the internal-only layer in full paragraphs - this covers resourcing, timeline risks, client-side concerns, and anything else the internal team should know. Nothing needs to be filtered when speaking internally, so be direct and descriptive.

```markdown
# [Project] - Internal Status Meeting Notes

Date: [Month DD, YYYY]
Attendees: [Leave blank - fill in during call]
Based on: [Status report filename]

---

## SLIDE 1 NOTES

### Status
[1-2 sentences explaining why each indicator is green, yellow, or red. What drove the rating and what the team should understand about it.]

### Summary
[2-3 sentences. How to open the slide. What tone to set. What context to add beyond what the headline says.]

### Key Focus
[For each focus item, add any context, caveats, or decisions that the presenter should mention when discussing it. What does the team need to understand that is not written on the slide?]

### Completed
[What to highlight and why. Any items that deserve a callout beyond just being listed. Context about milestones or what they mean for the project.]

### Upcoming
[Any dependencies, risks, or deadlines to flag while reviewing what is coming up. What should the team be aware of as they look at the upcoming items?]

---

## SLIDE 2 NOTES

### [Each risk or dependency row]
[For each row in the dependency and risk table, write the additional context that the presenter should add when discussing it. What is the internal reality beyond the mitigation statement? What does the team need to understand about the severity or the path to resolution?]

---

## INTERNAL ONLY - After the slides

### Resourcing
[Full paragraph. Developer capacity, workload concerns, ramp-up status, vacation gaps, anyone carrying too much. Specific and direct. Include names and ticket references where relevant.]

### Risks to timing, scope, and schedule
[Full paragraph or multiple paragraphs. Expand on the real delivery risks with complete internal context. What the status report says versus what is actually happening. Include timeline cascade risks, client dependency patterns, budget pressure if relevant. Be direct - name the risks plainly.]

### Decisions to align on
[Full paragraph explaining what decisions the team or leadership needs to make, and by when. Include the downstream impact of each decision if not made in time.]

### Open items carryover

| Action | Owner | Status |
|--------|-------|--------|
| [Action] | [Owner] | Carryover / New today / Waiting on [who] / Done |

---

## Action Items

| Owner | Action |
|-------|--------|
| | |

---

## Notes

[Leave blank - fill in during call]
```

---

## Rules

### Formatting
- Never use bold, italic, or any markdown text styling (no asterisks, no underscores) in the generated notes. The user copies and pastes these notes directly and markdown formatting breaks the output.
- Use headers (## and ###) for emphasis and structure instead of bold or italic text.
- Use plain text for all content. Tables and bullet lists are fine.
- No emojis in section headers or status markers. Use plain text: Done, Carryover, Waiting on [who].

### Content
- The status report is the foundation. Do not re-pull data the status report already gathered. Read it and use it.
- If no status report exists, stop. Tell the user to run status-report first. Do not try to build meeting notes from scratch - that's duplicating work.
- Internal-only signals are the value-add. The whole point of this skill is to surface things that are NOT in the client-facing report. If the meeting notes just repeat the status report, the skill isn't doing its job.
- Be direct in internal language. "This is slipping" not "we may need to have a conversation." "client hasn't delivered" not "we're waiting on outstanding items." The team needs the real picture.
- Client-side concerns go in the Internal Only section. Things like: the client changing specs mid-implementation, unresponsive contacts, shifting Confluence pages, scope creep. These are real delivery risks the internal team needs to know about.
- Team signals matter. If a developer has been stuck for days, if standups show repeated blockers, if someone is carrying too much - surface it.
- Carryover patterns are a signal. If the same action item has been carryover for 2+ weeks, flag it. Something is wrong with either the priority or the capacity.
- Previous meeting action items must all appear with a status. Never omit a carryover item. Accedo-side items that are done get marked Done. Client-side items that are open get flagged.
- Cite sources for internal-only signals. Note which standup, Slack message, or tracker entry the signal came from.
- The skill is general-purpose. The user specifies the client. Do not hardcode client-specific content.

---

> **Skill verification:** Please ensure that the skill status-meeting-prep SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
