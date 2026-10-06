---
name: sow-importer
description: Parse a Statement of Work and any supporting pre-sales or delivery documents into a structured project-context.md. Triggers on "import this SOW", "parse this SOW", "extract project context from this document", "onboard from the SOW", "import these docs", or any request to bootstrap a new engagement from client documents. Produces a structured project-context.md and a gap/confidence report ready to feed into PRD generation or the project kickoff agent.
---

# Skill: SOW Importer

Parse a SOW and any supporting documents (sales deck, proposal, Gong summaries, Slack exports, meeting notes, technical architecture docs) into a structured `project-context.md`. Validates extracted values against `OTT-requirements-reference.md`. Surfaces gaps, anomalies, and DECISION REQUIRED blocks. Outputs a confidence score showing how complete the extracted context is.

This skill is the **pre-step to PRD generation**. Run it when you have raw client documents and need a structured project context before the PRD can be created.

For interactive intake when no documents exist, use `/onboard-project`.
For ongoing completeness validation, use `/gap-analysis`.

---

## Two-File Pattern

| File | Role |
|---|---|
| `OTT-requirements-reference.md` | Catalog - read-only menu of possible values, feature inventory, cert requirements |
| `project-context.md` | Per-engagement - what was extracted and confirmed for this project. This skill writes here. |

Treat the catalog like `.env.example` and `project-context.md` like `.env`. The catalog is the menu; the context file records what was chosen.

---

## Process

### 1. Load context

Invoke `.claude/sub/context-loader.md`.

- 🚫 Not configured: `project-context.md` does not exist yet - this run creates it.
- ⚠️ Partial: File exists with some fields filled. Ask: "Merge new data from documents into existing fields, or replace the whole file?" Default to merge - never clobber confirmed data.
- ✅ Complete: File exists and is fully populated. Ask: "Do you want to update specific sections from these new documents, or do a full re-import?"

### 2. Accept input documents

Accept any combination of:
- SOW / MSA / Change Requests / addendums (paste or file path)
- Sales proposal / pitch deck
- Signed order form
- CRM opportunity notes
- Gong call summaries / meeting transcripts
- Slack thread exports
- Discovery call notes
- Technical architecture docs
- Security / compliance requirements
- Previous project files (for expansion clients)

Tell the PM: "Share whatever you have - even a partial SOW or rough email chain is useful. I will extract what I can and flag everything missing."

Read and synthesize all documents together. Cross-reference conflicting information and flag conflicts explicitly.

### 3. Extract structured fields

Map extracted content to the catalog schema. For each field, check against `OTT-requirements-reference.md`:

- In catalog: record verbatim
- Plausibly project-specific (vendor name, custom provider): record verbatim, no flag
- Does not match categorical options (e.g. DRM method not in catalog): flag as anomaly - ask PM whether it is a typo, custom integration, or unsupported requirement

Extract in this order:

**A. Project Identity**
Project name, client, engagement type (Net-new / Migration / Enhancement), Jira project key, Confluence space, Slack channels, active phase, target launch date

**B. Team**
Named individuals or roles from all documents - PM, tech lead(s), designers, QA lead, client product owner, client tech lead, delivery manager

**C. Commercial**
Subscriber projections Y0-Y3, pricing tiers (name + price point), regions launching, languages supported, Merchant of Record

**D. Content Scope**
Live Events/PPV (types, count, free vs PPV), Live Channels (count), FAST Channels (count, syndication), VOD Movies/Series/Clips (count, average length), encoding ladder (SD/HD/4K, HDR variants, audio variants)

**E. Platform Launch Plan**
Per phase: specific platforms (validate against catalog list - iOS / Android / Web / Roku / Samsung Tizen / LG webOS / Fire TV / Apple TV / Vizio / HiSense / Meta Quest / AVP / Google TV / Fire Tablet / X1), target date, cert authorities

**F. Systems** - for each, extract provider + owner (Accedo / Client / Vendor):
OVP, CDN, DRM (provider + Widevine/FairPlay/PlayReady combination), Player SDK per platform, manifest format, IDM (auth methods, concurrent streams, access model), SMS (MOR, tiers, trials, coupons), Recommendations, Analytics, Advertising (DAI strategy, ad server, tag formats), CMS, Search, Front-End (design source, QA approach, form factors, Accedo Control)

**G. Compliance**
GDPR, CCPA, COPPA, ATT, accessibility standard, data residency regions

**H. Migration** (only if Migration or Enhancement)
Existing apps/platforms, user base size, account/token migration, subscription/billing migration, content catalog migration, cutover strategy, contract expiry on replaced systems

**I. Delivery Cadence**
Delivery model, client review rhythm, cert submission windows, external dependency dates

**J. Features**
For every feature mentioned or implied, map to catalog category and mark phase (MVP/Phase 2/Phase 3/Future/Out of Scope). Include verbatim source quote for traceability. Flag implicit features - "subscription-based" implies SMS + IAP + free trials + churn analytics.

### 4. Flag common SOW pitfalls

During extraction, watch for and flag:

- Scope vs timeline mismatch - 8 platforms in 4 months, or 6-month build with no discovery time
- Vague platform references - "smart TVs", "all major platforms", "connected devices" without explicit list
- System mentioned without provider - "DRM" with no vendor, "analytics" with no platform
- "Industry standard" - flag and ask for concrete spec
- "Same as last time" - ask which engagement specifically
- Implicit features not costed - flag each one
- Missing cert windows - any platform in scope without cert authority or submission window
- Vendor-speak - translate to Accedo terms and confirm

