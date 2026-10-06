---
name: spec-to-tickets
description: Use this skill when the user wants to go from a raw feature idea directly to Jira-ready tickets in one structured pipeline. Triggers on "spec to tickets", "run the pipeline", "interview me", "build tickets from scratch", "feature pipeline", or any request to convert a feature description into tickets without separately running feature-discovery and generate-stories. Runs a 3-phase pipeline - interview -> feature brief -> tickets with dependency map - with two approval gates before any Jira creation. Lightweight alternative to running full discovery; use /feature-discovery for deep scoping of complex or unknown features.
version: 1.0.0
---

# Skill: Spec-to-Tickets Pipeline

Convert a raw feature idea into Jira-ready tickets with dependency mapping in three phases. Two approval gates: one before writing, one before creating in Jira. Nothing goes to Jira without explicit user approval.

This is the **lightweight middle path**:
- Faster than `/feature-discovery` (no 20-dimension gap analysis, no catalog search)
- More structured than ad-hoc ticket creation (interview -> spec -> brief -> tickets -> dependency map)
- Use `/feature-discovery` when the feature is complex, poorly understood, or touches DRM / auth / payment across multiple platforms
- Use `/generate-stories` standalone when a feature brief already exists

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load and validate project context. If 🚫, recommend `/onboard-project` and stop. If ⚠️, proceed - treat every missing critical field as an `⚑ Open` block inline within relevant outputs.

---

## Phase 1: Interview

Ask all questions in a **single block**. Do not ask one at a time.

> **Let's build this out. Answer as much as you know - skip anything that isn't decided yet.**
>
> 1. **What is the feature or change?** *(1-2 sentences: what it does and why it matters)*
> 2. **Which platforms does it affect?** *(Samsung Tizen, LG webOS, Xbox UWP, X1/Ignite, Xumo - or "all")*
> 3. **What does success look like for a user?** *(describe the experience when it works correctly)*
> 4. **Are there known prerequisites?** *(anything that must exist or be completed before this can be built - another ticket, an API, a design deliverable)*
> 5. **Is there a design reference?** *(Figma link, or "no design yet")*
> 6. **Any constraints?** *(deadline, technical limitation, out-of-scope items - or "none")*
> 7. **Any related Jira tickets or epics?** *(optional - CBC-XXX keys, or leave blank)*

Wait for answers. Do not proceed until the user has responded to at least questions 1-3.

---

## ⛔ GATE 1 - Spec Review

From the interview answers, generate and display the Spec. Do not proceed to Phase 2 until the user confirms.

**Spec format:**

```
## Spec: [Feature Name]

**Feature:** [1-2 sentence description from Q1]
**Platforms:** [confirmed list from Q2]
**User success:** [from Q3]
**Prerequisites:** [from Q4, or "None stated"]
**Design:** [Figma link from Q5, or "TBD"]
**Constraints:** [from Q6, or "None stated"]
**Related tickets:** [from Q7, or "None"]
```

After displaying the spec, ask:

> **Does this spec look right? I'll generate the feature brief and tickets once you confirm - or tell me what to change.**

**Do not proceed to Phase 2 until the user explicitly confirms** the spec is correct. Accept "yes", "looks good", "confirmed", "proceed", or equivalent. If changes are requested, revise and re-present.

---

## Phase 2: Feature Brief

Auto-generate from the confirmed spec. No additional questions unless a DECISION REQUIRED block must be raised.

**Feature brief format:**

```
## Feature Brief: [Feature Name]

**Goal:** [1-sentence from spec: what this delivers and why]
**Platforms:** [confirmed list]
**User success:** [from spec Q3]
**Systems touched:** [inferred - BFF, CMS, Auth, Player, Analytics, etc. - flag as assumed if not confirmed]
**Design:** [Figma link or "TBD"]
**Constraints:** [from spec, or "None"]
**Dependencies:** [from spec Q4 + any inferred system dependencies]

**Scope:**
- In: [what's included based on the spec]
- Out: [what's explicitly excluded - infer from constraints + common scope traps for this feature type]
- Deferred: [anything that should be a follow-up ticket rather than part of this batch]

**Effort estimate:** S / M / L
*(S = 1-2 days, single component, no new APIs; M = 3-7 days, new components + 1-2 APIs; L = 1-3 weeks, multi-system, complex state, design unknowns. Add 20-30% for DRM, auth, or payment.)*

**Risks:** [top 1-3 - prioritize by I×L; use 🔴 Critical (7-9) · 🟠 High (4-6) · 🟡 Medium (2-3) · 🟢 Low (1)]

⚑ DECISION REQUIRED: [surface any unresolved question that blocks ticket writing - if none, omit this block entirely]
```

**If any DECISION REQUIRED blocks exist:** Pause. Surface them clearly. Wait for resolution before proceeding to Phase 3.

**If no DECISION REQUIRED blocks:** Proceed automatically to Phase 3 - no gate between brief and tickets.

---

## Phase 3: Tickets with Dependency Map

### Story generation

Break the feature brief into tickets using `generate-stories` tier logic:

