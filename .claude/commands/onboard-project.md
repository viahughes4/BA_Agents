# Skill: Onboard Project

Structured project onboarding. Creates or updates the per-engagement `project-context.md` for the current project. Conversational PM intake — not a form fill.

---

## Two-File Pattern

| File | Role |
|---|---|
| `OTT-requirements-reference.md` | **Catalog** — possible values, feature inventory, cert requirements. Read-only menu. |
| `project-context.md` | **Per-engagement** — values selected for *this* project. This skill writes here. |

Treat the reference catalog like `.env.example` and `project-context.md` like `.env`. The catalog is the menu of options; the project file records what was chosen.

---

## Process

### 1. Inspect current state
Invoke `/sub:context-loader`.
- 🚫 Not configured — greet the PM, explain the two-file pattern, propose walking through onboarding section by section.
- ⚠️ Partial — show what's confirmed and what's missing; ask whether to update existing fields or only fill gaps.
- ✅ Complete — ask whether the PM wants to update specific sections (new sprint, new platform added, scope change) instead of full re-onboarding.

### 2. Walk through sections one at a time
Do **not** dump every question at once. Confirm one section, write the proposed block, get a `confirm`, then move to the next. Use the catalog as a menu — when asking about a field, surface its possible values from `OTT-requirements-reference.md` so the PM knows the options without flipping documents.

Sections, in order:

#### A. Project Identity
- Project Title
- Client (legal entity / brand)
- Engagement type: Net-new build / Migration / Enhancement
- Jira project key
- Confluence space key
- Slack / Teams channel
- Active phase: Discovery / Design / MVP build / Phase 2 / Live ops
- Target launch date

#### B. Team
- PM (name + email)
- Tech Lead(s)
- Designers (Accedo / Client / Hybrid)
- QA Lead
- Client product owner
- Client tech lead
- Delivery Manager / Account

#### C. Commercial Dimensions
- Subscriber projections Y0–Y3
- Pricing tiers (T1 / T2 / T3 — name + price point)
- Regions launching (and post-launch expansion)
- Languages supported (e.g., EN, EN+FR, EN+ES+FR)
- Merchant of Record (Yes / No — and who)

#### D. Content Scope
- Live Events / PPV — Yes / No / N/A; if Yes: event types, count, free vs PPV split
- Live Channels — count
- FAST Channels — count, syndication Yes/No
- VOD Movies — count, average length
- VOD Series — count, average length
- VOD Clips — count, average length
- Encoding ladder — SD / HD / 4K combination, HDR variants (HDR10 / DV / HLG), audio variants

#### E. Platform Launch Plan
For each phase (MVP / Phase 2 / Phase 3):
- Specific platforms — pull from catalog list: iOS, Android, Web, Roku, Samsung Tizen, LG webOS, Fire TV, Apple TV, Vizio, HiSense, Meta Quest, AVP, Google TV, Fire Tablet, X1
- Target launch date for the phase
- Cert authority obligations per platform (App Store, Google Play, Amazon, Samsung Seller Office, LG Seller Lounge, Roku)

#### F. Systems
For each catalog system, capture **provider**, **owner** (Accedo / Client / Vendor), and the per-system selections from the catalog. Walk through:

- **OVP** — provider, video resolutions, HDR, ABR streaming, FAST creation/syndication
- **CDN** — provider, tokenized URL signing, annual bandwidth
- **DRM** — Widevine / FairPlay / PlayReady (which combination), provider (Axinom / EZDRM / BuyDRM / Verimatrix / Irdeto / PallyCon / Custom), license server pattern
- **Player SDK per platform** — AVPlayer (iOS) / ExoPlayer or Media3 (Android) / Bitmovin / THEOplayer / Shaka / Video.js / JW Player / Brightcove / Samsung PSDK / LG webOS pipeline / Amazon IVS — per platform in MVP scope
- **Manifest format(s)** — HLS fMP4 / DASH / Smooth Streaming
- **IDM** — provider, auth methods (OAuth2 / OIDC / Email+Password / SSO / Social Sign-In), Sign in with Apple if iOS, multi-profiles, parental controls, child profiles, concurrent stream limits, access model (Gated / Ungated / Mixed), first-line support
- **SMS** — provider, MOR, tax calculations, tier model, free trials, coupons, churn analytics
- **Recommendations** — provider (or internal/N/A), recommendation types
- **Analytics** — provider(s), QoS / QoE / Content Performance / User Behaviour
- **Advertising** — DAI strategy (CSAI / SSAI / Both / N/A), ad server, ad tag formats (VAST / VMAP / VPAID / SIMID), video / display / dynamic use cases, ad reporting, CMP
- **CMS / Metadata** — source
- **Search** — provider (or internal)
- **Front-End** — design provided by, QA, QA automation, unit testing + coverage target, form factors, Accedo Control remote configuration

