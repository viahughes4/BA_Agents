---
name: biweekly-sync-prep
description: Use this skill to prepare slide content for an internal biweekly team sync (sprint planning and alignment call). Triggers on "biweekly prep", "prep the biweekly", "update the team slides", "biweekly sync", "internal team meeting slides", "sprint planning slides", or any request to prepare for the internal team biweekly. Generates slide-ready content covering the last two weeks of completed work and the next two to three weeks of planned work. Content is copied directly into the team's slide deck.
---

# Skill: Biweekly Sync Prep

Prepare slide-ready content for the internal team biweekly sync. Focus: what was finished in the last two weeks, and what is coming in the next two to three weeks. This is an internal sprint planning and alignment call - not a client-facing meeting.

Configure the slide deck URL in `.claude/config.yml` under `biweekly_deck_url`. If not configured, output only the text content.

---

## Slide Structure

Standard internal biweekly deck has 13 slides. Only content slides need populated text. Divider slides are noted but skipped.

| Slide | Name | Type |
|-------|------|------|
| 1 | Title | Content |
| 2 | Release Overview | Content |
| 3 | Sprint Focus | Divider |
| 4 | Engineering | Content |
| 5 | QA | Content |
| 6 | Release Planning | Divider |
| 7 | Roadmap Update | Divider |
| 8 | Roadmap: [feature] | Divider |
| 9 | Blocked Features | Content |
| 10 | Upcoming Roadmap | Divider |
| 11 | Client Priorities | Content |
| 12 | Open Issues / Team Discussion | Content |
| 13 | Thank you | Divider |

---

## Process

### 1. Load context

Read `.claude/sub/context-loader.md` to load team roster, folder paths, and project config. Read `.claude/config.yml` for the slide deck URL if configured.

### 2. Confirm the date

Use `currentDate` from context. Lookback window: 14 days. Forward window: 14-21 days.

### 3. Load the roadmap

Read the project roadmap file (path from context-loader). Extract:

**Completed (last 14 days):**
Items marked Done, Closed, or Merged with a date in the lookback window from each developer section.

**In progress now:**
Each developer's active Now section.

**Coming next (14-21 days):**
Each developer's Up Next section and any upcoming milestones or key dates.

**Blocked:**
Any item marked Blocked across all developer sections, and PM holding queue items blocked on external parties.

### 4. Load recent standup notes

Read the five most recent standup files from the project's standup notes folder. Extract per-developer updates, blockers, and items that moved or got stuck in the last two weeks. Use to add texture to the engineering slide.

### 5. Load the risk register

Read the active risk register file. Extract HIGH and MEDIUM risks that are currently open. These feed into slide 9 (Blocked Features) and slide 12 (Open Issues).

### 6. Check the weekly tracker

Read the most recent weekly tracker file. Extract open action items, unresolved dependencies, and any items in the "Waiting on Others" table. These feed into slide 12.

### 7. Build slide content

Generate slide-ready copy for slides 1, 2, 4, 5, 9, 11, and 12 only. Skip divider slides (3, 6, 7, 8, 10, 13). Write in short single-line bullet points. No em dashes. No markdown bold or italic.

See template below.

### 8. Save the prep file

Save to:
`[meetings-path]/biweekly-sync-prep/YYYY-MM-DD-biweekly-sync-prep.md`

Run `python3 scripts/md-to-html.py` on the saved file if the script exists. Report the HTML path.

### 9. Report to user

- File saved, HTML generated
- Count of completed items in the recap
- Top 3 things to raise in the meeting
- Any open risks or blockers that need team decision

---

## Output Template

```markdown
# Internal Team Sync - Slide Content
Date: [Month DD, YYYY]
[Deck URL if configured]

---

## SLIDE 1 - Title

[Project] Internal Team Sync - Release & Sprint Alignment
[Company] | [Year]
[Month DDth, YYYY]

---

## SLIDE 2 - Release Overview

Success Criteria and Key Upcoming Deliverables:
[3-6 bullets: mix of completed items from last 2 weeks and targets for next 2 weeks. Lead with most important delivery.]

Release Health: [Green / Yellow / Red]
Development Status: [Green / Yellow / Red]
QA Readiness: [Green / Yellow / Red]

Target Release: [version or milestone]
Release Window:
- [Platform A]: [status]
- [Platform B]: [status]

---

## SLIDE 4 - Engineering

Completed since last biweekly (2 weeks lookback):
[Bullet per merged/closed ticket: key + one-line description. Label each as Done.]

[Developer 1 name]:
[3-5 bullets: active tickets, PRs in review, what is blocked. One line each.]

[Developer 2 name]:
[3-5 bullets.]

[Developer 3 name]:
[3-5 bullets.]

---

## SLIDE 5 - QA

Active QA work:
[3-5 bullets: what is in QA now, what is queued, cert or build status. One line each.]

Notes:
[QA dependencies, blockers, or timing risks. One line each.]

---

## SLIDE 9 - Blocked Features

Feature | Reason
[ticket key + name] | [who is blocking and what is needed]

---

## SLIDE 11 - Client Priorities

[Priority Area 1]:
[2-3 bullets: date, owner, status.]

[Priority Area 2]:
[2-3 bullets.]

[Priority Area 3]:
[2-3 bullets.]

---

## SLIDE 12 - Open Issues / Team Discussion

Any blockers?
[Named blockers that need team input. Not generic questions.]

Any tickets at risk?
[Tickets near deadline or stuck. One line each.]

Dependencies from external teams?
[Outstanding items owned by client, vendors, or third parties. One line each.]

---

## Action Items from this meeting

| Owner | Action |
|-------|--------|
| | |

---

## Notes

[Fill in during call]
```

---

## Rules

- No em dashes. Use hyphons or colons instead.
- No markdown bold or italic. Content is pasted directly into slides.
- One bullet per line. No sub-bullets.
- Completed items go at the top of the engineering slide under a clear "Completed since last biweekly" header.
- Forward-looking work goes under each developer's name.
- Slide 9 (Blocked Features) lists only items where external action is needed.
- Slide 11 (Client Priorities) covers milestones and major features - not individual ticket assignments.
- Slide 12 (Open Issues) lists real named blockers from the risk register and tracker - not placeholder questions.
- Divider slides are omitted entirely from the output.
- Tone is internal and direct. "Blocked on [client]" not "awaiting client input."
- If a developer has no active dev work (all PRs in review), say so explicitly.

---

> Skill verification: Please ensure that the skill biweekly-sync-prep SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