| Tier | Signals |
|---|---|
| **Minimal** | Single platform, no system boundary, no user input beyond a tap, no playback / auth / payment, no multi-locale UI |
| **Standard** | Single platform, one system boundary OR meaningful user input, no playback / auth / payment cross-cutting concerns |
| **Heavy** | Playback / auth / payment / multi-platform / multi-system / multi-locale UI |

Each ticket follows the appropriate tier template. Omit empty sections - do not leave "N/A" or placeholder text.

**All generate-stories quality rules apply:**
- Write AC in present tense: "The component should..." / "The screen should..."
- Describe observable behavior and appearance - not implementation
- Cover edge states: include at least one AC for loading, error, and empty states where applicable
- No vague verbs: "works correctly", "handles gracefully", "as expected", "appropriately", "if needed", "properly"
- Each AC scenario must be independently verifiable by QA
- Minimal: max 2 scenarios. Standard: max 8-10 AC statements. Heavy: multiple scenarios with error/edge/alternate paths required
- Use `CBC-DRAFT` as the Jira key placeholder - never append a number
- Label generation: one platform label per platform in scope + one domain label (lowercase, hyphen-separated)
- Design field: "no design needed" only for non-visual stories; any UI story requires a Figma link or raises an ⚑ Open block

**Playback gate:** For any playback story, note Player SDK, DRM provider, license server pattern, CDN, and manifest format. If unconfirmed, flag in Open block.

### Ticket templates

#### Minimal

```
## [Story Title]
> CBC-DRAFT | Epic: [epic name from loaded project context, or TBD]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [single platform]
**Labels:** [platform label(s) + domain label]

**Acceptance Criteria**

Scenario 1: [Happy path]
Given [precondition with concrete data]
When [single action]
Then [observable outcome]

[Scenario 2 only if happy path doesn't cover an obvious failure or alternate.]

**Design:** [Figma link, or "no design needed"]
```

#### Standard

```
## [Story Title]
> CBC-DRAFT | Epic: [epic name from loaded project context, or TBD]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [list]
**Labels:** [platform label(s) + domain label]
**Dependencies:** [CBC-DRAFT-N] *(omit if none)*

**Integration Contracts** *(omit if no system boundary)*
- API endpoints: [linked spec]
- Events: [name + parameter schema]
- Auth assumptions: [token type, refresh behaviour]

**Acceptance Criteria**

Scenario 1: [Happy path]
Given [concrete data]
When [single action]
Then [observable outcome with concrete expected value]

[Add scenarios for error/failure if crossing a boundary; edge cases for obvious boundaries; alternate paths for entitlement / role / locale variation.]

**NFRs** *(include only sub-bullets that diverge from loaded project context defaults - omit section if all defaults apply)*
- Performance: [Numeric SLA]
- Accessibility: [non-default specifics]
- Localization: [if multilingual]
- Error handling: [timeouts, retry, fallback]
- Analytics: [event name + parameters]

**Design:** [Figma link, or "no design needed"]

**Open** *(omit if none)*
⚑ Open: [one-liner] - defaulting to [X] unless changed.
```

#### Heavy

```
## [Story Title]
> CBC-DRAFT | Epic: [epic name from loaded project context, or TBD]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform Scope:** [list - include per-platform notes only when behaviour diverges]
**Labels:** [platform label(s) + domain label]
**Dependencies:** [CBC-DRAFT-N] *(omit if none)*

**Integration Contracts**
- API endpoints invoked: [linked spec]
- Events emitted/consumed: [full parameter schema]
- SDK / library versions: [e.g., AVPlayer iOS 17+, ExoPlayer 2.19+]
- Auth / token assumptions: [bearer, refresh, expiry behaviour]
- DRM / license server *(playback only)*: [provider, endpoint pattern, tokenization]

**Acceptance Criteria**

Scenario 1: [Happy path]
Given [concrete data]
When [single action]
Then [observable outcome with concrete expected value]

Scenario 2: [Error / failure - required for boundary stories]
...

Scenario 3: [Edge case - required when boundaries are obvious]
...

Scenario 4: [Alternate path - required when entitlement / role / locale changes behaviour]
...

**Platform-Specific Considerations** *(only for behaviours that diverge per platform)*

**NFRs**
- Performance: [Numeric SLA per platform if relevant]
- Accessibility: [VoiceOver, TalkBack, D-pad focus, captions, WCAG]
- Localization: [if multilingual]
- Error handling: [timeouts, retry, fallback]
- Analytics: [event names + parameters]
- Security / privacy: [token handling, PII, ATT/GDPR]

**Test Considerations** *(only when there are non-obvious mocks or integration boundaries)*

**Design:** [Figma link, or "design required before implementation"]

**Open** *(omit if none)*
⚑ Open: [one-liner]
What: ...
Why it matters: ...
Options: ...
Recommended default: ...
```

---

### Dependency Map

After generating all tickets, run a dependency pass. For each ticket, evaluate:
- Does it depend on another ticket in this batch? (e.g. auth must exist before a gated screen can be built)
- Does it depend on something external? (client BE, design delivery, another team, a 3rd-party API)
- Can it be built in parallel with any other ticket?

