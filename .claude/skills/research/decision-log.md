# Skill: Decision Log

**Triggers:** "create decision log", "update decision log", "add to decision log", "log this decision", "client decision log", "what decisions are pending", "decision tracker"

Creates and maintains a client-facing decision log tracking decisions that client / the client must make or has made that affect delivery. Separate from internal PM decisions.

---

## When to Use

Use this skill whenever:
- A client decision is pending and blocking work
- A verbal agreement has been made that needs documenting
- A previously confirmed decision has changed
- A scope, timeline or budget decision has been made on the call
- You want a summary of all unresolved client decisions

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` to load project foundation, team, and folder paths.

### 2. Find or create the decision log
Check for an existing decision log at:
`product-development/product/customers/accounts/[client]/decisions/[client]-decision-log.md`

If it exists - read it and work from the existing content.
If it does not exist - create it using the template below.

### 3. Auto-scan for decisions (run before accepting PM input)

Before asking the PM for input, automatically scan:

1. **Weekly tracker** - read the current week's tracker file from `product-development/product/customers/accounts/[client]/weekly-trackers/`. Extract:
   - Any item in "Decisions Made This Week" not already in the decision log
   - Any action items flagged as "Waiting on CBC" or "Waiting on client" that represent a pending decision
   - Any notes from meeting sections that contain a decision or open question

2. **Meeting notes** - scan the most recent files in `product-development/product/meetings/[CLIENT]/[client]-meeting-notes/` (last 7 days). Extract any decisions made or questions raised that are not already in the log.

3. **Risk register** - check for any risk items that represent an unresolved client decision not already logged.

Surface any newly found decisions to the PM before writing them to the log:
> "I found [N] potential decisions to add from this week's tracker and meeting notes. Here's what I found - confirm which to add."

### 4. Parse PM input
Accept new decisions in any format. Extract:
- **Decision:** What needs to be decided or what was decided
- **Status:** Pending / Confirmed / Overridden / Deferred
- **Owner:** Who at the client must make or has made the decision
- **Date:** When it was raised or confirmed
- **Impact:** What is blocked or affected if not resolved
- **Context:** Background, options considered, constraints

### 4. Populate or update the log
- Add new decisions under the correct status section
- Update existing decisions when status changes (e.g. Pending → Confirmed)
- Never delete decisions - move them to the Confirmed or Overridden section with a resolution note
- Flag any Pending decisions that are overdue or blocking sprint work

### 5. Save and export
Save to: `product-development/product/customers/accounts/[client]/decisions/[client]-decision-log.md`

Run: `python3 scripts/md-to-html.py [path]`

Report to user:
- Number of pending decisions
- Any decisions that are overdue or actively blocking work
- Any decisions confirmed this session

---

## Output Template

```markdown
# [Client] - Client Decision Log

**Project:** [Project name]
**PM:** [PM name]
**Last updated:** [Date]

---

## Pending Decisions (client / Client action required)

These decisions are unresolved. Until confirmed, they represent a risk to delivery.

| # | Decision | Owner | Raised | Blocking | Notes |
|---|---------|-------|--------|---------|-------|
| D-001 | [What needs to be decided] | [Name] | [Date] | [What is blocked] | [Context] |

---

## Confirmed Decisions

| # | Decision | Confirmed by | Date | Outcome |
|---|---------|-------------|------|---------|
| D-002 | [What was decided] | [Name] | [Date] | [What was agreed] |

---

## Deferred / Under Review

| # | Decision | Status | Date | Notes |
|---|---------|--------|------|-------|

---

## Overridden / Changed

Decisions that were previously confirmed but have since changed.

| # | Original decision | Changed to | Date | Reason |
|---|-----------------|-----------|------|--------|

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
```

---

## Rules

- Only log decisions that require or have required CLIENT action - not internal Accedo decisions
- Every pending decision must have a named owner at the client and a clear statement of what is blocked
- When a decision is confirmed, move it to the Confirmed section with the outcome noted - never delete it
- If a confirmed decision later changes, move it to Overridden with both the original and new decision recorded
- Flag any pending decision that has been open for more than 2 weeks as overdue
- Decisions that affect scope, timeline or budget must be logged - verbal agreements are not enough
- The decision log is the PM's protection if a client later says "we never agreed to that"

---

> **Skill verification:** Please ensure that the skill decision-log SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
