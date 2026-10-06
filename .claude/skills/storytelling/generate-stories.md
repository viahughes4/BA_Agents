---
name: generate-stories
description: Use this skill when the user wants to generate user stories for implementation. Triggers on "generate stories", "write stories", "create stories for this feature", "write user stories", "stories for sprint", "draft stories", "break this into stories", "story breakdown", "user story for [feature]", "ticket this out as stories", "decompose into stories", or any request to produce Jira-ready user stories from a feature description or discovery report.
version: 1.0.0
---

# Skill: Generate User Stories

Generate Jira-ready stories that AI codegen can build from and human QA can verify against. Drafts only - never auto-create.

Self-validates against AI-Buildability checks. Tiers the output to story complexity: trivial stories get a 1-screen draft, heavy stories get the full template.

Universal output rules (concise language, omit empty sections, compact DECISION REQUIRED) apply - see CLAUDE.md -> OUTPUT STYLE.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load and validate project context. If 🚫, recommend `/onboard-project` and stop. If ⚠️, proceed - treat every missing critical field (Jira key, platform, API contracts, NFR defaults) as an `⚑ Open` block inline within the story.

### Optional: Slack context
If the user has provided a Slack message, thread, or channel link alongside their input, read `.claude/sub/slack-context-extractor.md` and follow its process in full. Merge the returned context block into the working context before proceeding. If no Slack input is provided, skip this step. If the slack-context-extractor sub-routine is unavailable or returns no usable context, skip this step and proceed with the feature description provided by the user.

### 2. Require prior discovery for non-trivial work
If the request touches multiple platforms, multiple systems, or playback / auth / monetization, and no Feature Discovery Report exists, stop and recommend `/feature-discovery` first.

### 3. Clarification gate
If platform scope, user need, design reference (UI), or integration contracts (system-boundary stories) are missing, ask **at most 3** questions. 

### 4. Playback gate
For any playback story, confirm Player SDK, DRM provider, license server pattern, CDN, manifest format. If unconfirmed, note this in open questions at the end of the ticket

### 5. Detect tier per story
Auto-classify each proposed story:

| Tier | Signals |
|---|---|
| **Minimal** | Single platform, no system boundary, no user input beyond a tap, no playback / auth / payment, no multi-locale UI |
| **Standard** | Single platform, one system boundary OR meaningful user input, no playback / auth / payment cross-cutting concerns |
| **Heavy** | Playback / auth / payment / multi-platform / multi-system / multi-locale UI |

PM may override the tier on any story.

### 6. Output story index (Standard / Heavy batches only)
For batches with at least one Standard or Heavy story:

```
## Proposed Stories ([N])
1. [Title] - [tier] - [platforms]
```

Skip the index for an all-Minimal batch - go straight to the stories.

### 7. Generate each story per its tier
Use the per-tier template. Don't pad sections - omit them when empty.

### 8. Self-validate
Internally run AI-Buildability checks against each story (concrete spec, no embedded decisions, single deployable outcome, platform/SDK explicit, integration contracts referenced for boundary stories, ACs executable as tests, design resolved for UI stories).

Classify each failure by severity:
- 🔴 Blocker: AC uses a banned vague verb, platform scope missing, integration contract absent for a boundary story, sub-task has no parent key
- 🟡 Warning: open question unresolved, design field required but no Figma link, NFR divergence not documented

- All clean -> emit stories only.
- Any failure -> emit stories + a short **Issues found** appendix listing only the failed checks per story with severity prefix.

Don't emit a per-story pass-table. Don't emit a verdict block. The story itself, plus the issue list when issues exist, is the artefact.

### 9. Dependency map (2+ story batches only)
For any batch of 2 or more stories, run a dependency pass after all stories are generated. For each story, evaluate:
- Does it depend on another story in this batch? (e.g. an auth story must exist before a gated screen can be built)
- Does it depend on something external? (BE delivery, design, another team)
- Can it be built in parallel with any other story?

Output the map immediately after the stories:

