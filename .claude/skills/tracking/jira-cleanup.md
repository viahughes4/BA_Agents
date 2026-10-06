# Skill: Jira Cleanup

Jira board health analysis. Read-only analysis phase, then proposed write actions gated on PM approval.

Under AI-assisted code generation, this skill optimises for **spec readiness, decision resolution, and traceability** rather than sprint-commitment hygiene. A "clean board" is one where every active ticket is buildable - not one where points sum cleanly to velocity.

---

## Triggers

- clean up Jira
- Jira cleanup
- Jira hygiene
- board health
- audit Jira
- Jira health check
- clean the board

---

## Process

### 1. Load context
Invoke `.claude/sub/context-loader.md` to confirm Jira project key, platform scope, and delivery cadence.

**Use the project key returned by context-loader (expected: CBC) for all JQL queries and ticket references. Do not use placeholder `<KEY>` or `[PROJ]` in final output.**

### 2. Pull data
- If Jira MCP connected: invoke `.claude/sub/jira-project-snapshot.md` to pull the active backlog, in-progress, and recently closed tickets.
- If not connected: ask the PM to paste an export including ID, Title, Status, Assignee, Description, ACs, Labels, Parent Epic, Priority, Sprint (if used).

### 3. Run all checks
Story-level, epic-level, flow-level. Categorize each finding **Critical / Warning / Observation.**

### 4. Produce health report
Sections: Workset Snapshot - Issues by Severity - Patterns - Proposed Actions.

Save the report to: `product-development/product/customers/accounts/[client]/health-reports/YYYY-MM-DD-jira-cleanup.md`

Then run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/health-reports/YYYY-MM-DD-jira-cleanup.md`

### 5. Propose actions
List each action as a numbered item referencing specific ticket IDs and the exact change.

### 6. Approval gate
Ask:
> Shall I execute these via `.claude/sub/jira-project-snapshot.md`? Reply with the action numbers to apply (e.g. `1, 3-5, 8`), `all`, or `cancel`.

### 7. Execute approved actions
For each, invoke `.claude/sub/jira-project-snapshot.md` with the proposed change. Capture results.

If Jira MCP becomes unavailable during execution, stop, record which actions completed and which did not, report to PM, and do not retry automatically.

---

## Story-Level Checks *(spec readiness for AI codegen)*

| Severity | Check |
|---|---|
| 🚫 Critical | Missing ACs entirely |
| 🚫 Critical | In Progress with unresolved DECISION REQUIRED in description/comments |
| 🚫 Critical | In Progress with no testable AC (vague Thens: "works correctly", "handles gracefully") |
| 🚫 Critical | In Progress with no platform label |
| 🚫 Critical | In Progress with no parent epic (excluding spike/chore) |
| 🚫 Critical | In Progress with no assignee |
| 🚫 Critical | Blocked status with no documented blocker (link or comment) |
| 🚫 Critical | Playback story missing player SDK or DRM provider |
| 🚫 Critical | System-boundary story (auth/playback/payment/analytics) with no integration contract reference |
| ⚠️ Warning | ACs as bullets, not Given/When/Then |
| ⚠️ Warning | Vague one-line description |
| ⚠️ Warning | Story spans multiple decision boundaries or platforms (clarity, not size - recommend decomposition) |
| ⚠️ Warning | Bilingual project: UI story without l10n AC |
| ⚠️ Warning | Dependencies not documented (link or note) |
| ⚠️ Warning | Stale `In Progress` (> 7 days with no comment / status change - tunable per project) |
| ℹ️ Observation | Done with no PR / build / test-run link captured |
| ℹ️ Observation | Vague title (e.g. "fix bug", "playback work") |

## Epic-Level Checks

| Severity | Check |
|---|---|
| 🚫 Critical | Active epic with no linked stories |
| 🚫 Critical | Epic with no traceable parent feature, PRD section, or SOW reference |
| ⚠️ Warning | No description / no goal statement |
| ⚠️ Warning | No platform scope listed |
| ℹ️ Observation | Mixed unrelated stories under one epic |

## Flow-Level Checks *(replaces sprint-level checks)*

| Severity | Check |
|---|---|
| 🚫 Critical | Decision-blocked queue: stories tagged with unresolved DECISION REQUIRED that are sitting in active states (anything past Backlog) |
| ⚠️ Warning | High WIP: more `In Progress` than the project's stated WIP limit (or > 1 per active contributor if no limit set) |
| ⚠️ Warning | Stale workset: > 20% of `In Progress` items haven't moved in 7+ days |
| ⚠️ Warning | Untraceable Done: > 20% of recently Done items have no PR / build link captured |
| ℹ️ Observation | No epic-level goal or outcome statement on the active epics |

*Sprint-specific checks (velocity over-commitment, carryover %, sprint goal) only run if `project-foundation.md` Delivery Cadence is set to Sprint-based and the optional Agile fields are populated. Otherwise skip.*

---

## Output Format

```
## Jira Board Health Report - client - [Date]

