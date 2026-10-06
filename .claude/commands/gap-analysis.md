# Skill: Gap Analysis

Analyze a PRD, SOW, requirements doc, or Confluence page for completeness against the **whitelabel baseline catalog** (`OTT-requirements-reference.md`).

This skill compares an **input document** against the **catalog**. It does not require an active project — you can run it on a prospective SOW before any onboarding has happened. If `project-context.md` does exist, the analysis is more targeted (only checks categories relevant to the project's confirmed scope).

---

## Files

- **Baseline:** `OTT-requirements-reference.md` — the catalog. Every category and possible value is the standard against which the input doc is compared.
- **Optional context:** `project-context.md` — if present, narrows the analysis to in-scope categories and surfaces conflicts between the doc and the agreed project context.

---

## Process

### 1. Load context (optional)
Invoke `/sub:context-loader`.
- ✅ / ⚠️: use to focus the analysis. Categories already confirmed in `project-context.md` get a stricter check (the input doc must reinforce them); categories N/A in the project are skipped.
- 🚫: proceed without it. The input doc is judged purely against the full catalog.

### 2. Accept input
- Pasted document
- Confluence URL or ID → invoke `/sub:confluence-read` with `PAGE <id-or-url>`
- File path / file content

### 3. Parse the document
Extract everything the document states or strongly implies:
- Content scope (Live / VOD / FAST / events; counts; encoding)
- Platform scope (and phasing)
- Systems mentioned (OVP, CDN, DRM + provider, Player SDK, manifest format, IDM, SMS, Recommendations, Analytics, Ads, CMS, Search)
- Features mentioned or implied
- NFRs (performance, accessibility, localization, security, observability)
- Compliance triggers (GDPR, CCPA, COPPA, ATT, accessibility, data residency)
- Migration details (existing systems, cutover, account migration)
- Commercial dimensions (tiers, pricing, regions, MOR)

### 4. Walk every catalog category via `/sub:catalog-match`
Build the list of catalog paths to check, scoping appropriately:

**Systems Requirements** — OVP, CDN, Privacy & Compliance, IDM, SMS, Recommendations, Analytics, Advertising, Video Player, Front-End, Operations, Migration

**Feature Catalog** — Set Up / Architecture, Customer Management, Video Playback, Analytics, Advertising, In-App Purchase, User Experience, EPG, DVR, Content Discovery, Accessibility, PayTV

**Platform Certification Requirements** — only check platforms in scope per the doc

**Performance KPIs** — Core App KPIs, EPG / Playback KPIs

Invoke `/sub:catalog-match BATCH COVERAGE` with the catalog paths and the doc text. The sub-agent returns one of:
- ✅ **Confirmed** — entry is named **and** required specifics are present
- ⚠️ **Partial** — entry is named but specifics are missing or vague
- 🚫 **Missing** — entry is not mentioned and not implied

Mark entries N/A only when the doc explicitly excludes them.

**Do not redefine these classifications inline.** The vocabulary is owned by `/sub:catalog-match` so coverage scoring stays consistent with feature-discovery and estimate-review.

### 5. Cross-check against project context (if available)
If `project-context.md` exists, additionally flag:
- 🔁 **Conflict** — doc states something different from what's already confirmed in project context (e.g., doc says CSAI; project says SSAI)
- 🆕 **New scope** — doc introduces something not in the agreed project scope

### 6. Generate DECISION REQUIRED blocks
For every Partial / Missing / Conflict that materially affects scope, cost, or delivery risk.

### 7. Produce Requirements Analysis Report

---

## Report Format

```
## Requirements Analysis — [Document Title] — [Date]

### Completeness Score (catalog coverage)
[X of Y catalog entries confirmed; Z partial; W missing] — Overall risk: Low / Medium / High

### Spec Readiness Score (for AI codegen)
- **Concrete inputs/outputs:** ✅/⚠️/🚫 — does the doc specify actual data, formats, schemas, or only describe behaviour in prose?
- **Integration contracts referenced:** ✅/⚠️/🚫 — are API endpoints, event schemas, SDK versions linked to specs, or only named?
- **Vague-verb density:** Low / Medium / High — count of "works correctly", "handles gracefully", "as appropriate", "industry standard", "best practice" relative to the doc length
- **Open decisions in scope:** [N] — unresolved DECISION REQUIRED items the doc surfaces or implies
- **Codegen-readiness verdict:** Ready / Hold / Block — Block if the doc cannot be reduced to testable scenarios without inventing significant detail.

A doc can score high on catalog completeness and still Block on spec readiness — they measure different things.

### Summary
[2–3 sentences: highlights of the doc, top 2–3 highest-risk gaps]

### Gap Register

| Catalog Category | Entry | Status | Gap Description | Risk if Unresolved |
|---|---|---|---|---|
| Content — Live | FAST Channels | ⚠️ Partial | "FAST channels mentioned" but no count, provider, or syndication plan | Sizing impossible |
| Systems — DRM | Provider | 🚫 Missing | No DRM provider named despite SVOD content | High — playback architecture cannot proceed |
| Systems — Player | iOS SDK | 🚫 Missing | No player SDK selected for iOS | High — blocks playback estimating |
| Compliance — ATT | — | 🚫 Missing | iOS in scope; no ATT plan | High — App Store rejection risk |
| Platform — Samsung Tizen | PSDK + Tizen versions | ⚠️ Partial | Listed but no PSDK or OS range | Medium — texture / memory unknowns |
| Feature — Customer Management | Multi-Profiles | 🔁 Conflict | Doc says "no profiles"; project context says profiles=Yes | High — confirm with client |
| ... | ... | ... | ... | ... |

### DECISION REQUIRED Blocks
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...

### Recommended Actions
1. **Ask the client** — [Specific question to client about a gap]
2. **Escalate to architecture** — [System selection that needs an internal decision]
3. **Apply default** — [Where the PM can proceed with a documented default]
4. **Flag as CR risk** — [Where a future CR will likely be needed]
5. **Block until resolved** — [Where work cannot proceed]

### Suggested Next Steps
- [If no project-context.md exists] Run `/sow-importer` if this doc is the source of truth for the engagement.
- [If gaps include features] Run `/feature-discovery` on each high-risk feature before story generation.
- [If cert risks identified] Run `/cert-checklist` once platform scope is locked.
```

---

## Quality Rules

- **Never mark Confirmed if specifics are missing.** "Has DRM" is Partial, not Confirmed. The sub-agent enforces this — don't override its classifications.
- **DRM and playback gaps are always High risk.**
- **Compliance gaps (GDPR / CCPA / COPPA / ATT / accessibility) are always High risk** — they can block cert and launch.
- **Platform scope gaps block downstream story generation entirely** — say so explicitly.
- **Conflicts between the doc and `project-context.md` are always High risk** — they signal misalignment between client docs and agreed scope.
- **Don't be diplomatic about big gaps.** A SOW with no NFRs is a serious gap, not "an area for further detail."
- **Spec readiness is independent of catalog completeness.** A doc can list every catalog system and still produce wrong code under AI codegen if every behaviour is described with vague verbs. Score both axes.
- **Always provide a Recommended Action** for every gap — the PM needs to know what to do, not just what's missing.
- The catalog is the source of truth for the comparison. Do not introduce baseline criteria that aren't in `OTT-requirements-reference.md`.