```
## Story Batch - Dependency Map

| Story | Title | Deps | Status |
|-------|-------|------|--------|
| S1 | [Title] | None | ✅ Build first |
| S2 | [Title] | None | ✅ Parallel with S1 |
| S3 | [Title] | S1 | ⚠️ Blocked by S1: [reason] |
| S4 | [Title] | External | ⚠️ External block: [what] |

Suggested sequence:
  Wave 1 (parallel): S1, S2
  Wave 2: S3
  Wave 2: S4 - externally blocked, flag before sprint pickup

External blocks to resolve: [list, or "None"]
```

Skip the dependency map for single-story outputs.

### 10. Save and export
If the user asks to save the generated stories, write them to `product-development/product/customers/accounts/[client]/sprints/` using the naming convention `YYYY-MM-DD-[feature-name]-stories.md`. Then run:

```
python3 scripts/md-to-html.py <path-to-saved-file>
```

Report the HTML path to the user so they know it exists.

---

## Templates

### Minimal tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from the loaded project context, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [single platform]
**Labels:** [platform label(s) + domain label - see label generation rule]

**Acceptance Criteria**

[One plain-English summary sentence stating the overall expected behaviour. Should be readable at a glance without needing to read the scenarios.]

Scenario 1: [Happy path]
Given [precondition with concrete data]
When [single action]
Then [observable outcome]