When the PM hesitates ("TBD", "same as last project"), drill in:
- "Same as which engagement specifically?"
- "TBD because the OVP isn't selected, or because the contract is still open?"

Convert ambiguity into a `⚑ DECISION REQUIRED` block — never accept a vague final answer.

#### G. Compliance Triggers
- GDPR — Required / N/A
- CCPA — Required / N/A
- COPPA — Required / N/A
- ATT (iOS) — Required / N/A
- Accessibility — WCAG 2.1 AA / EAA / Section 508 / other
- Data residency — list of regions / None

#### H. Migration *(only if engagement type is Migration or Enhancement)*
- Existing apps and platforms
- User base size to migrate
- Account / token migration plan
- Subscription / billing migration plan
- Content catalog migration
- Cutover strategy: Big bang / Phased / Shadow run
- Migration timeline (months)
- Existing systems being reused
- Replacement systems
- Contract expiry on replaced systems (months)

#### I. Delivery Cadence
- **Delivery model:** Continuous / Weekly drop / Fixed milestones / Sprint-based
- **Client review rhythm:** e.g., weekly demo, biweekly steerco, milestone gates
- **Cert submission windows:** any fixed dates per platform (App Store review lag, Samsung TV Seller Office windows, etc.) that constrain the schedule
- **External dependency dates:** content rights live dates, marketing launch dates, hardware availability

##### Optional — Traditional Agile fields
Only fill these if the engagement explicitly runs on Scrum/SAFe and the client requires the artifacts:
- Sprint length (weeks)
- Current sprint name + dates
- Team velocity (story points per sprint)
- Ceremonies schedule

If the team is using AI-assisted code generation, treat velocity and points as advisory at best — story counts and decision-resolution rate are usually better signals.

#### J. Features in Scope *(optional but recommended)*
Walk through the catalog feature categories and mark each feature: MVP / Phase 2 / Phase 3 / Future / Out of Scope. Skip features that are clearly Out of Scope to keep this section concise. If the PM hasn't decomposed the SOW yet, recommend running `/sow-importer` first and skip this section.

