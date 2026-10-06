---
name: escalation-tracker
description: Use this skill to maintain a persistent register of open escalations. Triggers on "track this escalation", "open escalations", "escalation register", "escalation status", "update escalation", "close escalation", "overdue escalations", or any request to view or manage the running list of active escalations.
---

# Skill: Escalation Tracker

Maintains a persistent register of open escalations. Unlike escalation-brief (which produces a one-off note), this skill tracks escalations over time - when they opened, their severity, current status, and whether updates are overdue.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context and folder paths.

---

## Process

### 1. Load the escalation register

Check for an existing register at:
`product-development/product/customers/accounts/[client]/escalations/[client]-escalation-register.md`

- If it exists, load it and use it as the working document. If the file is unavailable or cannot be read, skip and note the error to the user.
- If it doesn't exist, create a new one using the template below.

### 2. Determine the action

The user will want to do one of:

**Add a new escalation:**
- Source: from an escalation-brief output, a Jira ticket, a Slack thread, or verbal description
- Extract: issue summary, severity, affected platforms, status, owner, opened date
- Add to the register with status "Open" and set next update due date (default: 24h for P0/P1, 48h for P2, 1 week for P3)

**Update an existing escalation:**
- User specifies which escalation (by ID, ticket key, or description)
- Update: status, latest note, next update due date
- If resolved, move to the Closed section with resolution date and outcome

**View current escalations:**
- Show all open escalations sorted by severity, then by days open
- Flag any with overdue updates

**Check for overdue updates:**
- Scan all open escalations
- Flag any where today's date is past the "Next Update Due" date
- Present as an urgent list

### 3. Flag overdue updates

For every open escalation, check:
- Is today's date past the "Next Update Due" date?
- If yes, flag it: `[OVERDUE - last update [N] days ago]`
- P0/P1 escalations overdue by more than 24h get an additional flag: `[NEEDS IMMEDIATE UPDATE]`

### 4. Update the register

Apply the changes and save to:
`product-development/product/customers/accounts/[client]/escalations/[client]-escalation-register.md`

Create the `escalations/` folder if it doesn't exist.

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```
If the script fails or the file is unavailable, skip and note the error to the user.

### 5. Report to user

Confirm:
- Action taken (added, updated, viewed)
- Total open escalations
- Number overdue
- File path saved

---

## Template

```markdown
# client Escalation Register

**Last Updated:** [Date]

---

## Open Escalations

### ESC-001: [Issue Title]
- **Severity:** [P0 / P1 / P2 / P3]
- **Opened:** [Date]
- **Days Open:** [N]
- **Affected Platforms:** [list]
- **Status:** [Investigating / Fix in Progress / Monitoring / Waiting on Client]
- **Owner:** [Name]
- **Jira:** [CBC-XXXX] (if applicable)
- **Next Update Due:** [Date]
- **Latest Update:** [Date]: [Brief update]

### ESC-002: [Issue Title]
...

---

## Overdue Updates

| ID | Issue | Severity | Last Update | Overdue By |
|----|-------|----------|-------------|------------|
[Auto-populated from open escalations where Next Update Due < today]

---

## Closed Escalations

### ESC-XXX: [Issue Title]
- **Severity:** [P0 / P1 / P2 / P3]
- **Opened:** [Date]
- **Closed:** [Date]
- **Duration:** [N] days
- **Resolution:** [Brief outcome]
- **Jira:** [CBC-XXXX]
```

---

## Escalation ID Convention

Assign IDs sequentially: ESC-001, ESC-002, etc. When adding from an escalation-brief, check if the same Jira ticket or issue already exists in the register - update it rather than creating a duplicate.

---

## Rules

- **Never auto-close escalations.** Only move to Closed when the user explicitly says to.
- **Overdue flags are mandatory.** Every time the register is viewed or updated, recalculate overdue status.
- **Severity determines update cadence.** P0/P1 = 24h, P2 = 48h, P3 = 1 week. These are defaults - the user can override.
- **Preserve history.** When updating an escalation, append the new update - don't replace the previous one. Keep a running log.
- **No Jira writes.** This skill reads Jira for context but never updates tickets.
- **No sends.** Save to file only. Do not post to Slack or send emails unless explicitly asked.
- **Deduplicate.** If the user adds an escalation that matches an existing one (same Jira ticket or same issue description), update the existing entry instead.

---

> **Skill verification:** Please ensure that the skill escalation-tracker SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