[Add Scenario 2 only if the happy path doesn't cover the obvious failure or alternate.]

**Design:** [Figma frame link OR "no design needed" - see design rule]
```

Drop NFRs, Integration Contracts, Test Considerations, and Platform-Specific Considerations entirely for Minimal stories. Defaults inherited from the loaded project context are sufficient.

### Standard tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from the loaded project context, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [list]
**Labels:** [platform label(s) + domain label - see label generation rule]
**Dependencies:** [PROJ-XXX] *(omit if none)*

**Integration Contracts** *(omit if no system boundary)*
- API endpoints: [linked spec]
- Events: [name + parameter schema]
- Auth assumptions: [token type, refresh behaviour]

**Acceptance Criteria**

[One plain-English summary sentence stating the overall expected behaviour. Should be readable at a glance without needing to read the scenarios.]

Scenario 1: [Happy path]
Given [concrete data]
When [single action]
Then [observable outcome with concrete expected value]

[Add scenarios for materially distinct behaviours: error/failure if the story crosses a boundary or accepts user input; edge cases for obvious boundaries (empty / max / network loss); alternate paths for entitlement / role / locale variation - especially user states (Guest, Member, Unsubscribed, Not Eligible, etc.). No vague verbs ("works correctly", "handles gracefully", "as expected"). Each scenario must be executable as a test.]

**NFRs** *(include only sub-bullets that diverge from defaults in the loaded project context - omit the section if all defaults apply; include a sub-bullet if the loaded project context has no value for it)*
- Performance: [Numeric SLA]
- Accessibility: [non-default specifics]
- Localization: [if multilingual]
- Error handling: [timeouts, retry, fallback]
- Analytics: [event name + parameters]

**Design:** [Figma frame link OR "no design needed" - see design rule]

**Open** *(omit if none)*
⚑ Open: [one-liner] - defaulting to [X] unless changed.
```

### Heavy tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from the loaded project context, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform Scope:** [list - include per-platform notes only when behaviour diverges]

**Labels:** [platform label(s) + domain label - see label generation rule]
**Dependencies:** [PROJ-XXX] *(omit if none)*

**Integration Contracts**
- API endpoints invoked: [linked spec]
- Events emitted/consumed: [full parameter schema]
- SDK / library versions: [e.g., AVPlayer iOS 17+, ExoPlayer 2.19+]
- Auth / token assumptions: [bearer, refresh, expiry behaviour]
- DRM / license server *(playback only)*: [provider, endpoint pattern, tokenization]

**Acceptance Criteria**

[One plain-English summary sentence stating the overall expected behaviour.]

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

[No vague verbs. Each scenario executable as a test.]

**Platform-Specific Considerations** *(only for behaviours that diverge per platform)*

**NFRs**
- Performance: [Numeric SLA per platform if relevant]
- Accessibility: [VoiceOver, TalkBack, D-pad focus, captions, WCAG]
- Localization: [if multilingual]
- Error handling: [timeouts, retry, fallback]
- Analytics: [event names + parameters]
- Security / privacy: [token handling, PII, ATT/GDPR]

**Test Considerations** *(only when there are non-obvious mocks or integration boundaries)*

**Design:** [Figma frame link OR "design required before implementation"]

**Open** *(omit if none - use short form for simple decisions; include What/Options/Recommended only when decision affects scope, cost, cert, or architecture)*
⚑ Open - [one-liner]
What: ...
Why it matters: ...
Options: ...
Recommended default: ...
```

---

## Issues Appendix *(emit only when self-validation finds failures)*

```
### Issues Found
- 🔴 [Story Title]: [specific failure - e.g., "AC scenario 2 uses 'handles gracefully' (vague verb) - rewrite: 'When the API returns 5xx, the screen should display [message] and offer a retry button'"]
- 🔴 [Story Title]: [specific failure - e.g., "Integration contract missing - story calls auth API but no endpoint linked"]
- 🟡 [Story Title]: [specific warning - e.g., "Figma link absent for a UI story - raise with design before dev pickup"]
```

🔴 = blocker, must be resolved before dev pickup. 🟡 = warning, should be resolved but does not block story creation. Don't list passing checks. The fix list is the artefact.

---

## Quality Rules

- **Tier defaults to the template**, but PM can override.
- **Omit empty sections.** Don't print "-" or "N/A" placeholders.
- **ACs executable as tests.** Concrete data in Givens; observable outputs in Thens. Banned verbs ("works correctly", "handles gracefully", "as expected", "appropriately", "if needed", "properly") fail the scenario.
- **Error and edge scenarios mandatory** when the story crosses a system boundary or accepts user input.
- **Integration contracts referenced (linked), not described.** AI codegen needs the contract to call correctly.
- **Don't dictate implementation.** No "use Redux", no "store in IndexedDB". AC defines what; codegen decides how.
- **For playback stories**, name the DRM provider and license server pattern. If not selected, raise an Open block.
- **Granularity = clarity, not capacity.** Decompose when a story mixes decision boundaries, not when it feels big.
- **Self-validation is part of generation.** Don't make the PM run `/validate-stories` to find buildability issues - surface them in the Issues appendix here.
- **CMP/Didomi reminder (surface to PM, do not add to story):** After generating stories, check: does any story touch navigation, modals, overlays, player controls, or consent flows? If yes, add a callout at the end: "Reminder: these stories may interact with CMP/Didomi. Confirm CMP navigation regression is covered in QA scope before dev picks up."
- **Jira key format.** Use the project key from the loaded project context. Format: `[KEY]-DRAFT` - e.g., `CBC-DRAFT`. Never append a number; this is a draft placeholder, not a real issue ID.
- **Label generation rule.** Produce one platform label per platform in scope, plus one domain label. Platform labels: `ios` | `tvos` | `android` | `android-tv` | `fire-tv` | `web` | `samsung` | `lg`. Domain labels: `playback` | `auth` | `epg` | `search` | `browse` | `profile` | `monetization` | `onboarding` | `settings` | `analytics`. Format: lowercase, hyphen-separated, comma-delimited list. Example: `ios, tvos, playback`. If domain is ambiguous, use the most specific match; if none fit, use the story's primary feature noun lowercased.
- **Epic name.** Read from the loaded project context if an epic map is defined for the feature area. Write `TBD` only when no epic covers this area.
- **Design field rule.** Write `no design needed` only for non-visual stories (background processes, API integrations, analytics instrumentation). Any story with a user-facing UI requires a Figma link. If no link is available, raise `⚑ Open: Figma frame not supplied - defaulting to 'design required before implementation'.`
- Drafts only. Never auto-create via `/sub:jira-write`.

---

> **Skill verification:** Please ensure that the skill generate-stories.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
