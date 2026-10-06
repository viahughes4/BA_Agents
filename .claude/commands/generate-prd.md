# Skill: Generate PRD

**Opt-in skill.** Run only when the client contract or Accedo deliverable plan requires a PRD artefact. For most engagements, `project-context.md` + per-feature stories + `/feature-discovery` reports are the source of truth — a separate PRD duplicates them and adds maintenance overhead.

Assembles a Product Requirements Document for the current engagement. Pulls project identity, scope, systems, and platform plan from `project-context.md`; KPI targets and cert requirements from `OTT-requirements-reference.md`. Features are expressed as feature-level user stories with the full Jira-ready implementation stories embedded beneath each one.

The PRD lives at `PRD.md` in the project root. Optionally publishes to Confluence.

Universal output rules apply — see CLAUDE.md → OUTPUT STYLE.

---

## Files

- **Reads:**
  - `project-context.md` — identity, scope, systems, platform plan, compliance (mandatory)
  - `OTT-requirements-reference.md` — KPI benchmark targets, cert requirements (catalog reference)
- **Optionally reads (PM-provided):**
  - Feature discovery reports (output of `/feature-discovery`) — populate feature descriptions and high-level user stories
  - Generated Jira-ready stories (output of `/generate-stories`) — embedded in full under each feature
- **Writes:**
  - `PRD.md` — the assembled document at project root
- **Optionally publishes:**
  - Confluence page via `/sub:confluence-write`

---

## Process

### 1. Confirm a PRD is actually needed
Ask the PM:
> Is a PRD required by the client contract or Accedo deliverable plan? Reply `yes` to proceed, `no` to skip — `project-context.md` + stories will serve as the source of truth instead.

Stop on `no`. Continue only on `yes`.

### 2. Load context
Invoke `/sub:context-loader`. If 🚫, stop and recommend `/onboard-project`. If ⚠️, proceed but flag missing context as ⚑ DECISION REQUIRED in the PRD.

### 3. Determine mode
- If `PRD.md` does **not** exist → **create** mode.
- If `PRD.md` **exists** → ask the PM:
  > Existing `PRD.md` found at v[X.Y]. Reply `update` to refresh from sources (manual-edit blocks preserved), `rebuild` to overwrite entirely, or `cancel`.
- In `update` mode, content between `<!-- manual:section-name -->` and `<!-- /manual -->` markers is preserved verbatim. Everything else is regenerated.

### 4. Accept supplemental content
Ask the PM, one at a time:

1. **Feature discovery reports** to embed?
   > Paste the reports, point to file paths, or reply `skip` to use catalog descriptions only.

2. **Generated Jira-ready stories** to embed in section 5?
   > Paste the `/generate-stories` output, point to file paths, or reply `skip` to list features without implementation detail.

3. **Additional content** (extra risks, NFRs, open decisions, custom sections)?
   > Paste content or reply `skip` to use only what's in `project-context.md`.

For each, accept whatever the PM provides — don't probe further.

### 5. Synthesize feature-level user stories
For every feature listed in `project-context.md → Features in Scope`:

| Source available | Use this for the feature-level user story |
|---|---|
| Feature discovery report exists | The discovery report's *User Need Statement* verbatim |
| No discovery, catalog match | Synthesize "As a [persona], I want [catalog feature], so that [outcome]" using project's user personas + catalog feature description |
| Bespoke feature, no catalog match | Ask the PM for the user story (or flag for `/feature-discovery` first) |

The persona is inferred from `project-context.md` — typically *subscriber* if access is gated; *free-tier user* for AVOD content; *child profile* for kids content; *account owner* for parental controls.

### 6. Assemble the PRD
Use the structure in the next section. Pull verbatim where possible. Do not invent values.

### 7. Show draft
Display the full assembled content. Ask:
> Reply `confirm` to write to `PRD.md`, `cancel` to abort, or describe edits.

