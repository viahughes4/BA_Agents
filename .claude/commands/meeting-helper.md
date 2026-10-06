# Skill: Meeting Helper

Process meeting transcriptions, automated notes, or summaries into a prioritized PM action digest. Optionally create Jira tasks for "Today" actions.

---

## Process

### 1. Load context
Invoke `/sub:context-loader`. Use project name, client, Jira key, and active phase to ground the digest.

### 2. Accept input
- Single or multiple Google Meet transcriptions
- Automated meeting notes (Gemini, Otter, etc.)
- Manual summaries pasted by the PM

If multiple meetings provided for the same day: process together, produce a single unified digest.

### 3. Extract per meeting
- **Action items** (explicit commitments — "I'll do X by Friday")
- **Decisions reached** (something concluded, even informally)
- **Blockers / risks surfaced**
- **Unresolved questions** (raised but unanswered)
- **Scope changes or new client asks** (anything that wasn't already in scope)

### 4. Deduplicate and consolidate
Merge identical or overlapping items. Flag contradictions across meetings explicitly:
> ⚠️ Contradiction: Meeting A confirmed X; Meeting C contradicted X. Needs resolution.

### 5. Prioritize all action items
- 🔴 **Today** — blocks others, blocks active build work, client waiting, cert deadline risk, unassigned items, unresolved decision blocking In Progress tickets
- 🟡 **This week** — needed before the next decision gate, client review, or cert submission window
- 🔵 **Backlog / FYI** — useful to know, not time-pressured

### 6. Produce the digest

### 7. Offer Jira creation
After the digest:
> Would you like me to create Jira tasks for the 🔴 Today items? Reply `yes` to invoke `/sub:jira-write`, `no` to skip.

---

## Output Format

```
# PM Daily Digest — [Project] — [Date]
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
- 🚨 [Flag]: [Description] — PM action: [What you should do]

## Scope Alerts
- ⚠️ [Meeting]: [What was said that suggests scope creep] — PM action: [Confirm with client / push back / raise CR]

[Omit Scope Alerts section if none]
```

---

## Quality Rules

- **Every action item has an owner.** If unassigned, mark `[UNASSIGNED]` and auto-promote to 🔴 Today.
- **Actions are specific and executable.** Not "follow up on DRM" — "Email Axinom rep to confirm license server endpoint for staging."
- **Scope alerts are never buried.** A new client ask in a 60-minute meeting becomes the lead bullet under Scope Alerts.
- **Contradictions across meetings are flagged**, not silently reconciled.
- **Don't fabricate.** If a transcription is unclear, mark the item ambiguous rather than guess.
- **Decisions vs. action items:** a decision is concluded; an action is committed work. Don't conflate.
- **Cert / launch / contract dates** trigger automatic 🔴 promotion if mentioned.
- **Decisions blocking active build work** trigger automatic 🔴 — under AI codegen, an in-flight ticket waiting on a decision burns calendar time and produces wrong code if forced through.
- Keep the digest short enough to scan in 60 seconds. Detail belongs in the linked Jira tasks, not the digest.
