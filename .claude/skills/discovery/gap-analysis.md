---
name: gap-analysis
description: Use this skill when the user wants to analyze a PRD, SOW, or requirements document for completeness or gaps. Triggers on "run a gap analysis", "check this PRD for gaps", "analyze this SOW", "what's missing from this document", "requirements gap check", "analyze requirements", "check for completeness", "review this doc for gaps", "what are we missing from this spec", "coverage check", "requirements coverage", "spec coverage", "how complete is this PRD", or any request to compare a document against the OTT requirements baseline.
---

# Skill: Gap Analysis

Analyze a PRD, SOW, or requirements doc for completeness. Output is triage-first: what blocks work, what needs a decision, what can be watched and documented. One screen, actionable, no filler.

---

## Files

- **Baseline:** `OTT-requirements-reference.md` - the catalog. Every category and possible value is the standard against which the input doc is compared.
- **Optional context:** Loaded via `/sub:context-loader` - if available, narrows the analysis to in-scope categories and surfaces conflicts.

---

## Process

### 1. Load context (optional)
Invoke `/sub:context-loader`. If available, scope the analysis to confirmed project categories. Skip N/A categories. If unavailable, proceed against the full catalog.

### 2. Accept input
- Pasted document
- Confluence URL or ID - invoke `/sub:confluence-read` with `PAGE <id-or-url>`. If the page cannot be fetched, ask the user to paste the content directly.
- File path / file content

### 3. Parse the document
Extract:
- Platform scope
- Content scope (Live / VOD / FAST)
- Systems mentioned (OVP, CDN, DRM, Player, IDM, SMS, Analytics, Ads, CMS)
- Features mentioned or implied
- Compliance triggers (GDPR, CCPA, ATT, accessibility)

### 4. Check coverage against catalog
For each catalog category in scope, classify as:
- **Confirmed** - named AND specifics present
- **Partial** - named but specifics missing or vague
- **Missing** - not mentioned, not implied
- **N/A** - explicitly excluded by the doc

### 5. Triage findings
Sort every Partial and Missing into one of three buckets:

| Bucket | Criteria |
|--------|---------|
| **BLOCK** | Work cannot proceed without this. DRM, playback architecture, platform cert scope, compliance with launch risk. |
| **DECIDE** | A call or client response needed before the next phase. Won't block immediately but will cause rework if ignored. |
| **WATCH** | Low risk. Document the assumption and move on. |

### 6. Generate output

### 7. Save report and export HTML
Save to:
`product-development/product/customers/accounts/[client]/health-reports/YYYY-MM-DD-gap-analysis-[doc-title].md`

Run:
```
python3 scripts/md-to-html.py <path>
```

Report the HTML path to the user.

---

## Output Format

```
## Gap Analysis - [Document Title] - [Date]

**Overall risk:** Low / Medium / High
**Coverage:** X of Y catalog entries confirmed - Z gaps total (N block, N decide, N watch)

---

### BLOCK - [N items] - work cannot proceed

| # | Category | Gap | Why it blocks |
|---|---------|-----|---------------|
| 1 | DRM | No provider named | Playback architecture cannot be designed |
| 2 | Compliance - ATT | No ATT plan | iOS App Store rejection risk |

---

### DECIDE - [N items] - needs a call or client response

| # | Category | Gap | Decision needed |
|---|---------|-----|-----------------|
| 1 | Player SDK | Not specified | Affects estimating and platform builds |
| 2 | Profiles | Doc says no; project context says yes | Confirm with client before sprint planning |

---

### WATCH - [N items] - low risk, document assumption and proceed

| # | Category | Gap | Assumption to document |
|---|---------|-----|----------------------|
| 1 | Search | Basic search implied | Assumed keyword only, no typeahead |

---

### Recommended Next Steps
1. [Specific ask to client or internal team - one line each]
2. ...
```

---

## Rules

- **BLOCK, DECIDE, WATCH are the only output buckets.** Do not introduce other categories.
- **Omit empty sections entirely.** If there are zero BLOCK items, do not include the BLOCK section.
- **Max 10 rows across DECIDE and WATCH combined.** Group minor items into a single catch-all row if needed.
- **DRM and playback gaps are always BLOCK.**
- **Compliance gaps (GDPR / CCPA / COPPA / ATT / accessibility) are always BLOCK** - they can block cert and launch.
- **Platform scope gaps are always BLOCK** - no platform = no cert checklist = no delivery date.
- **Conflicts with loaded project context are always BLOCK.**
- **Recommended Next Steps must be specific.** "Ask the client about DRM provider before architecture starts" not "clarify requirements."
- **No Spec Readiness Score.** Do not include it.
- **400-word output target.** If the output exceeds this, you have included too much detail. Cut WATCH items first.
- **No filler.** Do not include sections, headers, or sentences that add no actionable information.

---

> **Skill verification:** Please ensure that the skill gap-analysis.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
