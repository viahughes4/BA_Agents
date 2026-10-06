---
name: dependency-radar
description: Surface all cross-team and external dependencies from Jira - blocked tickets, unresolved links, client-side blockers - and produce a prioritized dependency view. Use before sprint planning or PO syncs.
triggers:
  - "dependency radar"
  - "show dependencies"
  - "what are we blocked on"
  - "dependency report"
  - "cross-team blockers"
  - "external dependencies"
  - "what's blocking us"
---

# Dependency Radar Skill

## Purpose
Produce a clear, prioritized view of all active dependencies so the PM can act on them before they block delivery.

## Steps

### 1. Load config
Read `.claude/config.yml` and get `jira.project_key`.

### 2. Find blocked and blocking tickets
Run these JQL queries:

**Blocked tickets** (things we're waiting on):
`project = [PROJECT_KEY] AND issueFunction in linkedIssuesOf("project != [PROJECT_KEY]", "is blocked by") AND status != Done`

**External blockers** (tickets linked to other projects):
`project = [PROJECT_KEY] AND issue in linkedIssues("project != [PROJECT_KEY]") AND status != Done`

**Tickets with "blocked" or "waiting" labels:**
`project = [PROJECT_KEY] AND labels in ("blocked", "waiting-on-client", "waiting-on-third-party", "dependency") AND status != Done`

### 3. Categorize each dependency
For each ticket found, categorize:
- **Client-side**: waiting on client decision, approval, or asset
- **Third-party**: waiting on external vendor, API, or service
- **Internal - other team**: waiting on another Accedo team
- **Internal - same team**: blocked by another ticket in this project

### 4. Prioritize
Score each dependency:
- In current sprint: HIGH
- In next sprint: MEDIUM  
- Backlog: LOW
- Has due date within 14 days: +1 priority level

### 5. Output format

```
## Dependency Radar
**Project:** [PROJECT_KEY]  
**Generated:** [date]

### High Priority (in current sprint)
| Ticket | Summary | Blocked By | Type | Age | Owner |
|--------|---------|-----------|------|-----|-------|
| [KEY-123] | Feature X | Client approval on wireframes | Client-side | 8d | @dev |

### Medium Priority (next sprint)
...

### Low Priority (backlog)
...

### Summary
- [N] total dependencies tracked
- [N] client-side (action required: flag in next PO sync)
- [N] third-party
- [N] internal
- [N] have been blocked 7+ days ⚠️
```

### 6. Check weekly tracker
Read the current week's tracker. For any HIGH priority dependency not already tracked, offer to add it.

### 7. Offer next steps
Ask if the user wants to:
- Add untracked HIGH blockers to the weekly tracker
- Add high-risk dependencies to the risk register
- Prep these for the PO sync (`/po-sync-prep`)
