---
name: feature-discovery
description: Use this skill to analyze and scope a feature request before stories are written. Triggers include "scope this feature", "run feature discovery", "create a feature brief", "assess risks for this feature", "help me plan this feature", "is this feature feasible", or any request to turn a raw feature idea into a structured plan. Produces a Feature Discovery Report or a client-facing Feature Brief. Do not generate stories - recommend /generate-stories once discovery is complete.
version: 3.0.0
---

# Skill: Feature Discovery

Structured analysis of a raw feature request **before** any stories are generated. Output is a Feature Discovery Report (internal) or Client Brief (stakeholder-facing) - not Jira tickets, not stories.

Three output modes:
- **Light** (default for trivial features): 6-line summary, proceed to `/generate-stories`
- **Full**: 20-dimension report with risk scoring, effort estimates, and downstream-ready blocks. For non-trivial features.
- **Client Brief**: stakeholder-facing 2-page document. Use when the PM needs a shareable brief for a client or internal review.

---

## Files

- **Catalog:** `OTT-requirements-reference.md` - standard feature inventory. Determines whether a feature is whitelabel-standard or bespoke.
- **Project:** loaded via `.claude/sub/context-loader.md` - confirms MVP / phase / out-of-scope status and which systems are in play for this engagement.

---

## Risk Scoring

Used in Technical Flags and Risk sections of the Full Report and Client Brief.

**Priority = Impact × Likelihood (1-3 each)**

| Level | Impact | Likelihood |
|-------|--------|------------|
| 1 | Minor - workaround exists, cosmetic, one person blocked | Unlikely (<30%) |
| 2 | Significant - feature delayed 1-2 weeks, quality concerns | Reasonable chance (30-60%) |
| 3 | Critical - project blocked, launch delayed, team crisis | Very likely (>60%) |

**Priority buckets:** 🔴 Critical 7-9 · 🟠 High 4-6 · 🟡 Medium 2-3 · 🟢 Low 1

**Risk entry format:**
> **Risk**: [title] | **Type**: Timeline / Technical / External / Resource | **I**: 1-3 | **L**: 1-3 | **Score**: I×L | **Owner**: [name or TBD] | **Mitigation**: [Prevention] / [Contingency] / [Trigger to escalate]

---

## Effort Sizing

Applied per story in the Story Breakdown Preview.

| Size | Duration | Signals |
|------|----------|---------|
| S | 1-2 days | Single component, no new APIs, designs finalized, no system boundary |
| M | 3-7 days | New + modified components, 1-2 APIs, some design iteration, moderate complexity |
| L | 1-3 weeks | Multiple new components, complex state, multiple APIs, design or architecture unknowns |

When in doubt, go up a size. Add 20-30% buffer for any feature touching DRM, auth, or payment.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load and validate project context.

### 1.5. Auto-pull existing context

Before parsing the feature request, automatically search for related context across three sources. Do not ask the user first - search first, then confirm.

**1.5.1. Jira auto-search:**
Invoke `.claude/sub/jira-project-snapshot.md` with the feature area keywords as the search scope. Extract relevant tickets, statuses, and linked issues from the returned snapshot. If the Jira search returns no results or the tool is unavailable, skip and note "No related Jira tickets found" in the context summary.

**1.5.2. Slack auto-scan:**
Invoke `.claude/sub/slack-signal-scan.md` with time window 14d and all configured channels. From the returned signals, extract decisions, blockers, and context relevant to the feature area keywords. If the scan returns no results or the tool is unavailable, skip and note "No relevant Slack signals found" in the context summary.

**1.5.3. Meeting notes auto-scan:**
Scan `product-development/product/meetings/[Client]/meeting-notes/` files from the last 14 days - check `standup/`, `weekly-client-sync/`, and `team-bi-weekly/` subfolders for feature-area keyword matches. Extract relevant decisions and action items. If the folder is empty or no matches are found, skip and note "No relevant meeting notes found" in the context summary.

**1.5.4. Present and confirm:**
After auto-scan, present a summary of what was found:

