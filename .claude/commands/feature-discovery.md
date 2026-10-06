# Skill: Feature Discovery

Structured analysis of a raw feature request **before** any stories are generated. Pre-story activity. Output is a Feature Discovery Report — not Jira tickets.

Two modes:
- **Light** (default for trivial features) — 6-line summary, jump to `/generate-stories`. Triggered automatically when triage signals show trivial scope.
- **Full** — 15-dimension report. For non-trivial features (multi-platform, system boundary, playback / auth / payment, multi-locale UI).

Universal output rules apply — see CLAUDE.md → OUTPUT STYLE.

You do not generate stories in this skill. If the PM is ready for stories, recommend `/generate-stories` once discovery completes.

---

## Files

- **Catalog:** `OTT-requirements-reference.md` — the standard feature inventory. Use to determine whether the requested feature is part of the standard whitelabel offering or net-new / bespoke.
- **Project:** `project-context.md` — confirms whether the feature is in MVP / a later phase / out of scope for *this* engagement, and the platform / system context the feature must respect.

---

## Process

### 1. Load context
Invoke `/sub:context-loader`. If health is 🚫 **Not configured**, stop and recommend `/onboard-project`. If ⚠️ Partial, proceed but flag missing context as DECISION REQUIRED in the report.

### 2. Parse the input
Identify, from the raw request:
- Feature area (e.g. playback, search, profile management, EPG, paywall)
- Candidate platforms (explicit or inferred — never assumed)
- User type(s) implied
- Business motivation implied
- Any external references provided (Figma, Confluence, ticket, transcript)

### 2.5 Triage — Light or Full?
Light mode if **all** of the following are true:
- Single platform
- No system boundary (no auth, playback, payment, analytics, sync, network call beyond passive content fetch)
- No multi-locale UI impact
- No cert risk surface (no D-pad introduction, no DRM touch, no IAP, no ATT/COPPA trigger)
- Catalog match returns High confidence (single clear feature) or the feature is obviously trivial (e.g. copy change, link relocation)

Full mode otherwise. If unclear, default to Full and let the PM downgrade.

**In Light mode, skip steps 3–5 and emit the Light Output below.**

### 3. Cross-reference catalog and project scope

**Against the catalog (`OTT-requirements-reference.md`):**
Invoke `/sub:catalog-match MATCH "<feature description>" WITH_EFFORT`. Use the returned **Confidence** to classify:
- High → ✅ Standard catalog feature
- Medium → ✅ Standard catalog feature, with the sub-agent's `Scope notes` and `Alternatives` propagated to the report so the PM can confirm the right entry
- Low → ⚠️ Likely a catalog feature but match is weak — flag for PM confirmation in DECISION REQUIRED
- None → 🚫 Bespoke (not in catalog) — typical-effort benchmark will not apply; estimating will need first-principles work

**Against the project (`project-context.md`):**
- Is this in MVP / Phase 2 / Phase 3 / Future / Out-of-scope per the project's launch plan?
- Is it called out in the SOW / PRD that drove onboarding?
- Are the systems it touches confirmed for this project (OVP, DRM, IDM, SMS, Ads, Analytics)?

If the feature is out of scope or in a later phase per project context, flag this prominently — the PM may need to raise a Change Request. If the feature is bespoke (None confidence), call it out — the design / estimation / risk profile differs from standard-catalog work.

### 4. Run 15-dimension gap analysis
Rate each as **Confirmed** / **Assumed** / **Missing**. Be ruthless — "the PRD probably implies this" is **Assumed**, not Confirmed.

| # | Dimension |
|---|---|
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
| 12 | Edge cases (offline, slow network, concurrent action, partial state) |
| 13 | Error / empty / loading states |
| 14 | Certification implications per platform |
| 15 | Spec readiness for AI codegen — concrete inputs/outputs, integration contracts referenced (not described), no vague verbs, no embedded unresolved decisions |

### 5. Targeted clarification
If critical gaps exist, ask **at most 3** questions, prioritized:
1. Platform scope (always first if missing)
2. User need / business goal (if unclear)
3. Design reference (if expected but absent)

Wait for answers before producing the final report — but if the PM says "use defaults," apply your recommended defaults and document them in DECISION REQUIRED.

### 6. Produce the Feature Discovery Report.

---

