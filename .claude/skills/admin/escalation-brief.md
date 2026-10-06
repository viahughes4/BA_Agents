---
name: escalation-brief
description: Use this skill when the user needs to write a client-facing escalation note for an active issue. Triggers include "write an escalation", "escalation brief for [issue]", "client escalation", "draft escalation note", "client escalation", "write up this incident", "escalation for [ticket]", or when the user shares a Jira ticket or Slack thread describing an urgent issue that needs client communication.
version: 1.0.0
---

# Skill: Escalation Brief

Generates a concise, professional client-facing escalation note from a Jira ticket, Slack thread, or verbal description. Output is ready to finalize - no filler, no alarm.

**Never send to Slack, Jira, or email without explicit instruction.**

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths.

---

## Inputs

Provide at least one of:
1. **Jira ticket key** - e.g. `<jira.project_key>-5212`
2. **Slack thread or channel link** - read `.claude/sub/slack-context-extractor.md` and follow its process
3. **Verbal description** - user describes the issue directly

---

## Process

### 1. Gather context

**From Jira (if ticket provided):**

Read `.claude/sub/jira-project-snapshot.md` and follow its process to retrieve ticket context for the provided ticket key.

If a Jira ticket cannot be retrieved, note the error and ask the user to describe the issue verbally. If the Slack thread is inaccessible, note it and proceed with available context. If context-loader is unavailable, skip and proceed with session context.

**From Slack (if thread provided):**
- Extract: what happened, when, who reported it, current status, any decisions made, urgency signals
- Note any brain captures or pinned messages

**From verbal input:**
- Extract: issue description, affected users/platforms, current status, owner

If context is thin, ask one focused question before proceeding. Do not generate a half-baked brief.

### 2. Determine severity

| Level | Criteria |
|-------|----------|
| **P0: Critical** | Service down, cert failure, data loss, broadcast impact |
| **P1: High** | Core feature broken for a platform, blocking cert, wide user impact |
| **P2: Medium** | Degraded experience, workaround exists, limited user impact |
| **P3: Low** | Cosmetic or edge case, no blocking impact |

Surface the severity and confirm with the user if unclear.

### 3. Generate escalation brief

```markdown
**[Client] Escalation - [Date]**

**Issue:** [One sentence describing what is broken and where]
**Severity:** [P0 / P1 / P2 / P3]
**Affected platforms:** [Samsung / LG / Xbox / X1 / Xumo / All]
**Status:** [Investigating / Fix in progress / Fix deployed / Monitoring]

**What happened:**
[2-3 sentences. Factual. When it was detected, what the impact is, what caused it if known.]

**Current actions:**
- [Owner] is [doing X]
- [Expected resolution / next update by: time/date]

**Workaround (if any):**
[One sentence or "None at this time."]

**Jira:** [CBC-XXXX]
```

### 4. Present and confirm

Show the brief in chat. Ask:
- Is the severity correct?
- Who should this go to? (client DM group, specific person, Slack channel)
- Any changes before we finalize?

Do not send until explicitly confirmed.

### 5. Deliver (on confirmation only)

- **Slack DM to client group** (`C09MKNACR26`) - only if user explicitly says to post to Slack
- **Slack channel** - only if user specifies
- **Add as Jira comment** - only if user specifies. Before adding as a Jira comment, display the exact comment text in chat and ask: "Does this look right? I will add it to [CBC-XXXX] once you confirm." Only post after the user says yes.
- **Gmail draft** - always create a Gmail draft of the escalation brief using `mcp__claude_ai_Gmail__create_draft`. Pre-fill the subject as `[Client] Escalation - [Date] - [Issue summary]`. Do not pre-fill recipients unless the user specifies them. Report that a draft was created. Never offer to send.

### 6. Save (optional)

Optionally, save the brief to `product-development/product/customers/accounts/[client]/escalations/YYYY-MM-DD-[issue-summary]-escalation-brief.md`. If saved, run `python3 scripts/md-to-html.py <path>` to generate the HTML export and report the HTML path to the user.

---

## Quality Rules

- **Factual only** - do not speculate on root cause unless confirmed. Write "under investigation" if cause is unknown.
- **Tone: calm and direct** - escalations should communicate control, not crisis. Avoid "urgent", "critical failure", "disaster". Use severity labels instead.
- **No internal context in client-facing output** - strip team names, internal Slack references, and unconfirmed diagnoses from the brief.
- **Always include a next update time** - the brief is incomplete without a commitment on when the next update will come.
- **Short** - the brief should fit in a Slack message without scrolling. If there's more to say, use a thread reply.

---

> **Skill verification:** Please ensure that the skill escalation-brief.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
