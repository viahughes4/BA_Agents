---
name: backlog-health
description: On-demand backlog sweep that queries Jira, classifies tickets against the client's North Star goals, flags stuck tickets, updates the roadmap, and sends a Slack DM report. Triggers on "run backlog health", "backlog sweep", "sweep the backlog", "what's stuck", "backlog north star", "health check", or any request to audit backlog tickets against priorities.
version: 1.0.0
---

# Skill: Backlog Health + North Star Sweep

Sweep configured Jira sprints, classify every ticket against the client's North Star goals, flag stuck tickets, update the roadmap, and send a DM summary. Run on demand whenever you want a full picture of what's in the queue and whether it's moving.

---

## Process

### 1. Load context

Read `.claude/sub/context-loader.md` to load project foundation (Jira key, team roster, folder paths). Load `.claude/config.yml` for Jira instance and Slack PM DM ID.

---

### 2. Query Jira sprints

Use the Jira API token from config or environment. Query ALL of the following sprints:

```
project = <jira.project_key> AND sprint in ("Prioritized Backlog", "July - Sept 2026", "Design UI Issues", "Accessibility", "Oct - Dec BE Prioritization") ORDER BY priority ASC, updated ASC
```

Also query for dev-assigned open tickets:
```
project = <jira.project_key> AND assignee in ("62aa6e86192edb006f9fd211","5f2d6dd2c9c094001cb6baf4","5e0f70ef0242870e996ec1a8") AND statusCategory != Done ORDER BY updated ASC
```

(Andy, Eduardo, Dario)

Collect all results. Deduplicate by ticket key.

---

### 3. Classify every ticket by North Star

For each ticket, read the summary and classify using these keyword rules:

**👥 Goal 1 - User Retention & Growth:**
Keywords: `auth`, `sign in`, `sign-in`, `login`, `CMP`, `consent`, `subscription`, `RDI`, `TTS`, `A11Y`, `accessibility`, `screen reader`, `voice guide`, `voice view`, `talkback`, `voiceover`, `profile`, `account`, `registration`, `SSO`, `token`, `refresh`

**▶️ Goal 2 - Content Discovery:**
Keywords: `search`, `in-app messaging`, `homepage`, `pagination`, `promo banner`, `carousel`, `browse`, `discovery`, `recommendation`, `swimlane`, `continue watching`, `my lineup`, `watchlist`, `favourite`, `see more`, `voir plus`, `section`

**📺 Goal 3 - Deeper Engagement:**
Keywords: `playback`, `player`, `live`, `trickplay`, `CC`, `closed caption`, `seek`, `chain play`, `DVR`, `PVR`, `pre-roll`, `mid-roll`, `ads`, `ad`, `volume`, `thumbnail`, `scrub`, `stream`, `bitrate`, `buffer`

**🔧 Maintenance:**
Keywords: `SDK`, `icon`, `SVG`, `logo`, `font`, `spacing`, `alignment`, `refactor`, `migration`, `dependency`, `build`, `CI`, `deploy`, `script`, `cleanup`, `Lotame`, `rebranding`

If a ticket matches multiple goals, assign to the most specific match. If no keyword matches, flag as `⚠️ No North Star mapping`.

---

### 4. Flag stuck tickets

A ticket is **stuck** if ANY of the following apply:

- Status is `Open` AND the ticket has been in the sprint for **30+ days** with no assignee
- Status is `Open` AND the ticket has been in the sprint for **60+ days** regardless of assignee
- Status is `In Progress` AND `updated` date is **14+ days ago** (no movement)
- Status is `Blocked` (always flag - needs attention)
- Sprint is `Design UI Issues` or `Accessibility` AND status has not changed in **21+ days**

For stuck tickets, note: ticket key, summary, days stuck, current assignee, sprint.

---

### 5. Build the North Star report

Organize findings into this structure:

```
## Backlog Health Report — [DATE]

### North Star Distribution
Goal 1 (Retention): [N] tickets — [list open ones]
Goal 2 (Discovery): [N] tickets — [list open ones]
Goal 3 (Engagement): [N] tickets — [list open ones]
Maintenance: [N] tickets
No mapping: [N] tickets — ⚠️ needs classification

### Stuck Tickets — Action Required
[For each stuck ticket:]
- CBC-XXXX | [Sprint] | [Summary] | [Days stuck] | [Assignee or Unassigned]

### Dev Queue Summary
Andy: [N] open | [N] in progress | [N] in code review
Eduardo: [N] open | [N] in progress | [N] in code review
Dario: [N] open | [N] in progress | [N] in code review

### Goal Coverage Gaps
[Any goal with zero active tickets gets flagged]
[Any sprint with 5+ stuck tickets gets flagged]

### Suggested Next Assignments
[For unassigned open tickets that are not stuck, suggest based on:]
- Eduardo: playback, content discovery, TTS/A11Y
- Andy: auth, CMP/consent, live/linear
- Dario: in-app messaging, player bugs, platform fixes
[Only suggest if ticket is unassigned AND not blocked]
```