**Always output this table, even for small batches:**

```
## Ticket Batch - Dependency Map

| Ticket | Title | Deps | Status |
|--------|-------|------|--------|
| T1 | [Title] | None | ✅ Build first |
| T2 | [Title] | None | ✅ Parallel with T1 |
| T3 | [Title] | T1 | ⚠️ Blocked: [reason - e.g., auth token must exist] |
| T4 | [Title] | External | ⚠️ External block: [what - e.g., client design delivery] |

**Suggested sequence:**
  Wave 1 (parallel): T1, T2
  Wave 2: T3
  Wave 2: T4 - flag as externally blocked in Jira

**External blocks to flag in Jira:** [list, or "None"]
```

---

### Self-Validate

Before presenting tickets + dependency map, run the following checks on each ticket. Do not emit a per-ticket pass table - emit an Issues appendix only for failures found.

**Structural checks:**
- 🔴 AC uses a banned vague verb -> flag with specific fix suggestion
- 🔴 Platform scope is absent or unclear -> flag
- 🔴 Sub-task has no Parent key -> flag (do not emit a parent-less sub-task)
- 🟡 Open question not resolved from the interview answers -> flag
- 🟡 Design field required for a UI story but no Figma link provided -> flag
- 🟡 Integration contract section empty for a story that crosses a system boundary -> flag

**Issues appendix format (emit only when failures exist):**

```
### Issues Found
- T2 [Story Title]: AC scenario 2 uses "handles gracefully" - vague verb. Rewrite: "When the API returns a 5xx error, the screen should display [specific message] and offer a retry button."
- T3 [Story Title]: Integration contract missing - story calls BFF auth endpoint but no endpoint linked.
```

---

## ⛔ GATE 2 - Jira Approval

After presenting all tickets + dependency map + issues appendix (if any):

> **Ready to create in Jira?**
>
> I'll create all [N] tickets with `blocks` / `blocked-by` links pre-mapped based on the dependency map above.
>
> Confirm with "create them" or "yes" - or request changes first.

**Do not call any Jira create tool until the user explicitly approves.** Accept "yes", "create them", "looks good", "confirmed", or equivalent. If changes are requested, revise and re-present.

---

## Jira Creation (Post-Gate 2)

Once the user approves, create all tickets in sequence respecting the dependency map:

1. Create Wave 1 tickets first (no deps)
2. Create Wave 2+ tickets after their dependencies exist, so real Jira keys are available for linking
3. For each ticket:
   - Set `projectKey` (from loaded project context - already available from context-loader)
   - Set `issueTypeName` (Story / Task / Sub-task as appropriate)
   - Set `parent` for sub-tasks (required - never omit)
   - Set `summary`, `description`, `labels` (if supported)
   - **ADF patch (always required):** Immediately after each `createJiraIssue` call, call `editJiraIssue` with the same ADF description unconditionally. The create tool silently drops ADF content - always patch via edit or the description will be empty.
   - Always include `created-via-claude` in the labels array, in addition to the platform and domain labels from the ticket template. Example: labels: [created-via-claude, samsung-tizen, playback]
   - Set `blocks` / `blocked-by` links using the dependency map (use `createIssueLink` after creation if needed)
4. If a ticket creation call fails, skip that ticket, note the failure in the final summary table with the error reason, and continue creating remaining tickets. Do not abort the entire batch on a single failure. After the batch completes, list any failures and prompt the user to retry them.
5. Return all created ticket keys and URLs in a summary table

**Dependency link format:**
- Internal dep -> use `createIssueLink` with type "is blocked by" / "blocks" between the real ticket keys
- External dep -> add an `⚑ Open` block in the ticket description: `⚑ External block: [description - e.g., client BE delivery of [endpoint] required before this ticket can start]`

---

## Quality Rules

- **Drafts only until Gate 2.** Never auto-create in Jira.
- **Omit empty sections.** No "-" or "N/A" placeholders.
- **Dependency map is mandatory.** Even for single-ticket batches (trivially: "No deps - build freely").
- **Issues appendix only when there are failures.** Don't emit a clean-pass table.
- **Do not re-ask questions already answered in the interview.** Pull from the spec.
- **Do not run a full 20-dimension gap analysis.** This is the lightweight path. Flag critical gaps only.
- **CMP/Didomi reminder (surface to PM, do not add to tickets):** Before finalizing the ticket batch, check: do any tickets touch navigation, modals, overlays, player controls, or consent flows? If yes, add a callout at the end: "Reminder: one or more tickets may interact with CMP/Didomi. Confirm CMP navigation regression is in QA scope before dev picks up."
- **Do not replace /feature-discovery.** If the feature is complex, multi-system, or touches DRM/auth/payment across platforms, recommend `/feature-discovery` at Gate 1 or Phase 2.
- **Gate 1 and Gate 2 are hard stops.** Do not skip or combine them.
- **Jira key format:** Use `CBC-DRAFT` as placeholder. Never append a number. This is a draft, not a real issue ID.

---

> **Skill verification:** Please ensure that the skill spec-to-tickets SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
