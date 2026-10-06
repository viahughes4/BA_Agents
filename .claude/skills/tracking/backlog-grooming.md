---
name: backlog-grooming
description: Use this skill to run a backlog grooming session. Triggers on "backlog grooming", "groom the backlog", "backlog refinement", "refine the backlog", "review the backlog", "what tickets need attention", "prep for grooming", or any request to review and triage open backlog tickets. Automatically sweeps Jira for unreviewed and under-specified tickets - no manual ticket pasting required. Groups tickets into action buckets and produces a grooming agenda saved to the repo.
---

# Skill: Backlog Grooming

Automatically sweep Jira for tickets that need attention and produce a prioritized grooming agenda. No ticket pasting required - the skill queries Jira directly and evaluates each ticket against readiness criteria.

---

## What "ready" means for a client ticket

A ticket is grooming-ready when it has:
- A description with at least one clear acceptance criterion (testable, not vague)
- An assignee (or is explicitly in Vanessa's holding queue)
- A summary specific enough that a dev could start without a follow-up question
- No unresolved blocking dependencies

A ticket needs attention when it has:
- Empty or very thin description (under 150 characters of meaningful text)
- No acceptance criteria section ("acceptance criteria", "expected result", "AC:", "given/when/then")
- No assignee and not in the holding queue
- Summary mentions all platforms or multiple independent features (may need splitting)
- Created more than 21 days ago without any update (stale)
- Status is Blocked with no mitigation note

---

## Process

### 1. Load context

Read `.claude/sub/context-loader.md` to load Jira config (cloud ID, project key, sprint names).

client Jira config:
- Cloud ID: `d64a6473-85fd-4a3d-9ade-ee101f312717`
- Project: `CBC`
- API base: `https://api.atlassian.com/ex/jira/{cloud_id}/rest/api/3`
- Auth: Basic auth with `vanessa.hughes@accedo.tv` + `$JIRA_API_TOKEN`
- SSL: disable cert verification in Python (`ssl.CERT_NONE`)
- Active sprint: `July - Sept 2026`
- Backlog sprint: `Prioritized Backlog`

### 2. Run Jira sweeps

Run all three queries using the Python snippet below. Fetch up to 50 tickets per query with fields: `summary, description, status, assignee, priority, created, updated, subtasks, issuelinks, issuetype`.

**Query A - Prioritized backlog (unstarted):**
```
project = <jira.project_key> AND sprint = "Prioritized Backlog" AND status in (Open, "To Do", Backlog) ORDER BY priority ASC, created ASC
```

**Query B - Active sprint tickets with no assignee:**
```
project = <jira.project_key> AND sprint = "July - Sept 2026" AND assignee is EMPTY AND status not in (Done, Closed) ORDER BY priority ASC
```

**Query C - Stale open tickets (no update in 21+ days):**
```
project = <jira.project_key> AND status in (Open, "To Do", Backlog) AND updated <= -21d AND sprint not in (Done) ORDER BY updated ASC
```

Python API snippet:
```python
import urllib.request, base64, os, json, ssl, urllib.parse

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE
token = os.environ.get('JIRA_API_TOKEN', '')
creds = base64.b64encode(f'vanessa.hughes@accedo.tv:{token}'.encode()).decode()
cloud_id = 'd64a6473-85fd-4a3d-9ade-ee101f312717'
base = f'https://api.atlassian.com/ex/jira/{cloud_id}/rest/api/3'

def search(jql, fields='summary,description,status,assignee,priority,created,updated,subtasks,issuelinks,issuetype'):
    params = urllib.parse.urlencode({'jql': jql, 'fields': fields, 'maxResults': 50})
    req = urllib.request.Request(
        f'{base}/search?{params}',
        headers={'Authorization': f'Basic {creds}', 'Content-Type': 'application/json'}
    )
    resp = urllib.request.urlopen(req, context=ctx)
    return json.loads(resp.read())['issues']
```

### 3. Evaluate each ticket

For each ticket returned, extract the description text (strip ADF formatting), then evaluate against the readiness checklist:

**Extract description text from ADF:**
```python
def extract_text(node):
    if not node:
        return ''
    if node.get('type') == 'text':
        return node.get('text', '')
    return ' '.join(extract_text(c) for c in node.get('content', []))
```

**Readiness checks (run in order - a ticket only needs one failure to be flagged):**

| Check | Signal | Bucket |
|-------|--------|--------|
| Description missing or empty | `description` is null or `extract_text(description).strip()` is empty | Needs Description |
| No acceptance criteria | Description text does not contain any of: "acceptance criteria", "expected result", "AC:", "given", "when then", "should", "must" (case-insensitive) AND description text is under 200 chars | Needs AC |
| Thin description | Description text under 150 characters total (after stripping whitespace) | Needs Description |
| No assignee | `assignee` is null | Needs Assignee |
| Possibly too large | Summary contains "[All]" or "[all]" AND `subtasks` list is empty | Consider Splitting |
| Blocked with no note | Status name is "Blocked" AND `issuelinks` list is empty | Blocked - No Context |
| Stale | `updated` is more than 21 days before today | Stale - Check Priority |
| Passes all checks | None of the above triggered | Ready |

A ticket can only appear in one bucket - use the first failing check from the table above.

### 4. Deduplicate across queries

Queries A, B, and C may return overlapping tickets. Deduplicate by ticket key before bucketing - keep the first occurrence.

### 5. Load the weekly tracker for context

Read the most recent file in `<client.paths.root>weekly-trackers/`. Extract any tickets or items the PM flagged as needing attention this week. Cross-reference with the Jira results - if a ticket appears in both, add a note "(flagged in tracker)" next to it.

### 6. Build the grooming agenda

Group tickets into sections. Within each section, sort by priority (Critical first, then High, Medium, Low). Include ticket key, summary (truncated to 70 chars), priority, and assignee (or "Unassigned").

For tickets in "Needs Description" and "Needs AC" buckets, pull the first 100 characters of the description so the PM can see what exists without opening Jira.

### 7. Save the grooming session file

Save to:
`<client.paths.root>sprints/YYYY-MM-DD-backlog-grooming.md`

Run `python3 scripts/md-to-html.py` on the saved file and report the HTML path.

### 8. Report to user

Confirm:
- Total tickets reviewed
- Breakdown by bucket
- Top 3 tickets that most urgently need attention (Critical or High priority in Needs Description / Needs AC buckets)
- Any tickets flagged in the weekly tracker that were not found in the Jira sweep (may have been closed or moved)

---

## Output Template

```markdown
# client Backlog Grooming
Date: [YYYY-MM-DD]
Tickets reviewed: [N] | Queries: Prioritized Backlog + Active Sprint unassigned + Stale

---

## Action Required

### Needs Description ([N] tickets)
These tickets have empty or near-empty descriptions. Dev cannot start without more context.

| Ticket | Summary | Priority | Assignee | Current Description |
|--------|---------|----------|----------|---------------------|
| CBC-XXXX | [summary] | High | [name] | "[first 100 chars]" |

### Needs AC ([N] tickets)
These tickets have a description but no clear acceptance criteria.

| Ticket | Summary | Priority | Assignee | Current Description |
|--------|---------|----------|----------|---------------------|

### Needs Assignee ([N] tickets)
These tickets are not assigned and not in the Vanessa holding queue.

| Ticket | Summary | Priority | Status |
|--------|---------|----------|--------|

### Consider Splitting ([N] tickets)
These tickets cover all platforms with no sub-tasks - may be too large for a single sprint.

| Ticket | Summary | Priority | Assignee |
|--------|---------|----------|----------|

### Blocked - No Context ([N] tickets)
These tickets are blocked but have no linked blocking issue or explanation.

| Ticket | Summary | Priority | Assignee |
|--------|---------|----------|----------|

### Stale - Check Priority ([N] tickets)
These tickets have not been updated in 21+ days. Confirm they are still relevant.

| Ticket | Summary | Priority | Last Updated | Assignee |
|--------|---------|----------|-------------|----------|

---

## Ready ([N] tickets)
These tickets pass all readiness checks and are ready to pull into a sprint.

| Ticket | Summary | Priority | Assignee |
|--------|---------|----------|----------|

---

## Flagged in Tracker This Week
[List any tickets from the weekly tracker that appeared in the Jira sweep, with notes]

---

## Grooming Notes

[Leave blank - fill in during session]

## Decisions

[Leave blank - fill in during session]
```

---

## Rules

- No em dashes in any output. Use hyphens or colons instead.
- Never open Jira in the browser or ask the PM to paste tickets. All data comes from the API.
- If a query returns 0 results, say so explicitly in that section - do not skip it silently.
- If the JIRA_API_TOKEN environment variable is not set, stop and tell the user: "JIRA_API_TOKEN is not set. Run: export JIRA_API_TOKEN=your-token"
- Do not flag a ticket as "Needs AC" if its description is substantial (over 200 chars) but does not contain the exact keywords - the PM may use different formatting. Flag only thin descriptions.
- Tickets in the Vanessa holding queue (assigned to Vanessa Hughes, status Open) are intentionally unassigned to a developer - do not flag them as "Needs Assignee."
- The "Ready" bucket is the output the PM pulls from for sprint planning. Keep it clean - if in doubt, flag rather than mark ready.
- If the same ticket appears in Query A and Query C (stale backlog ticket), count it once in the most urgent bucket (Needs Description beats Stale).

---

> Skill verification: Please ensure that the skill backlog-grooming SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
