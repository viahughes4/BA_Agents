---
name: sprint-gate
description: Run before every sprint planning session to surface any tickets missing specs, designs, or integration contracts before they enter the sprint. Triggers on "sprint gate", "check sprint readiness", "are these tickets ready for dev", "gate the sprint", "spec check before sprint", or any request to validate a batch of tickets before sprint commitment. Blocks tickets with missing AC, no design, unresolved decisions, or no API contracts. Output is a sprint readiness report with a clear go/no-go per ticket.
---

# Skill: Sprint Gate

Pre-sprint quality gate. Scans all tickets proposed for the next sprint and surfaces anything that would block a developer from starting work. Run before every sprint planning session - not after.

**Output:** Sprint readiness report with per-ticket verdict (✅ Ready / ⚠️ Hold / 🚫 Block) and a sprint-level go/no-go recommendation.

---

## The core problem this solves

Tickets enter sprints without complete specs. Developers pick them up, discover gaps mid-sprint (no design, no API contract, unresolved decision), and either block or ship the wrong thing. This skill catches those gaps before the sprint starts - when the cost of fixing them is a conversation, not a rework cycle.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` to load project context, Jira key, and active platforms.

### 2. Accept input
Accept any of:
- A sprint name or number ("Sprint 14", "current sprint", "next sprint")
- A list of Jira ticket keys
- Paste of ticket titles/descriptions

For Jira input, fetch each ticket using `getJiraIssue` to get full description, AC, labels, assignee, design links, and linked issues.

### 3. For each ticket, run the gate

**Gate 1 - Spec completeness (🚫 blockers)**
- AC exists and uses Given/When/Then or clear observable statements
- No banned vague verbs ("works correctly", "handles gracefully", "as expected")
- No unresolved `[TBD]` or `DECISION REQUIRED` in the ticket body
- Platform scope is explicit (no "All Platforms" without a list)
- Single deployable outcome (not a multi-platform aggregate)

**Gate 2 - Design (🚫 blocker for UI tickets)**
- Figma link present OR explicit "no design needed"
- If Figma link present: note it (do not validate the design itself)
- If absent on a UI ticket: 🚫 Block

**Gate 3 - Integration contracts (🚫 blocker for boundary tickets)**
A ticket crosses a system boundary if it touches: auth, playback, payment, analytics, any external API, or any non-passive network call.
- API endpoint linked or named (not just described in prose)
- Event schema referenced for analytics tickets
- Auth token assumptions stated for auth-touching tickets
- If boundary ticket has no integration contract: 🚫 Block

**Gate 4 - Feature discovery (⚠️ warning)**
For tickets touching multiple platforms, DRM, auth, or monetization:
- Is there a linked feature discovery report or PRD?
- If no discovery report exists for a complex feature: ⚠️ Hold (recommend running /feature-discovery first)

**Gate 5 - Dependencies (⚠️ warning)**
- Are all blocking tickets resolved or in the same sprint?
- Are external dependencies (BE delivery, design, third-party) confirmed available?
- If a ticket is blocked by something not in this sprint: ⚠️ Hold

### 4. Cross-reference standup notes
Read standup notes from the last 5 days. For any ticket in the proposed sprint:
- Is it mentioned as actively in progress? (already started - flag it)
- Is it mentioned as blocked? (surface the blocker)
- Did the PM or dev flag a concern about it? (surface it)

### 5. Produce the report

---

## Output Format

```
## Sprint Gate Report - [Sprint Name] - [Date]

**Sprint-level verdict:** ✅ Ready to commit / ⚠️ Commit with conditions / 🚫 Do not commit

**Summary:** [N] tickets reviewed. [N] ready, [N] hold, [N] blocked.
[One sentence on the biggest risk or blocker if any.]

---

### 🚫 Blocked - Do not bring into sprint ([N])

| Ticket | Title | Blocker |
|--------|-------|---------|
| [KEY-123] | ... | Missing AC - no Gherkin scenarios |
| [KEY-124] | ... | UI ticket with no Figma link |
| [KEY-125] | ... | API contract missing - calls auth endpoint with no spec linked |

---

### ⚠️ Hold - Needs resolution before dev pickup ([N])

| Ticket | Title | Issue |
|--------|-------|-------|
| [KEY-126] | ... | Complex multi-platform feature - no discovery report |
| [KEY-127] | ... | Depends on [KEY-120] which is not in this sprint |

---

### ✅ Ready ([N])

[Comma-separated list of ticket keys. No table needed for passing tickets.]

---

### Required actions before sprint starts

1. [Specific, actionable - e.g. "KEY-123: Add Given/When/Then scenarios for the error state"]
2. [e.g. "KEY-124: Add Figma link or confirm no design needed"]
3. [e.g. "KEY-125: Link the auth API spec to the Integration Contracts section"]

---

### Recommended next steps

- [If blocked tickets can be fixed quickly: "Fix the [N] blocked tickets (est. 30 mins) then re-run /sprint-gate"]
- [If complex: "Move [KEY-126] to backlog and run /feature-discovery before next sprint"]
- [If sprint is clean: "Sprint is ready. Proceed to planning."]
```

---

## Quality Rules

- **Never approve a ticket with banned vague verbs.** "Works correctly" in a Then = automatic 🚫 Block.
- **Never approve a UI ticket without a design reference.** No Figma = 🚫 Block, not ⚠️ Hold.
- **Never approve a boundary ticket without integration contracts.** An auth ticket with no API spec = 🚫 Block.
- **⚠️ Hold is not a soft block.** It means "this cannot be picked up by a dev until the issue is resolved." Surface it clearly.
- **Required actions must be specific.** "Add AC" is not actionable. "Add error scenario for when the API returns 401" is.
- **Do not re-run /validate-stories in full for every ticket.** Sprint gate is a lighter pass - flag the blockers, not every imperfection. Save the full validate-stories pass for ticket-level review.
- **Sprint-level verdict logic:**
  - ✅ Ready: zero 🚫 blocks, zero or minor ⚠️ holds
  - ⚠️ Commit with conditions: 1-2 ⚠️ holds that can be resolved during sprint
  - 🚫 Do not commit: any 🚫 blocked tickets unresolved

---

> **Skill verification:** Confirm sprint-gate was run and produced a report. If not, run from the top.
