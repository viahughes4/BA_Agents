# Skill: SOW Importer

Parse a client SOW or PRD and bootstrap the per-engagement `project-context.md` from a real client document. Faster and more opinionated than `/gap-analysis` — this is the **intake skill** for a new engagement.

For ongoing completeness validation against the whitelabel baseline, use `/gap-analysis`. For interactive PM intake when no SOW exists, use `/onboard-project`.

---

## Two-File Pattern

| File | Role |
|---|---|
| `OTT-requirements-reference.md` | **Catalog** — schema for what to extract; option lists for validating extracted values. |
| `project-context.md` | **Per-engagement** — populated by this skill from the SOW. |

When extracting, validate every value against the catalog's possible values. If the SOW says "Acme Streaming Platform" for OVP, that's project-specific (free text, fine). If it says "WideVine X42" — that's not in the catalog's DRM list; flag as anomaly.

---

## Process

### 1. Inspect current state
Invoke `/sub:context-loader`.
- 🚫 / ⚠️: continue — this skill is intended to populate or fill gaps in `project-context.md`.
- ✅: ask whether the PM wants to merge SOW data into the existing file, or replace it. Default to merge.

### 2. Accept input
- Pasted SOW / PRD content
- Confluence URL → invoke `/sub:confluence-read` with `PAGE <url-or-id>`
- File path / file content provided by the PM

### 3. Extract structured fields
Map extracted content to the catalog schema. Pull, in this order:

1. **Project Identity** — project name, client, engagement type (Net-new / Migration / Enhancement), target launch date
2. **Team & stakeholders** — named individuals or roles
3. **Commercial model** — tiers, pricing, regions, languages, MOR
4. **Content scope** — Live (Events/PPV, Live Channels, FAST Channels), VOD (Movies, Series, Clips); counts and average lengths where given; encoding ladder
5. **Platform launch plan** — MVP / Phase 2 / Phase 3 platforms with target dates; cert authorities implied
6. **Systems** — for each, pull provider, owner, and any version/tier info:
   - OVP, CDN, DRM (with provider — Axinom / EZDRM / etc.), Player SDK per platform, manifest format, IDM, SMS, Recommendations, Analytics, Advertising, CMS, Search
7. **Compliance** — GDPR / CCPA / COPPA / ATT / accessibility / data residency
8. **Migration details** — if engagement is Migration / Enhancement
9. **Timeline / milestones** — cert dates, beta windows, launch dates
10. **Features mentioned or implied** — map to catalog feature categories
11. **Explicit exclusions / out-of-scope statements**

### 4. Validate against the catalog
For each extracted value, check if it matches the catalog's possible values.
- ✅ In catalog: record verbatim.
- ⚠️ Not in catalog but plausibly project-specific (e.g. provider name): record verbatim, no flag.
- 🚫 Doesn't match a categorical field's options (e.g., "DRM: SuperLock" when catalog lists FairPlay / Widevine / PlayReady): flag as anomaly — ask the PM whether this is a typo, a custom integration, or an unsupported requirement.

### 5. Map features to catalog
For every feature mentioned or implied, identify the catalog category and feature name. Mark each as MVP / Phase 2 / Phase 3 / Future / Out of Scope based on the SOW. If phasing isn't stated, mark as MVP and flag it.

### 6. Identify gaps
Quick diff vs the catalog. List the major missing pieces (e.g., "No DRM provider named", "No NFRs", "No accessibility commitment", "No Phase 2 plan"). Don't run a full gap analysis — that's `/gap-analysis`'s job.

### 7. Surface ambiguities
For every ambiguous, conflicting, or vague statement, generate a `⚑ DECISION REQUIRED` block.

Common SOW pitfalls to flag:
- **MVP scope contradicting timeline** — e.g., 8 platforms in 4 months
- **Mentioned without provider** — "DRM" with no system / vendor named
- **"All major platforms"** — no explicit list
- **"Industry standard"** — no concrete spec
- **Implicit features** — "subscription-based" implies SMS, IAP, free trials, churn analytics. Confirm scope.
- **Vendor-speak** — translate to Accedo terms ("OTT video management platform" → OVP).

### 8. Produce the import package
Show the full extracted draft using the schema below.

### 9. Approval gate
Ask:
> Shall I write this draft to `project-context.md`? Reply `confirm` to write, `cancel` to keep local, or describe edits.

### 10. Execute
On `confirm`, write `project-context.md` using the schema in the next section. After writing, recommend `/onboard-project` for any DECISION REQUIRED items the SOW didn't resolve.

---

## Output Format (draft preview before write)