---

### 6. Update the roadmap

Load the roadmap file:
`<client.paths.root>roadmap/roadmap.md`

For every ticket key found in the Jira results that also appears in the roadmap:
- Compare the Jira status to the roadmap status
- If they differ, update the roadmap row to match Jira
- Append `(synced [DATE])` to the status cell

After updating, push via GitHub API:
```bash
python3 << 'PYEOF'
import subprocess, json, base64
repo = "viahughes4/accedo-cbc-PM"
branch = "main"
roadmap_path = "<client.paths.root>roadmap/roadmap.md"
r = subprocess.run(['gh','api',f'repos/{repo}/contents/{roadmap_path}?ref={branch}','--jq','.sha'], capture_output=True, text=True)
sha = r.stdout.strip()
with open('/tmp/roadmap_updated.md','rb') as f: encoded = base64.b64encode(f.read()).decode()
payload = {'message': f'Auto-sync: roadmap Jira sweep [DATE]', 'content': encoded, 'sha': sha, 'branch': branch}
with open('/tmp/roadmap_payload.json','w') as f: json.dump(payload, f)
r = subprocess.run(['gh','api',f'repos/{repo}/contents/{roadmap_path}','--method','PUT','--input','/tmp/roadmap_payload.json'], capture_output=True, text=True)
print('Roadmap: SUCCESS' if r.returncode == 0 else f'FAILED: {r.stderr[:80]}')
PYEOF
```

Also run: `python3 scripts/md-to-html.py <client.paths.root>roadmap/roadmap.md`

---

### 7. Save markdown report

Save to:
`<client.paths.root>health-reports/YYYY-MM-DD-backlog-health.md`

Content: the full North Star report from Step 5.

Run `python3 scripts/md-to-html.py` on it.

Commit and push:
```bash
git add <client.paths.root> && git commit -m "backlog health: Jira sweep [DATE]" && git push origin main
```

---

### 8. Suggest roadmap additions (requires PM approval)

After building the health report, identify unassigned open tickets that:
- Have a clear North Star mapping
- Are not blocked
- Are not already in any dev's roadmap section

For each, draft a suggested roadmap entry:

```
Suggested addition to [Dev]'s roadmap:
| [CBC-XXXX] | [Summary truncated 60 chars] | [Suggested week/month] | ⬜ Planned | [ticket key] | [North Star goal] | [one-line why it matters] |
```

Base dev suggestion on:
- Eduardo: playback, content discovery, A11Y/TTS
- Andy: auth, CMP/consent, live/linear, trickplay
- Dario: in-app messaging, platform bugs, player fixes

Present ALL suggestions in chat and ask:
> "I found [N] tickets to suggest adding to the roadmap. Review each and say 'add all', 'skip [key]', or 'add [key] to [dev]' to customize. I won't update the roadmap until you confirm."

Only add to the roadmap after explicit confirmation. Add to the correct dev's section in the Q2/Q3 tables - never create new sections.

---

### 9. DM Vanessa

Send to D03DQP5VANS. Max 20 lines. No em dashes. Use this format:

```
*Backlog Health — [DATE]*

*Stuck tickets ([N] total):*
- [CBC-XXXX] [Sprint] — [summary truncated 50 chars] — [N days]
[up to 5 most urgent]

*North Star gaps:*
[Any goal with 0 active tickets]
[Any goal with all tickets blocked]

*No North Star mapping ([N] tickets):*
[List ticket keys only — these need classification]

*Dev queues:*
Andy: [N] active | Eduardo: [N] active | Dario: [N] active

*Suggested next assignments:*
[Up to 3 unassigned tickets with recommended dev]

Full report: health-reports/[filename].md
```

---

## Rules

- Only suggest assignments for unassigned tickets. Never suggest reassigning a ticket that has an owner.
- Always use real Jira data. Never invent statuses.
- If Jira is unavailable, DM Vanessa: "Backlog health skipped - Jira unavailable. Try again."
- Roadmap update is silent - only report if it fails.
- No em dashes anywhere in output.
- Do not create new roadmap sections. Only update existing rows.
- Stuck ticket detection uses `updated` field from Jira as the staleness signal.

---

> **Skill verification:** Ensure backlog-health SKILL.md was actually run. If not, run from the top.