### 8. Write
On `confirm`:
- **Create mode** — write the new file. Set version to `0.1`.
- **Update mode** — replace regenerated sections; preserve manual blocks; bump minor version (e.g. `0.3 → 0.4`).
- **Rebuild mode** — overwrite entire file; reset version to `0.1` unless PM specifies.

### 9. Offer Confluence publish
After write:
> Publish to Confluence space `[KEY]` under `[parent page or — for root]`? Reply `confirm` to publish via `/sub:confluence-write`, or `cancel` to keep local.

If `confirm`, invoke `/sub:confluence-write` with the rendered PRD body.

---

## PRD Structure

```markdown
# Product Requirements Document — [Client] / [Project]
> Version: [X.Y] | Generated: [date] by /generate-prd
> Sources: project-context.md · OTT-requirements-reference.md
> Manual edits preserved within `<!-- manual:section-name -->` ... `<!-- /manual -->` blocks.

---

## 1. Executive Summary

### 1.1 What we're building
[One paragraph derived from engagement type, platform plan, and content scope. E.g., "A net-new whitelabel OTT app for [Client] launching in [Q] across iOS, Android, Fire TV, and Apple TV, supporting live channels, FAST, and a VOD catalogue of [N] movies and [N] series."]

### 1.2 Audience
[Inferred from access model + regions + languages + content scope. E.g., "Subscribers and free-tier users in North America, with content delivered in English, Spanish, and French. Child profiles supported via parental controls."]

### 1.3 Driver
[From SOW context if available; otherwise "Aligned to [Client] launch / migration target [date]."]

<!-- manual:executive-summary -->
<!-- /manual -->

---

## 2. Goals & Success Metrics

### 2.1 Business Goals
- Subscriber projections — Y0: [X] · Y1: [X] · Y2: [X] · Y3: [X]
- Pricing tiers and target subscriber mix
- Regional / market expansion plan

### 2.2 Product Goals
[Inferred. E.g., "Deliver a 10-foot-friendly, low-latency live channel experience with parity across mobile and CTV." Adapt based on access model, multi-profile, content types.]

### 2.3 KPI Targets
*From `OTT-requirements-reference.md` → Performance KPIs, narrowed to platforms in launch plan.*

| KPI | Target | Applicable Platforms |
|---|---|---|
| Sign-in to Homepage (re-launch) | ≤ 5s | All |
| Sign-in to Playback | ≤ 60s end-to-end | All |
| Home Screen — tvOS | ≤ 6s | Apple TV |
| Home Screen — Samsung / Fire TV / Roku | ≤ 12s | Samsung / Fire TV / Roku |
| ... | ... | ... |

<!-- manual:kpi-targets -->
<!-- /manual -->

---

## 3. Scope

### 3.1 In Scope
[Itemised list pulled from `project-context.md → Features in Scope` and `Platform Launch Plan`.]

### 3.2 Out of Scope
[Explicit exclusions from `project-context.md` and any `/sow-importer` extracted exclusions.]

### 3.3 Phased Launch Plan
| Phase | Platforms | Target Date | Cert Authorities | Feature Highlights |
|---|---|---|---|---|
| MVP | ... | ... | ... | ... |
| Phase 2 | ... | ... | ... | ... |
| Phase 3 | ... | ... | ... | ... |

<!-- manual:scope -->
<!-- /manual -->

---

## 4. User Personas

[Inferred from access model, profile types, parental controls, and content scope. Default personas:]

- **Subscriber** — paying user with full catalogue access
- **Free-tier user** — if FAST / AVOD in scope
- **Child profile** — if Multi-Profiles + Child Profiles + Parental Controls
- **Account owner** — if Parental Controls (managing PINs and child profiles)
- **Migrating user** — if engagement type is Migration

[Adjust based on project-context specifics. Omit personas not implied by scope.]

<!-- manual:personas -->
<!-- /manual -->

---

## 5. Functional Requirements

*Each feature is presented as a feature-level user story, followed by its full Jira-ready implementation stories. Stories embedded verbatim from `/generate-stories` output where provided.*

### 5.1 [Catalog Category — e.g., Customer Management]

#### 5.1.1 [Feature Name — e.g., Multi-Profiles]
> As a subscriber, I want to create multiple profiles within my account so household members get personalised experiences.

- **Phase:** MVP / Phase 2 / Phase 3 / Future
- **Source:** Onboarding / SOW (clause / page ref) / Discovery
- **Catalog effort range:** [from catalog typical effort]
- **Discovery report:** [link or "not yet performed"]
- **Systems touched:** [from discovery, or inferred]

##### Implementation Stories

[Full Jira-ready stories embedded verbatim. Each story includes:
 - Title
 - User Story line
 - Platform Scope
 - Labels
 - Dependencies
 - Acceptance Criteria
 - Platform-Specific Considerations (if present)
 - Non-Functional Requirements
 - Test Considerations (if present)
 - Design Reference
 - Open Questions (if present)]

---

#### 5.1.2 [Next feature]
> [User story]

[... same structure ...]

---

### 5.2 [Next category — e.g., Video Playback]

[... features ...]

[Continue across all catalog categories with features in scope.]

<!-- manual:functional-requirements-additions -->
<!-- /manual -->

---

## 6. Non-Functional Requirements

### 6.1 Performance
[Reference KPI targets in section 2.3 + any NFRs supplied by PM]

### 6.2 Accessibility
[Per-platform — VoiceOver (iOS), TalkBack (Android), D-pad accessibility (Fire TV / Samsung / LG / Apple TV / Roku), WCAG 2.1 AA (Web). Pull standard from `project-context.md → Compliance Triggers`.]

### 6.3 Localization
[Languages from `project-context.md`; RTL support flag if applicable; metadata + UI string scope.]

### 6.4 Security & Compliance
- **DRM strategy:** [systems + provider from `project-context.md`]
- **Compliance triggers:** [GDPR / CCPA / COPPA / ATT — only those marked Required]
- **Data residency:** [regions]
- **Sign in with Apple:** Required if any third-party auth is offered on iOS / N/A

### 6.5 Observability
- **Analytics provider(s):** [from `project-context.md`]
- **Event categories:** QoS / QoE / Content Performance / User Behaviour (only those marked Yes)
- **Error reporting expectations:** [if specified by PM]

<!-- manual:nfrs -->
<!-- /manual -->

---

## 7. Technical Architecture

### 7.1 Platform Stack
| Platform | Framework | Player SDK | DRM |
|---|---|---|---|
| iOS | Swift / AVPlayer | ... | FairPlay |
| Android | Kotlin | ... | Widevine |
| Web | JavaScript | ... | Widevine (Chrome/Edge) + FairPlay (Safari) |
| Fire TV | Kotlin | ... | Widevine |
| Samsung Tizen | JavaScript / PSDK | ... | PSDK (Widevine or PlayReady) |
| LG webOS | JavaScript / Lightning | ... | webOS pipeline |
| Apple TV | Swift / AVPlayer | AVPlayer | FairPlay |
| Roku | BrightScript | Native | — |

[Only include rows for platforms in launch plan.]

### 7.2 Backend Systems
| System | Provider | Owner | Notes |
|---|---|---|---|
| OVP | ... | Accedo / Client / Vendor | ... |
| CDN | ... | ... | Tokenized URL signing: Yes/No |
| IDM | ... | ... | Auth method, concurrent stream limit |
| SMS | ... | ... | MOR Yes/No |
| Recommendations | ... | ... | Types |
| Analytics | ... | ... | Providers |
| Advertising | ... | ... | DAI strategy: CSAI / SSAI / Both / N/A |
| CMS | ... | ... | ... |
| Search | ... | ... | ... |

### 7.3 Cert Authorities
| Platform | Cert Authority | Submission Window |
|---|---|---|
| iOS / Apple TV | Apple App Store Review | ... |
| Android | Google Play Console | ... |
| Fire TV | Amazon Appstore | ... |
| Samsung Tizen | Samsung TV Seller Office | ... |
| LG webOS | LG Seller Lounge | ... |
| Roku | Roku Channel Store | ... |

<!-- manual:architecture -->
<!-- /manual -->

---

## 8. Constraints & Assumptions

[Pull from `project-context.md → Migration` (if applicable) + Delivery Cadence + Team composition + Regulatory deadlines + any PM-supplied constraints.]

<!-- manual:constraints -->
<!-- /manual -->

---

## 9. Open Decisions

[Aggregate every `⚑ DECISION REQUIRED` block from:
 - `project-context.md` Open Decisions section
 - Embedded feature discovery reports
 - Embedded user stories' Open Questions
 - PM-supplied additions]

⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...

<!-- manual:decisions -->
<!-- /manual -->

---

## 10. Milestones & Timeline

| Milestone | Target Date | Status | Owner |
|---|---|---|---|
| MVP launch | ... | On Track / At Risk / Delayed | ... |
| Phase 2 launch | ... | ... | ... |
| Cert submissions ([per platform]) | ... | ... | ... |

[From `project-context.md → Platform Launch Plan` + Delivery Cadence (cert submission windows, external dependency dates, client review rhythm).]

<!-- manual:milestones -->
<!-- /manual -->

---

## 11. Risks

[Auto-detected risks:
 - Platform never delivered before by Accedo (cross-check launch plan against typical delivery mix)
 - Aggressive timeline (cross-check launch dates vs catalog effort + cert submission windows + external dependency dates)
 - Missing system selections (cross-check `_context-loader` ⚠️ output)
 - Unresolved DECISION REQUIREDs blocking implementation
 - Cert dependencies on systems not yet integrated
 - High volume of unresolved decisions in active scope (signals spec instability — particularly costly under AI codegen)]

[Plus PM-supplied risks.]

| # | Risk | Likelihood | Impact | Trend | Mitigation | Owner |
|---|---|---|---|---|---|---|

<!-- manual:risks -->
<!-- /manual -->
```

