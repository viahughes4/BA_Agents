# Skill: Estimate Review

Judge whether story estimates are accurate against the catalog's **Typical Effort** ranges. Nothing else.

This skill does **not** do sprint capacity planning, load balancing, or story splitting. Those belong in `/jira-cleanup`, `/feature-discovery`, or `/generate-stories`.

---

## Files

- **Effort benchmarks:** `OTT-requirements-reference.md` → *Feature Catalog*, `Typical Effort (days)` column. Ranges are **per-platform, single-platform**.
- **Project specifics:** `project-context.md` → *Platform Launch Plan* (to validate platform tags on stories).
- **Story input:** `/sub:jira-read` if Jira MCP available, else PM-supplied list.

---

## Process

### 1. Load context
Invoke `/sub:context-loader` to confirm platform scope. If `project-context.md` is missing, proceed — but flag every story whose platform isn't validated against a known project platform.

### 2. Accept input
- `/sub:jira-read` for the sprint or query the PM specifies, **or**
- A PM-supplied list. Each story must have: ID, title, estimate (with units), platform(s).

### 3. Normalize estimates
- Detect units: days vs. story points. If story points, ask the PM for the project's points-to-days conversion before proceeding. Do not guess.
- Reject (mark 🚫 Unverifiable) any story that spans multiple platforms in a single ticket — multi-platform aggregation is unreliable and the right answer is to split the ticket. Do not attempt to multiply ranges.

### 4. Per-story analysis
For each story:
1. Invoke `/sub:catalog-match MATCH "<story title + key AC phrases>" WITH_EFFORT`. Record the returned **Confidence** (High / Medium / Low / None) and **Typical Effort** range verbatim — confidence vocabulary is owned by the sub-agent, not redefined here.
2. Compare estimate to the range returned by the sub-agent and classify:
   - ✅ **In range**
   - ⚠️ **Low** — below lower bound. Likely under-scoped; surface as a scope-debt risk.
   - ⚠️ **High** — above upper bound. Surface for refinement.
   - 🚫 **Missing** — no estimate.
   - 🚫 **Unverifiable** — no platform, multi-platform ticket, or sub-agent returned Confidence: None.

### 5. Produce report

---

## Output Format

```
## Estimate Review — [Project] — [Date]

### Verdict Table
| ID | Story | Catalog Feature | Match | Platform | Estimate | Catalog Range | Verdict |
|---|---|---|---|---|---|---|---|
| PROJ-101 | Player UI | Video Playback › Video Player | High | Samsung Tizen | 3d | 2–10d | ✅ In range |
| PROJ-102 | EPG grid | EPG › EPG | High | LG webOS | 12d | 2–3d | ⚠️ High |
| PROJ-103 | Sign in | Customer Mgmt › Auth — D2C | High | Web | — | 2–3d | 🚫 Missing |
| PROJ-104 | Search | Content Discovery › Search | High | (none) | 5d | 2–4d | 🚫 Unverifiable |
| PROJ-105 | Onboarding wizard | (no match) | None | iOS | 4d | — | 🚫 Unverifiable |
| PROJ-106 | Favourites | Personalization › Favourites | High | Android | 0.5d | 2–3d | ⚠️ Low |

### Outliers — Reasoning Required
For each ⚠️ High or ⚠️ Low, state the deviation and the most likely explanations the PM should test in refinement.

- **PROJ-102 — 12d vs 2–3d (4× upper bound).** Likely drivers: live data layer is new build rather than reuse, Lightning rendering perf work, timezone/region handling. Confirm in refinement; if all three apply, the estimate may be correct and the *ticket* is the problem (too broad).
- **PROJ-106 — 0.5d vs 2–3d.** Under-scoped. Common miss: sync across devices, offline behaviour, server-side persistence. Confirm whether AC covers these.

### Blockers
- **Missing estimates:** PROJ-103. Cannot commit without an estimate.
- **Unverifiable:** PROJ-104 (no platform), PROJ-105 (no catalog match — confirm estimate manually or extend the catalog if this work recurs).
```

---

## Quality Rules

- **The catalog match is the load-bearing assumption.** Always show match confidence as returned by `/sub:catalog-match`. Low/None confidence means the verdict is advisory, not authoritative — surface the sub-agent's `Alternatives` to the PM in those cases.
- **Always show the catalog range** next to the estimate. The PM should be able to judge for themselves.
- **Treat ⚠️ Low with at least as much weight as ⚠️ High.** Under-estimates are silent scope debt; they don't show up as missed sprints until late.
- **Do not aggregate across platforms.** Multi-platform tickets are 🚫 Unverifiable. Recommend splitting per platform before re-estimating.
- **Do not propose splits, capacity adjustments, or sprint changes.** Out of scope for this skill.
- **Do not infer estimates** for stories the PM didn't estimate. Flag and stop.
- **No magic thresholds.** "Too big" is defined by the catalog upper bound for that feature, not a fixed dev-day number.
- **50% deviation is a discussion, not a rejection.** Catalog ranges are anchors. New platforms, integration overhead, and migration work are legitimate reasons to exceed them — the skill's job is to surface the gap so the PM can decide.
