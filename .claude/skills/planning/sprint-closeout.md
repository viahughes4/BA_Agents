---
name: sprint-closeout
description: Use this skill to close out a sprint with velocity calculation, carryover analysis, and next sprint planning inputs. Triggers on "sprint closeout", "close the sprint", "sprint review", "sprint velocity", "sprint summary", "what shipped this sprint", or any request to summarize and close a sprint.
---

# Skill: Sprint Closeout

Replaces the manual sprint-closeout shell script. Pulls sprint data, calculates velocity, identifies carryover with reasons, generates sprint review notes, and drafts next sprint planning inputs. Saves to the sprints folder with HTML.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths.

---

## Process

### 1. Identify the sprint

Ask the user if not specified:

> **Which sprint are you closing out? (e.g., "current sprint", "Sprint 42", or "the sprint ending this week")**

### 2. Pull sprint data

**If a `## Jira Snapshot` block is already present in context (pre-loaded by batch-runner), use it directly.**

Otherwise, invoke `.claude/sub/jira-project-snapshot.md` with:
- **Project key:** from config
- **Time window:** sprint duration (typically 2 weeks)
- **Sprint scope:** the sprint being closed

If the snapshot returns no issues or is unavailable, surface a warning to the user and ask them to confirm the sprint name or date range before continuing.

From the snapshot, categorize all issues:

| Category | Criteria |
|----------|----------|
| **Delivered** | Status = Done, Closed, Resolved, Released |
| **Carried Over** | Status = In Progress, To Do, Open, Ready for QA - still in the sprint |
| **Blocked** | Status = Blocked |
| **Removed from Sprint** | Issues that were in the sprint at start but removed mid-sprint (if detectable) |

### 3. Calculate velocity

From the delivered issues:
- **Story points completed:** sum of story points on delivered issues (if story points are used)
- **Issue count completed:** count of delivered issues
- **Points planned:** total story points in the sprint at start (if available)
- **Completion rate:** delivered / total (by count and by points)

If previous sprint data is available (from earlier closeout files in the sprints folder), compare:
- Velocity trend (up, down, stable)
- Average velocity over last 3 sprints

### 4. Analyze carryover

For each carried-over issue, determine the likely reason:
- **Blocked:** issue has a blocked status or linked blocker
- **Scope added mid-sprint:** issue was added after sprint start
- **Underestimated:** in progress but not completed (work started, didn't finish)
- **Not started:** still in To Do / Open
- **Waiting on external:** blocked on client, third party, or another team

Cross-reference with standup notes from the sprint period to validate reasons. If a standup mentions why something slipped, use that context.

### 5. Load meeting notes for context

Read standup notes from the sprint period from `product-development/product/meetings/[Client]/meeting-notes/standup/`:
- Extract any sprint-relevant decisions or scope changes
- Note any team capacity issues (vacations, sick days, onboarding)
- Identify any mid-sprint reprioritization

If no standup notes exist for the sprint period, skip this step and note in the Sprint Observations section that no standup context was available.

### 6. Generate sprint review notes

Use this template:

```markdown
# Sprint Closeout: [Sprint Name] - [Date Range]

**Closeout Date:** [Today]
**Sprint Duration:** [N] days

---

## Velocity

| Metric | This Sprint | Previous Sprint | 3-Sprint Avg |
|--------|-------------|-----------------|--------------|
| Story Points Completed | [N] | [N] | [N] |
| Issues Completed | [N] | [N] | [N] |
| Points Planned | [N] | [N] | [N] |
| Completion Rate (points) | [N]% | [N]% | [N]% |
| Completion Rate (count) | [N]% | [N]% | [N]% |

**Velocity Trend:** [Up / Down / Stable] - [brief context, no em dashes]

---

## Delivered - [N] Issues ([N] points)

| Ticket | Summary | Type | Assignee | Points |
|--------|---------|------|----------|--------|
[rows]

---

## Carried Over - [N] Issues ([N] points)

| Ticket | Summary | Assignee | Points | Reason |
|--------|---------|----------|--------|--------|
[rows]

---

## Blocked - [N] Issues

| Ticket | Summary | Assignee | Blocker |
|--------|---------|----------|---------|
[rows]

[If 0: "No blocked issues at sprint close."]

---

## Sprint Observations

### What went well
- [From standup notes and completion data - specific accomplishments]

### What slipped
- [Carryover patterns, scope changes, blockers]

### Capacity notes
- [Team availability issues during the sprint]

---

## Next Sprint Planning Inputs

### Carryover candidates (recommend including)
| Ticket | Summary | Points | Priority | Reason to include |
|--------|---------|--------|----------|-------------------|
[Top carryover items that should go into next sprint]

### Recommended focus areas
- [Based on current blockers, client priorities, and roadmap]

### Capacity considerations
- [Known vacations, onboarding, or availability changes for next sprint]

### Risks for next sprint
- [Carryover patterns that may repeat, dependencies, external blockers]
```

### 7. Save the closeout

Save to: `product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-sprint-[N]-closeout.md`

Create the `sprints/` folder if it doesn't exist.

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```

### 8. Report to user

Confirm:
- File path saved
- HTML path (same directory as the markdown file, inside an `html/` subfolder)
- Velocity summary (points completed, completion rate)
- Number of carryover issues and top reasons
- Top 3 recommendations for next sprint

---

## Rules

- **Pull from Jira first.** Sprint data comes from Jira. Meeting notes add context but don't replace ticket data.
- **Never fabricate velocity.** If story points aren't used, report by issue count only. Don't invent point values.
- **Carryover reasons must be evidence-based.** Use standup notes, blocked status, and sprint membership dates - not guesses.
- **Previous sprint comparison is optional.** If no previous closeout files exist, skip the trend comparison and note it.
- **Link all ticket keys** to the Jira instance from config.
- **No Jira writes.** This skill reads only. Do not close sprints, move tickets, or update statuses.
- **No Slack or email writes.** Present in chat and save to file only.
- **No em dashes.** All output, templates, and file content must use hyphens (-) or colons (:) instead of em dashes.
- **Read standup notes from the correct folder.** Use `product-development/product/meetings/[Client]/meeting-notes/standup/` for source meeting notes. Never read from or write to a prep folder.

---

> **Skill verification:** Please ensure that the skill sprint-closeout SKILL.md was actually run. If the skill was not run, it needs to be run again from the top. Expected output: a saved closeout file at `product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-sprint-[N]-closeout.md`, a corresponding HTML file, and a velocity + carryover summary presented in chat.