> **Related context found:**
> - **Jira:** [N tickets found - list key/summary/status for top 3-5]
> - **Slack:** [N relevant messages - list key decisions/blockers]
> - **Meeting notes:** [N mentions - list key decisions/context]
>
> Are there additional tickets or threads to include, or should I proceed with this context?

- **If user provides additional tickets/threads:** Fetch them using `mcp__plugin_atlassian_atlassian__getJiraIssue` or `.claude/sub/slack-context-extractor.md` and merge into context.
- **If user says proceed:** Continue to Step 2 with the auto-pulled context.

Context pulled here feeds directly into the gap analysis, scope boundary, and DECISION REQUIRED sections - preventing re-opening decisions already made or duplicating work already in flight.

### 2. Parse the input
Identify from the raw request:
- Feature area (playback, search, profile, EPG, paywall, etc.)
- Candidate platforms (explicit or inferred - never assumed)
- User type(s) and business motivation implied
- External references provided (Figma, Confluence, ticket, transcript)

### 3. Triage - Light, Full, or Client Brief?

**Light** if ALL of the following are true:
- Single platform
- No system boundary (no auth, playback, payment, analytics, sync, or non-passive network call)
- No multi-locale UI impact
- No cert risk surface (no D-pad introduction, DRM touch, IAP, ATT/COPPA)
- Catalog match: High confidence or obviously trivial (copy change, link relocation)

**Client Brief** if: PM explicitly asks for a shareable brief, or if the output is going to a client/stakeholder review rather than directly to story generation.

**Full** otherwise. When unclear, default to Full - PM can downgrade.

*In Light mode: skip steps 4-6 and emit the Light Output.*

### 4. Cross-reference catalog and project scope

**Catalog (`OTT-requirements-reference.md`):**
- High: ✅ Standard catalog feature
- Medium: ✅ Standard, flag scope notes and alternatives for PM confirmation
- Low: ⚠️ Likely standard but weak match - flag as DECISION REQUIRED
- None: 🚫 Bespoke - no typical-effort benchmark; flag architecture spike recommendation

**Project (per the loaded project context from step 1):**
- Is this in MVP / Phase 2 / Phase 3 / Out-of-scope per the engagement's launch plan?
- Are the systems it touches (OVP, DRM, IDM, SMS, Ads, Analytics) confirmed for this project?
- If out of scope or bespoke, flag prominently - PM may need to raise a Change Request.

### 5. Run 20-dimension gap analysis
Rate each as **Confirmed** / **Assumed** / **Missing**. "The PRD probably implies this" = Assumed, not Confirmed.

| # | Dimension |
|---|-----------|
| 1 | User type(s) and persona |
| 2 | Platform scope |
| 3 | Design reference (Figma or wireframe) |
| 4 | Technical systems touched (OVP, CMS, IDM, SMS, etc.) |
| 5 | DRM / playback stack (player SDK, DRM provider, manifest, CDN) |
| 6 | Authentication / identity flows (token refresh, concurrent streams, profiles) |
| 7 | Subscription / monetization (tier gating, IAP vs web, free trial) |
| 8 | Advertising (CSAI/SSAI, VAST/VMAP, ad consent) |
| 9 | Analytics events and parameters |
| 10 | Localization (EN/FR or other variants, string length) |
| 11 | Accessibility (WCAG, VoiceOver/TalkBack, captions, audio description) |
| 12 | Edge cases - evaluate: happy path · error conditions (network/API/auth failure) · boundary conditions (empty/single/max) · concurrency (double-tap, race condition) · recovery (mid-op disconnect, app close) |
| 13 | Error / empty / loading states |
| 14 | Certification implications per platform |
| 15 | Spec readiness for AI codegen - concrete inputs/outputs, integration contracts referenced, no vague verbs, no embedded unresolved decisions |
| 16 | Backend dependency assessment - are required APIs confirmed, built, documented? New API contracts needed? Existing contracts changing? |
| 17 | Feature flag strategy - progressive rollout plan? A/B testing? Flag infrastructure (Accedo Control) support? Rollback strategy? |
| 18 | Regression risk surface - what existing features could break? Shared components touched (LRUD, player, Redux store)? Blast radius? |
| 19 | Content rights & licensing - geo-blocking rules, time windows, tier gating? Interaction with client validation API? Offline rights? |
| 20 | Device performance profiling - memory/rendering assessment for low-end devices? Xbox UWP, Xumo, older Tizen/webOS constraints? Performance budget defined? |