```
# SOW Import — [Document Title] — [Date]

## Extracted Project Context (Draft for project-context.md)

### Project Identity
- Project Title: ...
- Client: ...
- Engagement Type: Net-new / Migration / Enhancement
- Target Launch: [date]
- Jira Project Key: [if stated, else TBD]
- Confluence Space: [if stated, else TBD]

### Team
| Role | Name | Contact |
|---|---|---|
| ... | ... | ... |

### Commercial
- Subscriber Projections: Y0/Y1/Y2/Y3
- Pricing Tiers: T1 / T2 / T3
- Regions Launching: [list]
- Languages: [list]
- MOR: Yes / No — [who]

### Content Scope
- Live Events / PPV: ...
- Live Channels: ...
- FAST Channels: ...
- VOD Movies / Series / Clips: ...
- Encoding Ladder: ...

### Platform Launch Plan
| Phase | Platforms | Target Date | Cert Authorities |
|---|---|---|---|
| MVP | ... | ... | ... |
| Phase 2 | ... | ... | ... |
| Phase 3 | ... | ... | ... |

### Systems
| System | Provider | Owner | Notes / Validation |
|---|---|---|---|
| OVP | ... | ... | ✅ in catalog / 🚫 anomaly |
| CDN | ... | ... | ... |
| DRM | Widevine + FairPlay via [vendor] | ... | ✅ |
| Player iOS | ... | ... | ... |
| Player Android | ... | ... | ... |
| Manifest format | HLS / DASH | — | ✅ |
| IDM | ... | ... | ... |
| SMS | ... | ... | ... |
| Recommendations | ... | ... | ... |
| Analytics | ... | ... | ... |
| Advertising | DAI: CSAI / SSAI / Both / N/A | ... | ... |
| CMS / Metadata | ... | ... | ... |
| Search | ... | ... | ... |

### Compliance
- GDPR: Required / N/A
- CCPA: Required / N/A
- COPPA: Required / N/A
- ATT (iOS): Required / N/A
- Accessibility: WCAG 2.1 AA / EAA / Section 508 / —
- Data Residency: [regions / None]

### Migration *(if applicable)*
- Existing apps · User base size · Account migration · Subscription migration · Content migration · Cutover strategy · Timeline · Reused systems · Replaced systems · Contract expiry

### Delivery Cadence
- **Delivery Model:** [Continuous / Weekly drop / Fixed milestones / Sprint-based — infer from SOW; default to Fixed milestones if SOW is milestone-driven]
- **Client Review Rhythm:** [extracted if stated, else TBD]
- **Cert Submission Windows:** [extracted if stated, else TBD]
- **External Dependency Dates:** [content go-lives, marketing launch dates, hardware availability — extracted from SOW]

*Traditional Agile fields (sprint length, velocity, ceremonies) are not extracted by default — capture in `/onboard-project` only if the client explicitly mandates Scrum/SAFe artifacts.*

### Features in Scope (mapped to catalog)
| Category | Feature | Phase | Source quote |
|---|---|---|---|
| Customer Management | Multi-Profiles | MVP | "users can create multiple profiles" |
| Video Playback | Live TV Playback | MVP | "live channel viewing required" |
| ... | ... | ... | ... |

### Explicit Exclusions
- [list]

## Gap Summary (vs OTT-requirements-reference.md)
- 🚫 Missing: [...]
- ⚠️ Partial: [...]
- ⚠️ Anomalies (values not in catalog options): [...]

## DECISION REQUIRED
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...

---

Ready to write to project-context.md — reply `confirm` to proceed.
```

---

## `project-context.md` Schema

This skill writes the same schema as `/onboard-project`. See `/onboard-project` for the full schema. The two skills must produce identical structure so a project can flip between SOW-driven imports and conversational fill-in without breaking downstream skills.

---

## Quality Rules

- **Don't paraphrase the SOW into vagueness.** "Widevine L1 with PallyCon multi-DRM" → capture exactly. Don't reduce to "Widevine."
- **Surface conflicting statements** — MVP scope vs timeline, contradictory platform lists, double-defined systems → DECISION REQUIRED.
- **Validate every categorical value against the catalog.** If a stated DRM, ad format, manifest, or auth method isn't in the catalog's option list, raise an anomaly flag.
- **Flag implicit features.** "Subscription-based" implies SMS + IAP + free trials + churn analytics. Confirm scope.
- **Translate vendor-speak.** "OTT video management platform" → OVP. "Subscriber backend" → SMS or IDM (ask which).
- **Verbatim source quotes** in the Features mapping table — gives the PM auditability for every extracted feature.
- **Do not write `project-context.md` without explicit `confirm`.**
- After writing, recommend `/onboard-project` for any DECISION REQUIRED items, and `/gap-analysis` for a fuller completeness pass against the whitelabel baseline.