### 3. Per-section interaction pattern
1. Ask the questions conversationally — surface the catalog's possible values inline so the PM doesn't have to look them up.
2. When answers reveal complexity, ask follow-ups (don't accept "TBD" as terminal).
3. Show the **proposed update block** (the markdown that will land in `project-context.md` for that section).
4. Ask: `confirm` to write, or describe edits.
5. On confirm, move to the next section.

### 4. Write `project-context.md`
After all sections are confirmed, write the file using the schema below.

### 5. Final summary
```
## Onboarding Complete — [Project Title]

### Confirmed Sections
[List with one-line summary per section]

### Open / Deferred Items (DECISION REQUIRED)
⚑ ...

### Recommended Next Steps
- [e.g., "Run /sow-importer to populate Features in Scope from the SOW PDF"]
- [e.g., "Run /gap-analysis on the PRD to validate completeness"]
- [e.g., "Schedule architecture call to confirm DRM provider"]
- [e.g., "Run /cert-checklist now that platform scope is defined"]
```

---

## `project-context.md` Schema

The file written by this skill must follow this schema. `/sow-importer` writes the same shape — keep them in sync.

```markdown
# Project Context — [Client] / [Project Title]

> Generated: [YYYY-MM-DD] by /onboard-project
> Reference catalog: OTT-requirements-reference.md

---

## Project Identity
- **Project Title:** ...
- **Client:** ...
- **Engagement Type:** Net-new build / Migration / Enhancement
- **Jira Project Key:** ...
- **Confluence Space:** ...
- **Slack / Teams Channel:** ...
- **Active Phase:** ...
- **Target Launch:** [date]

## Team
| Role | Name | Contact |
|---|---|---|
| PM | ... | ... |
| Tech Lead | ... | ... |
| Designer | ... | ... |
| QA Lead | ... | ... |
| Client Product Owner | ... | ... |
| Client Tech Lead | ... | ... |
| Delivery Manager | ... | ... |

## Commercial
- **Subscriber Projections:** Y0: X · Y1: X · Y2: X · Y3: X
- **Pricing Tiers:**
  - T1: [name, price]
  - T2: [name, price]
  - T3: [name, price]
- **Regions Launching:** [list]
- **Languages Supported:** [list]
- **Merchant of Record:** Yes / No — [who]

## Content Scope
- **Live Events / PPV:** Yes / No / N/A — [details]
- **Live Channels:** [count]
- **FAST Channels:** [count]
- **VOD Movies:** [count, avg length]
- **VOD Series:** [count, avg length]
- **VOD Clips:** [count, avg length]
- **Encoding Ladder:** SD / HD / 4K — HDR: [variants] — Audio: [variants]

## Platform Launch Plan
| Phase | Platforms | Target Date | Cert Authorities |
|---|---|---|---|
| MVP | ... | ... | ... |
| Phase 2 | ... | ... | ... |
| Phase 3 | ... | ... | ... |

## Systems

### OVP
- Provider · Owner · Resolutions · HDR · ABR · FAST creation

### CDN
- Provider · Owner · Tokenized signing · Annual bandwidth

### DRM
- Systems · Provider · License server pattern

### Player SDK
- Per-platform SDK · Manifest format(s)

### IDM
- Provider · Auth methods · Sign in with Apple · Multi-profiles · Parental controls · Child profiles · Concurrent streams · Access model

### SMS
- Provider · MOR · Tax · Tiers · Free trials · Coupons · Churn analytics

### Recommendations
- Provider · Types

### Analytics
- Provider(s) · QoS · QoE · Content perf · User behaviour

### Advertising
- DAI strategy · Ad server · Tag formats · Video / Display / Dynamic · Reporting · CMP

### CMS / Metadata
- Source

### Search
- Provider

### Front-End
- Design provided by · QA · QA automation · Unit testing + coverage · Form factors · Accedo Control

## Compliance Triggers
- GDPR / CCPA / COPPA / ATT — Required or N/A
- Accessibility standard
- Data residency regions

## Migration *(if applicable)*
- Existing apps / users · Account migration · Subscription migration · Content migration · Cutover · Timeline · Reused systems · Replaced systems · Contract expiry

## Delivery Cadence
- **Delivery Model:** Continuous / Weekly drop / Fixed milestones / Sprint-based
- **Client Review Rhythm:** ...
- **Cert Submission Windows:** ...
- **External Dependency Dates:** ...

### Optional — Traditional Agile (only if required by client)
- Sprint length · Current sprint · Velocity · Ceremonies

## Features in Scope
| Category | Feature | Phase |
|---|---|---|
| ... | ... | MVP / Phase 2 / Phase 3 / Future |

## Open Decisions
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...
```

---

## Quality Rules

- **One section at a time.** Never dump 80 questions in a single message.
- **Surface complexity.** "TBD" is not an acceptable final answer — turn it into a DECISION REQUIRED block with options.
- **Use the catalog as a menu.** Quote possible values from `OTT-requirements-reference.md` when asking the PM to choose. Don't make them recall option lists.
- **Validate selections against the catalog.** If the PM offers a value that isn't in the catalog's option list, either (a) confirm it's a custom case worth noting, or (b) ask a follow-up — don't silently accept.
- **Confirm before writing.** Never modify `project-context.md` without explicit `confirm` per section.
- **Preserve existing data.** When updating a partially populated file, do not clobber confirmed fields.
- **Don't fabricate defaults.** If the PM doesn't know, mark it `TBD` with a `DECISION REQUIRED`, not a plausible-sounding guess.
- The conversation should feel like a senior TBA leading intake — opinionated, prompting, never robotic.