### 6. Targeted clarification
If critical gaps exist, ask **at most 3** questions, prioritized:
1. Platform scope (always first if missing)
2. User need / business goal (if unclear)
3. Design reference (if expected but absent)

Wait for answers before producing the report - if PM says "use defaults," apply recommended defaults and document in DECISION REQUIRED.

### 7. Produce the output.

### 7.1. Save the output file.

Save the report to the appropriate path:
- Full report: `product-development/product/PRDs/[client]/YYYY-MM-DD-[feature-name]-discovery.md`
- Client Brief: `product-development/product/PRDs/[client]/YYYY-MM-DD-[feature-name]-brief.md`

Use today's date and a kebab-case feature title.

### 7.2. Generate HTML export.

After saving, run:

```
python3 scripts/md-to-html.py <path-to-saved-file>
```

This generates a styled `.html` file inside an `html/` subfolder in the same directory. Report the HTML path to the user so they know it exists.

### 7.5. Self-validate before emitting (Full and Client Brief only)

Before emitting the report, internally evaluate the output against 4 spec readiness dimensions:

1. **Concrete inputs/outputs**: Are data formats specified (JSON schemas, API contracts, event payloads) rather than prose-only descriptions? Count concrete vs. prose-only.
2. **Integration contracts referenced**: Are linked specs, API docs, or Figma URLs cited by name - or just referred to generically ("the API", "the backend")?
3. **Vague-verb density**: Count instances of imprecise language: "handles gracefully", "works correctly", "as appropriate", "properly manages", "seamlessly integrates". Each is a spec gap.
4. **Open decision count**: Count all ⚑ DECISION REQUIRED blocks and unresolved Missing dimensions.

Append a `## Spec Readiness Score` block to the output:

```markdown
## Spec Readiness Score

| Dimension | Result |
|-----------|--------|
| Concrete I/O | [N of M fields have data format specs] |
| Integration contracts | [N named / N generic references] |
| Vague-verb count | [N instances - list top 3] |
| Open decisions | [N unresolved] |

**Verdict:** Ready / Hold / Block
- **Ready**: 0 open decisions, ≤2 vague verbs, >80% concrete I/O
- **Hold**: 1-2 open decisions, or 3-5 vague verbs, or 50-80% concrete I/O
- **Block**: 3+ open decisions, or >5 vague verbs, or <50% concrete I/O
```

If verdict is Hold or Block, add specific actions to the Recommended Next Step section.

---

## Light Output

```
## Feature - [Title]
- Catalog match: [Category > Feature] (confidence: High/Medium/Low/None)
- Platform: [single platform]
- User need: As a [user type], I [need], so that [outcome].
- Open: [one-liner if anything material; omit otherwise]
- Ready for /generate-stories
```

---

## Full Feature Discovery Report

