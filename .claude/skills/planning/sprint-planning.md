---
name: sprint-planning
description: Use this skill to prepare for or run a sprint planning session. Triggers on "sprint planning", "plan the sprint", "plan next sprint", "what should we work on next sprint", "sprint capacity", "pick sprint tickets", or any request to select and assign work for an upcoming sprint. Checks team capacity, pulls ready tickets from the backlog, handles carryover, and produces a proposed sprint plan for PM review.
---

# Skill: Sprint Planning

Prepare a proposed sprint plan by checking team capacity, surfacing ready backlog tickets, and handling carryover from the current sprint. The PM reviews and confirms - the skill does not write to Jira until approved.

---

## Process

### 1. Load context

Read `.claude/sub/context-loader.md` to load:
- Jira project key, cloud ID, and API config
- Active sprint name and backlog sprint name
- Team roster (developers, QA) and any known availability gaps
- Sprint folder path for saving output

### 2. Confirm the sprint window

Ask the user (or infer from current date):
- Sprint start date and end date
- Any team member PTO, holidays, or partial availability this sprint

### 3. Check carryover

Query Jira for tickets in the current active sprint that are not Done or Closed:
```
project = [PROJECT_KEY] AND sprint = "[ACTIVE_SPRINT]" AND status not in (Done, Closed) ORDER BY assignee ASC, priority DESC
```

For each carryover ticket, evaluate:
- Why was it not completed? (check status, last comment, linked blockers)
- Is the scope unchanged? If blocked or scope-changed, flag for PM decision before re-committing
- Is it still a priority? Or should it return to backlog?

Present carryover tickets as the first decision the PM must make before selecting new work.

### 4. Pull ready backlog tickets

Query Jira for the grooming-ready backlog:
```
project = [PROJECT_KEY] AND sprint = "[BACKLOG_SPRINT]" AND status in (Open, "To Do") ORDER BY priority ASC
```

Filter for tickets that pass the readiness bar (from the backlog-grooming skill):
- Has a meaningful description (over 150 characters)
- Has at least one acceptance criterion signal
- Has an assignee or is in the PM holding queue

Flag any ticket that does not pass as "needs grooming before pulling in."

### 5. Assess capacity

For each developer and QA member, estimate available capacity for the sprint:
- Standard sprint: 10 working days
- Subtract known PTO days and any sprint ceremony time (planning, review, retro: ~0.5 days)
- Apply 80% rule: plan to 80% of total available days to buffer for carryover and unexpected work
- Note any developer who is mid-task from last sprint (likely needs first 1-2 days to close out)

Present capacity as a simple table: Developer | Available days | 80% target | Carryover load.

### 6. Propose the sprint

Match ready backlog tickets to developers based on:
- Existing expertise (platform knowledge, ticket history)
- Current load (carryover tickets first, then new work)
- Priority order (Critical and High before Medium and Low)
- Dependencies (do not pull in a ticket blocked on another unstarted ticket)

Flag any ticket with an unresolved external dependency (client BE, third-party SDK, design not finalized) as a risk before committing.

Present the proposed sprint as a per-developer table. Do not write anything to Jira until the PM explicitly approves.

### 7. Identify stretch tickets

If capacity allows after the primary plan, suggest 1-2 stretch tickets per developer - lower priority items that can be picked up if the primary work closes early. Mark these clearly as stretch.

### 8. Save the sprint plan

Save to:
`[sprints-path]/YYYY-MM-DD-sprint-plan-[sprint-name].md`

Run `python3 scripts/md-to-html.py` on the saved file if the script exists. Report the HTML path.

### 9. Get PM approval before writing to Jira

Present the full proposed plan and end with:

"Does this look right? I'll assign the tickets in Jira and move them into the sprint once you confirm."

Do not touch Jira until the PM says yes, confirmed, looks good, or equivalent.

---

## Output Template

```markdown
# Sprint Plan - [Sprint Name]
Date: [YYYY-MM-DD]
Sprint window: [Start] to [End]

---

## Carryover from Current Sprint

[Decisions needed before we continue:]

| Ticket | Summary | Reason Not Done | Recommendation |
|--------|---------|----------------|----------------|
| [key] | [summary] | [blocked / scope changed / still in progress] | [Re-commit / Return to backlog / Descope] |

---

## Team Capacity

| Developer | Available Days | 80% Target | Carryover Load |
|-----------|---------------|------------|----------------|
| [name] | [N] | [N x 0.8] | [N tickets carrying over] |

---

## Proposed Sprint

### [Developer 1]
| Ticket | Summary | Priority | Notes |
|--------|---------|----------|-------|
| [key] | [summary] | [priority] | [any flag] |

### [Developer 2]
| Ticket | Summary | Priority | Notes |
|--------|---------|----------|-------|

### QA
| Ticket | Summary | Priority | Notes |
|--------|---------|----------|-------|

---

## Stretch (if capacity allows)
| Ticket | Developer | Summary | Priority |
|--------|-----------|---------|----------|

---

## Tickets Not Pulled In - Reason

| Ticket | Summary | Why Not This Sprint |
|--------|---------|---------------------|
| [key] | [summary] | [blocked / not groomed / lower priority] |

---

## Risks

[Any tickets with unresolved external dependencies, design gaps, or tight deadlines flagged here.]

---

## Sprint Goal

[One sentence describing what success looks like at the end of this sprint. Written by the PM during review.]

---

## Notes

[Fill in during session]
```

---

## Rules

- No em dashes. Use hyphens or colons instead.
- Carryover tickets are reviewed first, before any new work is selected. This is not optional.
- Plan to 80% of capacity. Never fill to 100%.
- Do not pull in a ticket blocked on an unresolved external dependency. Flag it and leave it in the backlog.
- Do not pull in a ticket that has not passed the readiness bar (no description, no AC, no assignee). Flag it as needing grooming.
- Never write to Jira without explicit PM approval. Show the full plan first.
- If the PM has not run backlog-grooming recently (within 7 days), recommend running it before sprint planning: "The backlog may have ungroomed tickets. Run /backlog-grooming first to get a clean ready list."
- The sprint goal is written by the PM, not generated by the skill. Leave it blank in the template.

---

> Skill verification: Please ensure that the skill sprint-planning SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