### Workset Snapshot
- Active backlog: [N items]
- In Progress: [N items] (assigned to [M contributors])
- Blocked: [N items]
- Decision-blocked (unresolved DECISION REQUIRED in active state): [N items]
- Recently Done (14d): [N items]

### Issues Found

#### 🚫 Critical
- [<jira.project_key>-123]: Missing ACs. Recommend: add ACs covering happy/edge/error.
- [<jira.project_key>-145]: Playback story In Progress, no DRM provider named. Recommend: comment requesting Widevine vs FairPlay confirmation; block until resolved.
- [<jira.project_key>-167]: In Progress with unresolved DECISION REQUIRED on auth method. Recommend: revert to Backlog until decision is logged.

#### ⚠️ Warning
- [<jira.project_key>-128]: ACs in bullet form, not GWT. Recommend: rewrite using GWT.
- [<jira.project_key>-204]: Spans iOS + Android + Web in a single ticket. Recommend: decompose to one ticket per platform.

#### ℹ️ Observation
- [<jira.project_key>-150]: Vague title "playback work". Recommend: rename for clarity.

### Patterns
- [e.g. "12 of 18 stories missing platform labels - likely a board-wide gap, not isolated."]
- [e.g. "5 of 8 In Progress items have no PR linked - traceability gap, not just one ticket."]

### Proposed Actions (Pending PM Approval)
1. [<jira.project_key>-123] - Add ACs (draft below). Type: Update description.
2. [<jira.project_key>-128] - Rewrite ACs in GWT format (draft below). Type: Update description.
3. Bulk: Add label `platform-ios` to [list of 12 IDs]. Type: Bulk label update.
4. [<jira.project_key>-145] - Comment requesting DRM provider confirmation from architect; transition to Blocked. Type: Add comment + transition.
5. [<jira.project_key>-167] - Transition back to Backlog with comment naming the unresolved decision. Type: Transition + comment.

### Escalation Items
- [Decision-blocked items needing PM-level intervention beyond Jira edits - scope decisions, vendor selections, missing architecture decisions. Reference the open decision log.]

---

Shall I execute these? Reply with action numbers (e.g. `1, 3-5`), `all`, or `cancel`.
```

---

## Quality Rules

- Every proposed action references a specific ticket ID (or list of IDs for bulk).
- Every proposed action specifies the exact field change - no "improve this story."
- Bulk actions are batched (e.g. labels) rather than presented as 12 individual proposals.
- Critical findings are never softened.
- For playback / DRM / cert findings, surface the relevant system boundary context loaded via `.claude/sub/context-loader.md` to make the gap visible.
- **Decision-blocked stories in active states are always Critical.** Under AI codegen these produce wrong code in hours - the right action is to revert to Backlog until the decision is logged, not to push through.
- **Do not flag stories as "too big" by point count.** If a story mixes decision boundaries or platforms, flag for decomposition (a clarity issue), not for resizing.
- **Sprint-commitment checks only fire when sprint-based delivery is configured.** Don't manufacture sprint findings on continuous-delivery projects.
- Never auto-execute. Always wait for explicit approval.

---

> **Skill verification:** Please ensure that the jira-cleanup skill (SKILL.md) was actually invoked. If not, run it from the top.