```markdown
# Feature Discovery - [Feature Title] - [Date]

## Feature Summary
[2-3 sentences: what, who, why]

## Catalog Match
- Standard catalog feature: ✅ [Category › Feature] / 🚫 Bespoke
- Match confidence: High / Medium / Low / None
- Catalog typical effort: [range or N/A for bespoke]
- Scope notes: [if input scope diverges from catalog entry]
- Alternatives considered: [list when confidence ≤ Medium]
- In project scope: ✅ MVP / ⚪ Phase 2 / 🔵 Phase 3 / 🚫 Out of scope / ⚠️ Not yet captured

## Business Goal
[Single sentence: the outcome this feature drives]

## User Need Statement
As a [user type], I need [capability] so that [outcome].

## Platform Scope
Build this table dynamically from the loaded project context. For CBC, the active platforms are:

| Platform | In Scope | Gem/TouTV Divergence | Notes |
|----------|----------|---------------------|-------|
| Samsung Tizen | ✅ / 🚫 / TBD | [Brand-specific differences if any] | PSDK AVPlay, LRUD focus, memory constraints |
| LG webOS | ✅ / 🚫 / TBD | [Brand-specific differences if any] | Magic Remote pointer mode, webOS media pipeline |
| Xbox UWP | ✅ / 🚫 / TBD | [Brand-specific differences if any] | PlayReady DRM (NOT Widevine), gamepad input |
| Comcast X1 / Rogers Ignite | ✅ / 🚫 / TBD | [Brand-specific differences if any] | RDK/Firebolt, operator-specific middleware |
| Xumo | ✅ / 🚫 / TBD | [Brand-specific differences if any] | Newest platform, limited docs |

**Rule:** If the feature request names a platform not in the active set above (e.g., iOS, Roku, Fire TV), flag it as ⚠️ - that platform is sunset or not in scope for the client. Do not assume it is in scope.

## Systems Touched
- [System]: [why]

## Gap Analysis (20 dimensions)
| # | Dimension | Status | Note |
|---|-----------|--------|------|
| 1 | User type | Confirmed / Assumed / Missing | |
[... all 20 rows]

## Technical Flags & Risks
- Architecture risk: [description] | I: _ L: _ Score: _ 🔴/🟠/🟡/🟢
- DRM / playback implications: [player SDK, DRM, license server, manifest]
- Certification risk: [per-platform]
- Performance / memory risk: [TV platforms especially]
[Use Risk Scoring format for each scored risk]

## Scope Boundary
**In scope:** [...]
**Out of scope:** [...]
**Deferred:** [...]

## Story Breakdown Preview
1. [Story title] - [single deployable outcome, one platform, one decision boundary] - **[S/M/L]**
2. [Story title] - **[S/M/L]**
[If a story mixes platforms or decision boundaries, propose decomposition. Split for clarity, not size.]

## Feasibility
**Rating:** Low Risk / Medium Risk / High Risk
**Go/No-Go:** ✅ Go / ⚠️ Go with conditions / 🚫 No-Go
**Conditions (if any):** [what must be resolved before proceeding]

## DECISION REQUIRED
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...

## Downstream Ready

Pre-formatted outputs for downstream skills. Each block is structured so the receiving skill can import directly without re-asking clarification questions.

### For /risk-register
| Risk | Type | I | L | Score | Owner | Mitigation | Trigger |
|------|------|---|---|-------|-------|------------|---------|
| [Risk from Technical Flags] | Timeline / Technical / External / Resource | 1-3 | 1-3 | I×L | [Owner name or TBD] | [Prevention] / [Contingency] | [Escalation trigger] |

### For /generate-stories
- **Recommended tier:** Minimal / Standard / Heavy
- **Confirmed platform scope:** [list of confirmed platforms from Platform Scope table]
- **Integration contracts:** [named API contracts, SDK versions, service endpoints]
- **Pre-validated AC patterns:**
  - Given [precondition], When [action], Then [expected outcome]
  - Given [error condition], When [trigger], Then [graceful handling]
  - Given [platform-specific condition], When [action], Then [platform-specific outcome]

### For /estimate-review
- **Catalog match:** [Category › Feature] | Confidence: [High/Medium/Low/None]
- **Typical effort range:** [from catalog or bespoke estimate]
- **Platform-split guidance:** [which platforms have base effort vs. inflated effort]
- **Complexity multipliers:** [DRM +X%, multi-platform +X%, bilingual +X%, etc.]

### For /roadmap-sync
- **Recommended quarter:** [Q based on dependencies and capacity]
- **Estimated duration:** [N sprints]
- **Hard dependencies:** [what must be done first - APIs, designs, other features]
- **Cert lead time:** [weeks needed for platform certification, if applicable]

## Recommended Next Step
- ✅ Ready for /generate-stories
- ⚠️ Needs clarification - see DECISION REQUIRED
- ⚠️ Needs design reference before story generation
- ⚠️ Needs architecture discussion
- 🚫 Out of current scope - raise Change Request first
```

