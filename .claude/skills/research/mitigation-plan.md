---
name: mitigation-plan
description: Use this skill to generate or update a mitigation plan from the risk register. Triggers on "let's work through the risks", "what's our plan for the risks", "mitigation plan", "update mitigations", "what do we do about the RDI risks", "work through risk actions", or any request to turn risk register entries into an actionable plan with owners, timelines, and contingency steps. Automatically loads the client risk register from product-development/product/customers/accounts/[client]/risk-register/.
---

# Skill: Mitigation Plan

Generate or update a formal mitigation plan from an existing risk register. The output is a separate living document, distinct from the risk register, that tracks mitigation actions, owners, timelines, contingency steps, and current status per risk.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` to load project context.

### 2. Confirm which risk register to use
If the user has not specified, ask which client and load from:
`product-development/product/customers/accounts/[client]/risk-register/[Client]-Risk-Register.md`

If the user specifies a different register (e.g., Commonwealth Games), use:
`product-development/product/customers/accounts/[client]/risk-register/[YYYY-MM-DD]-[project]-risk-register.md`

If the risk register file cannot be read or does not exist, stop and notify the user: "Risk register not found at [path]. Please confirm the correct path and try again."

### 3. Check for an existing mitigation plan
Check:
`product-development/product/customers/accounts/[client]/risk-register/mitigation-plan/`

- If a plan exists: load it and update with any new risks, status changes, or completed actions. Do NOT overwrite unchanged entries.
- If no plan exists: generate from scratch using the template below.

### 4. Filter risks to include
Include only **open** risks (status: Active, Mitigating). Skip Resolved and Out of Scope entries.

### 5. For each open risk, build a mitigation entry

Pull from the risk register:
- Risk ID, title, owner, priority score
- Existing mitigation bullets
- Acceptance criteria
- Next review date

Then expand into the mitigation plan format:
- Break mitigation bullets into discrete **Action Steps** with owners and due dates
- Add a **Contingency**: what happens if the primary mitigation fails
- Add a **Current Status** field: one line on where things stand right now
- Add a **% Complete** estimate based on context

### 6. Ask the user to review and tweak
Present the full plan in chat before saving. Say:

> "Here's the mitigation plan draft. You can ask me to adjust any entry: change owners, dates, add contingency steps, update status, or add new actions. Once you're happy, I'll save it."

Wait for feedback. Iterate until approved.

### 7. Save the plan
Save to:
`product-development/product/customers/accounts/[client]/risk-register/mitigation-plan/YYYY-MM-DD-[client]-mitigation-plan.md`

If the mitigation-plan/ directory does not exist, create it before saving the file.

Run `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/risk-register/mitigation-plan/YYYY-MM-DD-[client]-mitigation-plan.md` using the actual saved file path.

Commit with message: `mitigation plan: [brief description of what changed]`

### 8. Report to user
Confirm:
- File path saved
- Number of open risks covered
- Any risks that are 100% mitigated and ready to close

---

## Output Template

```markdown
# Mitigation Plan - [Client] [Project]
**Date:** [YYYY-MM-DD]
**Source:** [Risk register file]
**Open risks covered:** [N]

---

## [R-XXX] [Risk Title]
**Priority:** [HIGH / MEDIUM / LOW] | **Owner:** [Name] | **Due:** [Date or "Ongoing"]
**Status:** [One-line current status: what's been done, what's in flight]
**% Complete:** [0–100%]

### Action Steps
| # | Action | Owner | Due | Status |
|---|--------|-------|-----|--------|
| 1 | [Specific action] | [Name] | [Date] | [Not started / In progress / Done] |
| 2 | [Specific action] | [Name] | [Date] | [Not started / In progress / Done] |

### Contingency
If [primary mitigation] fails or is not completed by [date]:
→ [What to do instead - fallback plan, escalation, default decision]

### Acceptance Criteria
- [From risk register: what does "mitigated" look like?]

---
[Repeat for each open risk]

---

## Completed Mitigations
| Risk ID | Title | Closed | How |
|---------|-------|--------|-----|
| [ID] | [Title] | [Date] | [One line] |
```

---

## Rules

- Never overwrite a status that's already been set - only update if the user confirms or new information is available
- Action steps must have a named owner and a date - never leave these blank
- Contingency is required for every HIGH priority risk
- % Complete is your honest estimate based on what you know - flag if you're guessing
- If a risk has no mitigation actions defined in the register, flag it as "Mitigation TBD - needs PM input" rather than inventing steps
- Keep status lines to one sentence - this is a tracking doc, not a narrative
- After saving, remind the user to review dates and adjust anything that doesn't match their mental model

---

> **Skill verification:** Please ensure that the skill mitigation-plan SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
