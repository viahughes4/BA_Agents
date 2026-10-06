# Skill: Status Report

Generate one of four reports. Audience-aware. Optionally publish to Confluence.

| # | Report | Audience |
|---|---|---|
| 1 | Delivery Snapshot *(or Sprint Report when Delivery Cadence = Sprint-based)* | Internal team + PM |
| 2 | Weekly Client Summary | Client + PM |
| 3 | Executive Summary | Client leadership |
| 4 | Release Readiness | PM + QA + Dev leads |

The standalone Risk Register has been folded into the Risks subsection of Delivery Snapshot / Sprint Report. Risk-only reports are produced via the internal report; no separate template.

Universal output rules apply — see CLAUDE.md → OUTPUT STYLE.

---

## Process

### 1. Load context
Invoke `/sub:context-loader` for project name, client, Jira key, Confluence space, platform scope, and Delivery Cadence.

### 2. Identify report type
If the PM hasn't named the type, infer from `project-context.md` Delivery Cadence and the audience implied by the request. Ask only if genuinely ambiguous:
> Which report?
> 1. [Delivery Snapshot | Sprint Report — based on cadence] (internal)
> 2. Weekly Client Summary
> 3. Executive Summary
> 4. Release Readiness

Default the internal option to **Sprint Report** when Delivery Cadence is Sprint-based with optional Agile fields populated; otherwise **Delivery Snapshot**.

### 3. Pull data
- Jira MCP connected: invoke `/sub:jira-read`. For Delivery Snapshot use `JQL project = <KEY> AND statusCategory != Done` plus `JQL resolved >= -<period>` for the period covered. For Sprint Report use `SPRINT current <KEY>`.
- Not connected: ask the PM to paste board content and any QA/cert/risk inputs.

### 4. Generate using the appropriate template (below)

### 5. Offer publication
After producing the report:
> Publish to Confluence space `[KEY]` under `[parent page]`? Reply `confirm` to publish via `/sub:confluence-write`, or `cancel` to keep local.

If confirmed, invoke `/sub:confluence-write` with the formatted page.

---

## Templates

### 1. Delivery Snapshot — internal team + PM *(default)*

```
# Delivery Snapshot — [Project] — [Period]

## Headline
[One sentence: are we on track, what's the dominant signal]

## Shipped This Period
| ID | Title | Platform(s) | PR / Build | Verified |
|---|---|---|---|---|
[Items moved to Done in the period. "Verified" = AC test pass / QA sign-off captured.]

## In Flight
| ID | Title | State | Days in state | Owner |
|---|---|---|---|---|
[Active items. Flag any > 7 days in `In Progress`.]

## Decision Health
| Open decisions | Resolved this period | Avg age (days) | Oldest |
|---|---|---|---|
| N | M | X | [decision title — N days old] |

[List the top 3 unresolved decisions blocking active work, with owner and needed-by date.]

## External Dependencies
| Dependency | Owner | Status | Risk to schedule |
|---|---|---|---|
[Cert submissions, content rights, vendor deliveries, hardware availability — anything outside the team's control that gates a milestone.]

## Risks (delta from last snapshot)
| Risk | Likelihood | Impact | Trend | Mitigation | Owner |
|---|---|---|---|---|---|

## Assumptions Invalidated
- [Anything we believed at the start of the period that turned out wrong]

## Next Period
- [Top outcomes targeted]
- [Decisions that must close before [date]]
- [Open DECISION REQUIRED]
  ⚑ ...
```

### 2. Weekly Client Summary — client + PM

```
# Weekly Update — [Project] — Week of [Date]

**Overall Status:** 🟢 / 🟡 / 🔴
[One-sentence headline]

## This Week
- [Client-readable bullet — no Jira IDs]

## Next Week
- [Client-readable bullet]

## Milestone Health
| Milestone | Target | Status | Notes |
|---|---|---|---|

## Risks / Issues Needing Client Action
- [Only items the client must act on or be aware of]

## Open Decisions (awaiting client input)
- [Decision] — [needed by] — [why it blocks delivery]
```

### 3. Executive Summary — client leadership

