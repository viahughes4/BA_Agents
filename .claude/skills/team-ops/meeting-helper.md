---
name: meeting-helper
description: Use this skill when the user wants to process meeting notes, transcripts, or summaries into actionable PM tasks. Triggers on "process my meeting notes", "extract action items from this meeting", "turn this transcript into tasks", "what do I need to do from this meeting", "digest these notes", "summarize my meeting", "meeting recap", "notes from today's call", "what came out of the meeting", "add meeting notes", "log this meeting", "capture this call", or any request to convert meeting content into a prioritized PM action digest.
---

# Skill: Meeting Helper

Process meeting transcriptions, automated notes, or summaries into a prioritized PM action digest. Optionally create Jira tasks for "Today" actions.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths.
If context-loader.md is unavailable, proceed with default the client's Jira project context and note the gap.

### Optional: Slack context
If the user has provided a Slack message, thread, or channel link alongside their input, read `.claude/sub/slack-context-extractor.md` and follow its process in full. Merge the returned context block into the working context before proceeding. If no Slack input is provided, skip this step.

### 2. Accept input
- Single or multiple Google Meet transcriptions
- Automated meeting notes (Gemini, Otter, etc.)
- Manual summaries pasted by the PM
- Slack thread or channel link (handled via Optional: Slack context step above)

If multiple meetings provided for the same day: process together, produce a single unified digest.

### Optional: Gmail context
Invoke `.claude/sub/gmail-cbc-digest.md` with relevant search terms from the meeting as context. Cross-reference returned email and Confluence data with the meeting notes to catch supplementary decisions, action items, and Confluence page changes.
If the Gmail sub-routine returns no results, skip silently.

### 3. Extract per meeting
- **Action items** (explicit commitments - "I'll do X by Friday")
- **Decisions reached** (something concluded, even informally)
- **Blockers / risks surfaced**
- **Unresolved questions** (raised but unanswered)
- **Scope changes or new client asks** (anything that wasn't already in scope)

### 4. Deduplicate and consolidate
Merge identical or overlapping items. Flag contradictions across meetings explicitly:
> ⚠️ Contradiction: Meeting A confirmed X; Meeting C contradicted X. Needs resolution.

### 5. Prioritize all action items
- 🔴 **Today** - blocks others, blocks active build work, client waiting, cert deadline risk, unassigned items, unresolved decision blocking In Progress tickets
- 🟡 **This week** - needed before the next decision gate, client review, or cert submission window
- 🔵 **Backlog / FYI** - useful to know, not time-pressured

### 6. Produce the digest

### 6a. Save the digest
Save the digest as a .md file to `product-development/product/meetings/[Client]/meeting-notes/` using the naming convention `YYYY-MM-DD-meeting-digest.md`.

### 6b. Export to HTML
Run `python3 scripts/md-to-html.py <path-to-saved-file>` and report the HTML path to the user.

### 7. Cross-reference action items against the weekly tracker

Before offering Jira creation, find the current week's tracker file in `product-development/product/customers/accounts/[client]/weekly-trackers/` and read it.

For each extracted action item (all priorities), check whether it is already represented in:
- The Top Weekly Priorities table
- The Active Items sections
- The Follow-Up Schedule

Flag any action items that are NOT already in the tracker with: `[NOT IN TRACKER]`
Flag any that ARE already tracked with: `[IN TRACKER]` - these do not need to be added again.

Report the cross-reference summary to the user:
> "X action items from this meeting are already in your tracker. Y are new - I'll add the new ones to the tracker and offer Jira creation for Today items."

Add all `[NOT IN TRACKER]` items to the current week's tracker Follow-Up Schedule with the appropriate date. Do this silently and confirm at the end.

### 8. Offer Jira creation
After the digest and tracker cross-reference, generate a draft for each 🔴 Today action item that is NOT already in the tracker showing the proposed Jira task fields (summary, description, type, assignee, priority). Present the full draft in chat and ask: "Does this look right? I will create these in Jira once you confirm." Only create tickets after explicit user approval (yes / looks good / create it / equivalent). If Jira creation fails for any task, report the error to the user with the task details so they can retry manually.

---

## Output Format

```
# PM Daily Digest - [Project] - [Date]
> [X] meetings · [X] actions · [X] decisions · [X] blockers

## 🔴 Do Today
| # | Action | Owner | Context | Source |
|---|---|---|---|---|
| 1 | [Specific, executable action] | [Name or [UNASSIGNED]] | [Brief why] | [Meeting name] |

## 🟡 This Week
| # | Action | Owner | Context | Source |
|---|---|---|---|---|

## 🔵 Backlog / FYI
| # | Action | Owner | Context | Source |
|---|---|---|---|---|

## Decisions Made
| Decision | Made by | Meeting |
|---|---|---|

## Open Questions
| Question | Raised by | Needs answer from | Meeting |
|---|---|---|---|

## Risks & Flags
- 🚨 [Flag]: [Description] - PM action: [What you should do]

## Scope Alerts
- ⚠️ [Meeting]: [What was said that suggests scope creep] - PM action: [Confirm with client / push back / raise CR]

[Omit Scope Alerts section if none]
```

---

## Quality Rules

- **Every action item has an owner.** If unassigned, mark `[UNASSIGNED]` and auto-promote to 🔴 Today.
- **Actions are specific and executable.** Not "follow up on DRM" - "Email Axinom rep to confirm license server endpoint for staging."
- **Scope alerts are never buried.** A new client ask in a 60-minute meeting becomes the lead bullet under Scope Alerts.
- **Contradictions across meetings are flagged**, not silently reconciled.
- **Don't fabricate.** If a transcription is unclear, mark the item ambiguous rather than guess.
- **Decisions vs. action items:** a decision is concluded; an action is committed work. Don't conflate.
- **Cert / launch / contract dates** trigger automatic 🔴 promotion if mentioned.
- **Decisions blocking active build work** trigger automatic 🔴 - under AI codegen, an in-flight ticket waiting on a decision burns calendar time and produces wrong code if forced through.
- Keep the digest short enough to scan in 60 seconds. Detail belongs in the linked Jira tasks, not the digest.

---

> **Skill verification:** Please ensure that the skill meeting-helper.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