---

## Quality Rules

- **Derived, not invented.** Every claim in the PRD traces back to `project-context.md`, `OTT-requirements-reference.md`, or PM-supplied content. If a section can't be filled from sources, surface it as ⚑ DECISION REQUIRED rather than fabricating defaults.
- **Gaps in `project-context.md` become gaps in the PRD.** Do not silently fill them — flag them.
- **Feature-level user stories are conversions, not analyses.** Use the feature-discovery User Need Statement when present; otherwise synthesize from catalog description + project user persona. Don't invent new behaviours.
- **Embedded Jira stories are verbatim.** Do not paraphrase or summarize `/generate-stories` output. If the PM supplies stories that don't match a feature in scope, flag it — don't quietly attach them.
- **Manual blocks preserved.** In `update` mode, every content between `<!-- manual:name -->` and `<!-- /manual -->` is preserved exactly. Auto-generated content surrounding those blocks may change; manual content does not.
- **Versioning.** Create → `0.1`. Update → minor bump. Rebuild → reset to `0.1` unless PM overrides. PM marks the PRD as `1.0` only when approved for delivery (this skill does not auto-promote to 1.0).
- **Show draft before writing. Confirm before publishing.** Same approval gate pattern as the rest of the agent.
- **Don't duplicate `/generate-stories` work.** This skill assembles existing artefacts; it doesn't generate new stories. If the PM hasn't run `/generate-stories` for features in scope, surface that as a recommendation rather than fabricating ACs.
- **Don't fabricate risks.** Auto-detection of risks is OK only when the signal is concrete (e.g., a feature has no Player SDK named in `project-context.md`). Vague risks belong in PM-supplied content.