```
# Executive Summary — [Project] — [Period]

**Status:** 🟢 / 🟡 / 🔴
**Headline:** [One sentence]

## Outcomes
- [Business-level outcome bullets, not feature lists]

## Trajectory
[2–3 sentences on whether we are tracking to launch / KPI targets, with the key driver]

## Risks
- [Top 1–3 risks, business-impact framed]

## Asks
- [What leadership needs to do, if anything]
```

### 4. Release Readiness — PM + QA + Dev leads

```
# Release Readiness — [Release / Version] — [Date]

## Release Scope
[Summary of features in scope]

## Quality Gates
| Gate | Threshold | Current | Status |
|---|---|---|---|
| P1 bugs | 0 | X | ✅/🚫 |
| P2 bugs | < N | X | ✅/⚠️ |
| AC coverage in tests | 100% of release-scope ACs have an automated or manual test on file | X% | ✅/⚠️ |
| Regression suite | 100% pass | X% | ✅/⚠️ |
| Cert submissions | All platforms submitted | [list] | ✅/⚠️ |
| Accessibility | WCAG 2.1 AA / platform a11y | X | ✅/⚠️ |
| Performance benchmarks | [Per platform SLA] | X | ✅/⚠️ |
| DRM smoke tests | All DRM types pass | X | ✅/⚠️ |
| Analytics events validated | All named events firing with correct params | X | ✅/⚠️ |
| Open decisions | 0 in release scope | X | ✅/🚫 |
| Rollback plan | Documented + tested | X | ✅/⚠️ |

## Known Issues Shipping
| ID | Severity | Description | Workaround | Plan |
|---|---|---|---|---|

## Certification Status
| Platform | Submitted | Status | Notes |
|---|---|---|---|

## Go / No-Go Recommendation
**Recommendation:** GO / NO-GO / CONDITIONAL
**Rationale:** [Tied to gate results above. Be specific.]
**Conditions (if conditional):** [...]
```

### 5. Sprint Report — internal team + PM *(only when sprint-based)*

```
# Sprint Report — [Sprint Name] — [Date]

## Sprint Goal
[The committed goal — and a candid assessment of whether it was met]

## Completed Stories
| ID | Title | Platform(s) | Notes |
|---|---|---|---|

## Carryover
| ID | Title | Reason | Action |
|---|---|---|---|

## Blockers & Escalations
- [Blocker]: [owner, age, escalation needed Y/N]

## Decision Health
[Same format as Delivery Snapshot]

## Risks (updated)
| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|

## Assumptions Invalidated This Sprint
- [...]

## Next Sprint Preview
- Goal: [...]
- Key dependencies: [...]
- Open DECISION REQUIRED before next sprint starts:
  ⚑ ...
```

*Story-points columns are intentionally omitted from the default Sprint Report. If the engagement requires points reporting (`project-context.md` Delivery Cadence → Traditional Agile fields populated), add a Points column to Completed Stories and Carryover tables.*

---

## Quality Rules

- **Status indicators are earned, not aspirational.** 🟢 means tracking; 🟡 means risk; 🔴 means blocker. Don't say 🟢 if a P1 is open or a release-scope decision is unresolved.
- **Decision health is a first-class signal.** Under AI codegen, unresolved decisions are the dominant delivery risk — surface them prominently in every internal-facing report.
- **Risks are surfaced, not buried.** If the project is yellow because of one persistent blocker, lead with it.
- **Client-facing reports translate tech to business impact.** Don't write "Widevine license server returning 403s on Tizen" to a CMO — write "Samsung TV playback intermittently failing; engineering identified root cause, fix in QA."
- **Never mark a release Ready** with open P1 / P2 bugs, unsubmitted certs, or unresolved release-scope decisions. Recommendation must be NO-GO or CONDITIONAL with stated conditions.
- **No Jira IDs in client-facing reports** unless the client uses the same Jira instance.
- **Default to Delivery Snapshot.** Only produce a Sprint Report when the project is explicitly running sprints. Don't manufacture sprint structure on continuous-delivery engagements.
- Always include the report date and the period covered.
- Offer Confluence publication every time, but never auto-publish.
