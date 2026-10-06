---
name: sprint-velocity
description: Pull Jira sprint velocity data for the last N sprints, show trend, and surface any outlier sprints with explanation. Use for estimate reviews, planning, and team health checks.
triggers:
  - "sprint velocity"
  - "velocity report"
  - "how fast are we moving"
  - "what's our velocity"
  - "velocity trend"
  - "estimate review"
---

# Sprint Velocity Skill

## Purpose
Surface historical sprint velocity to inform planning and estimate reviews.

## Steps

### 1. Load config
Read `.claude/config.yml` and get `jira.project_key` and `jira.board_id`.

### 2. Get completed sprints
Use `searchJiraIssuesUsingJql` to find completed stories from the last 5 sprints:
JQL: `project = [PROJECT_KEY] AND sprint in closedSprints() AND status = Done AND issuetype in (Story, Task) ORDER BY sprint DESC`

Group results by sprint name. For each sprint, sum story points completed.

### 3. Calculate metrics
- Velocity per sprint (story points completed)
- Average velocity (last 3 sprints)
- Trend: compare last sprint to 3-sprint average - Accelerating (>10% above avg), Stable (within 10%), Decelerating (>10% below avg)
- Flag outlier sprints: any sprint more than 30% above or below the average. For outliers, note likely causes from ticket data (e.g. many bugs, unplanned work, partial sprint).

### 4. Output format

```
## Sprint Velocity Report
**Project:** [PROJECT_KEY]  
**Generated:** [date]

| Sprint | Points Completed | vs. Avg | Notes |
|--------|-----------------|---------|-------|
| Sprint 24 | 42 | +8% | - |
| Sprint 23 | 38 | -2% | - |
| Sprint 22 | 51 | +30% ⚠️ | High output - 3 carry-over stories from Sprint 21 |
| Sprint 21 | 28 | -28% ⚠️ | Below avg - 2 devs on PTO, release freeze mid-sprint |
| Sprint 20 | 39 | - | baseline |

**3-Sprint Average:** 43 points  
**Trend:** Stable

### Recommendation
[1-2 sentences: safe range for next sprint commitment, flag if trend is concerning]
```

### 5. Offer next steps
Ask if the user wants to:
- Update the weekly tracker with current velocity
- Use this data in an estimate review (`/estimate-review`)
- Add a velocity risk to the risk register if trend is Decelerating