### 5. Confidence scoring

After extraction, score each domain:

```
## Extraction Confidence

| Domain | Confidence | Missing |
|--------|-----------|---------|
| Project Identity | [%] | [what is missing] |
| Team | [%] | [what is missing] |
| Commercial | [%] | [what is missing] |
| Content Scope | [%] | [what is missing] |
| Platform Plan | [%] | [what is missing] |
| Systems | [%] | [what is missing] |
| Compliance | [%] | [what is missing] |
| Features | [%] | [what is missing] |

Requirements confidence: [overall %]
Timeline confidence: [%] - based on whether go-live date exists and scope vs timeline ratio is realistic
Technical scope confidence: [%] - based on systems confirmed vs assumed

Missing information preventing higher confidence:
- [specific gap 1]
- [specific gap 2]
```

Score as: fields confirmed / total expected fields for that domain.

### 6. Gap and blocker summary

Produce a structured gap list formatted for the kickoff agent Red Flag stage:

```
## Blockers Before Kickoff

CRITICAL (must resolve before work begins)
- [gap] - [why it blocks]

HIGH (must resolve before sprint 1)
- [gap] - [why it matters]

MEDIUM (resolve before phase end)
- [gap] - [what it affects]

LOW (track as risk)
- [gap] - [what to watch]
```

### 7. DECISION REQUIRED blocks

For every ambiguous, conflicting, or vague statement:

```
DECISION REQUIRED
What: [The unknown]
Why it matters: [Downstream impact]
Options:
  1. [Option A]
  2. [Option B]
Recommended default: [Recommendation + rationale]
Owner: [who decides]
```

### 8. Show draft and get approval

Present the full extracted draft. Ask:
"Does this look right? Reply confirm to write to project-context.md, or describe edits before I write."

Never write the file without explicit confirm.

### 9. Write project-context.md

On confirm, write (or merge into) `project-context.md` following the schema below. In merge mode, preserve any existing confirmed fields. Never overwrite a filled field with a blank.

### 10. Recommend next steps

After writing, surface the right next skill based on what is missing:
- Gaps remain: Run /onboard-project to fill DECISION REQUIRED items interactively
- Context confirmed: Run /generate-prd to produce the PRD
- Features need detail: Run /feature-discovery on [feature name]
- Platform scope set: Run /cert-checklist
- Full completeness check needed: Run /gap-analysis

---

## project-context.md Schema

```markdown
# Project Context - [Client] / [Project Title]

> Generated: [YYYY-MM-DD] by /sow-importer
> Reference catalog: OTT-requirements-reference.md

## Project Identity
- **Project Title:**
- **Client:**
- **Engagement Type:** Net-new build / Migration / Enhancement
- **Jira Project Key:**
- **Confluence Space:**
- **Slack / Teams Channel:**
- **Active Phase:**
- **Target Launch:**

## Team
| Role | Name | Contact |
|---|---|---|
| PM | | |
| Tech Lead | | |
| Designer | | |
| QA Lead | | |
| Client Product Owner | | |
| Client Tech Lead | | |
| Delivery Manager | | |

## Commercial
- **Subscriber Projections:** Y0: · Y1: · Y2: · Y3:
- **Pricing Tiers:**
  - T1: [name, price]
  - T2: [name, price]
- **Regions Launching:**
- **Languages Supported:**
- **Merchant of Record:** Yes / No

## Content Scope
- **Live Events / PPV:** Yes / No / N/A
- **Live Channels:**
- **FAST Channels:**
- **VOD Movies:**
- **VOD Series:**
- **VOD Clips:**
- **Encoding Ladder:**

## Platform Launch Plan
| Phase | Platforms | Target Date | Cert Authorities |
|---|---|---|---|
| MVP | | | |
| Phase 2 | | | |
| Phase 3 | | | |

## Systems

### OVP
### CDN
### DRM
### Player SDK (per platform)
### IDM
### SMS
### Recommendations
### Analytics
### Advertising
### CMS / Metadata
### Search
### Front-End

## Compliance
- **GDPR:** Required / N/A
- **CCPA:** Required / N/A
- **COPPA:** Required / N/A
- **ATT (iOS):** Required / N/A
- **Accessibility:**
- **Data Residency:**

## Migration
*(only if Migration or Enhancement)*

## Delivery Cadence
- **Delivery Model:**
- **Client Review Rhythm:**
- **Cert Submission Windows:**
- **External Dependency Dates:**

## Features in Scope
| Category | Feature | Phase |
|---|---|---|

## Open Decisions
DECISION REQUIRED
What:
Why it matters:
Options:
Recommended default:
```

---

## Quality Rules

- Never paraphrase into vagueness. "Widevine L1 with PallyCon multi-DRM" - capture exactly, not just "Widevine."
- Surface conflicting statements across documents - DECISION REQUIRED.
- Validate every categorical value against the catalog. Flag anomalies.
- Flag implicit features. "Subscription-based" implies SMS + IAP + free trials + churn analytics.
- Translate vendor-speak. Confirm with PM before writing.
- Verbatim source quotes in the Features table for auditability.
- Never write without explicit confirm.
- Preserve existing data in merge mode. Never overwrite a confirmed field with a blank.
- Confidence scores must be honest. A 60% score is useful. A false 95% destroys trust.

---

> **Skill verification:** Confirm sow-importer was run and produced an extraction confidence score and a Blockers Before Kickoff section. If not, run from the top.