---

## Client Brief Output

Use when output is going to a client or stakeholder review (not directly to story generation).

```markdown
# [Feature Name] - Feature Brief

**Client:** [Client Name]
**Date:** [YYYY-MM-DD]
**Target Release:** [date or TBD]
**Overall Feasibility:** Low Risk / Medium Risk / High Risk

---

## Overview
[1-2 sentences: what the feature is and its core value]

---

## Frontend Changes Required

**Components:** [new or modified]
**API Integration:** [endpoints and purpose]
**Design Status:** ☐ Available · ☐ In progress (ETA: ) · ☐ Not started
**Estimated Effort:** S / M / L

---

## Dependencies & Blockers
| Dependency | Type | Blocks | Status |
|------------|------|--------|--------|
| | | | |

---

## Risks
| Risk | Type | I | L | Score | Mitigation |
|------|------|---|---|-------|------------|
| | Timeline / Technical | | | | |

---

## Edge Cases & Testing
- [ ] [Happy path scenario]
- [ ] [Error condition]
- [ ] [Boundary / concurrency scenario]
- [ ] [Platform-specific scenario]

---

## Downstream Ready

### For /risk-register
| Risk | Type | I | L | Score | Owner | Mitigation | Trigger |
|------|------|---|---|-------|-------|------------|---------|
| [Risk from Risks table] | Timeline / Technical / External / Resource | 1-3 | 1-3 | I×L | [Owner] | [Prevention] / [Contingency] | [Trigger] |

### For /generate-stories
- **Recommended tier:** Minimal / Standard / Heavy
- **Confirmed platform scope:** [from Dependencies table]
- **Integration contracts:** [API contracts identified]
- **Pre-validated AC patterns:** Given/When/Then for key scenarios

### For /estimate-review
- **Catalog match + confidence:** [match details]
- **Typical effort range:** [from catalog]
- **Platform-split guidance:** [per-platform effort notes]

### For /roadmap-sync
- **Recommended quarter:** [Q]
- **Estimated duration:** [N sprints]
- **Hard dependencies:** [list]

---

## Spec Readiness Score
[See Step 7.5 for how to populate this section]

---

## Go/No-Go
**Recommendation:** ✅ Go / ⚠️ Go with conditions / 🚫 No-Go
**Next steps:**
1. [Action]
2. [Action]
```

After saving the Client Brief file, run:

```
python3 scripts/md-to-html.py <path-to-saved-brief>
```

Report the HTML path to the user so they know it exists.

---

## Quality Rules

- **Platform scope is never assumed.** "Add a search feature" with no platform list = Missing, not Assumed All.
- **DRM and playback stack must be surfaced** for any playback-touching feature. Never omit player SDK, DRM provider, CDN, and manifest format.
- **Scope boundary is mandatory.** Always state what is explicitly out.
- **Compliance triggers must be flagged**: data collection - GDPR/CCPA; iOS tracking - ATT; kids content - COPPA.
- **Catalog match is mandatory.** Don't skip the sub-agent call - confidence calibration must stay consistent across discovery, estimate-review, and gap-analysis.
- **Bespoke features (None confidence) warrant extra scrutiny.** Flag architecture/risk/estimation impact prominently. Recommend a spike.
- **Risk scores must be explicit.** Don't describe a risk without an I×L score.
- **Effort sizes must be explicit.** Every story in the breakdown gets S/M/L - no story left unsized.
- Be opinionated. If the PM has framed the feature too broadly, say so.
- Never silently fill missing dimensions - unknown goes in DECISION REQUIRED.
- **CMP/Didomi reminder (surface to PM, do not add to discovery output):** After completing discovery, check: does this feature involve navigation, modals, overlays, player controls, or consent flows? If yes, add a callout in the Risks section: "CMP/Didomi risk: this feature may affect consent navigation. Ensure CMP regression is scoped into QA planning."

---

> **Skill verification:** Please ensure that the skill feature-discovery SKILL.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