## Light Output *(trivial features)*

```
## Feature — [Title]
- Catalog match: [Category > Feature] (confidence: [High/Medium/Low/None])
- Platform: [single platform]
- User need: As a [user type], I [need], so that [outcome].
- Open: [one-liner if anything material; omit otherwise]
- Ready for /generate-stories
```

No 15-dimension table. No scope-boundary section. No technical-flags section. The PM can request a Full report if Light proves insufficient.

---

## Full Feature Discovery Report

```
# Feature Discovery — [Feature Title] — [Date]

## Feature Summary
[2–3 sentences: what, who, why]

## Catalog Match *(from /sub:catalog-match)*
- **Standard catalog feature:** ✅ [Category › Feature] / 🚫 Bespoke
- **Match confidence:** High / Medium / Low / None
- **Catalog typical effort:** [range or — for bespoke]
- **Scope notes:** [if input scope diverges from catalog entry]
- **Alternatives considered:** [list when confidence ≤ Medium]
- **In project scope:** ✅ MVP / ⚪ Phase 2 / 🔵 Phase 3 / 🚫 Out of scope / ⚠️ Not yet captured in project-context.md

## Business Goal
[Single sentence — the outcome this feature drives]

## User Need Statement
As a [user type], I need [capability] so that [outcome].

## Platform Scope
| Platform | In Scope | Notes |
|---|---|---|
| iOS | ✅ / 🚫 / TBD | [FairPlay implications, etc.] |
| Android | ✅ / 🚫 / TBD | [Widevine L1/L3] |
| Fire TV | ✅ / 🚫 / TBD | [D-pad, Amazon IAP] |
| Web | ✅ / 🚫 / TBD | [Browser DRM split] |
| Samsung Tizen | ✅ / 🚫 / TBD | [PSDK, memory] |
| LG webOS | ✅ / 🚫 / TBD | [Lightning lifecycle] |

## Systems Touched
- [System]: [why]

## Gap Analysis (15 dimensions)
| # | Dimension | Status | Note |
|---|---|---|---|
| 1 | User type | Confirmed / Assumed / Missing | ... |

## Technical Flags
- **Architecture risk:** [...]
- **DRM / playback implications:** [player SDK, DRM, license server, manifest]
- **Certification risk:** [per-platform]
- **Performance / memory risk:** [TV platforms especially]

## Scope Boundary
**In scope**
- [...]

**Out of scope**
- [...]

**Deferred to later phase**
- [...]

## Story Breakdown Preview
1. [Story title] — [single deployable outcome on one platform with one decision boundary]
2. [Story title]
[If any story above mixes multiple decision boundaries, platforms, or deployable outcomes, propose a decomposition. Splitting is for clarity, not capacity — do not split because of perceived size alone.]

## DECISION REQUIRED
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...

## Recommended Next Step
- ✅ Ready for `/generate-stories`
- ⚠️ Needs clarification — see DECISION REQUIRED
- ⚠️ Needs design reference before story generation
- ⚠️ Needs architecture discussion (player SDK / DRM / system selection)
- 🚫 Out of current scope — raise Change Request first
```

---

## Quality Rules

- **Platform scope is never assumed.** If the request says "add a search feature" with no platform list, this is Missing — not Assumed All.
- **DRM and playback stack must be surfaced** for any feature touching playback. Never produce a discovery report for a playback feature without naming player SDK, DRM provider, CDN, and manifest format (or flagging each as Missing).
- **Scope boundary is mandatory.** Always state what is explicitly out — vague scope is the leading cause of delivery overrun.
- **Compliance triggers must be flagged**: any data collection → GDPR/CCPA; any tracking on iOS → ATT; any kids content → COPPA.
- Be opinionated. If the PM has framed the feature too broadly, say so in the report.
- Never silently fill missing dimensions. If unknown, it goes in the DECISION REQUIRED section.
- **Catalog match is mandatory** and must come from `/sub:catalog-match`, not inline heuristics. Don't skip the sub-agent call — confidence calibration and alternatives must come from one place so verdicts stay consistent across `/feature-discovery`, `/estimate-review`, and `/gap-analysis`.
- **Bespoke features (Confidence: None) warrant extra scrutiny.** When the catalog doesn't fit, flag the architecture / risk / estimation impact prominently. Recommend an architecture spike if needed.
