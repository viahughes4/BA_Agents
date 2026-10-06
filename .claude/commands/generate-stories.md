# Skill: Generate User Stories

Generate Jira-ready stories that AI codegen can build from and human QA can verify against. Drafts only — never auto-create.

Self-validates against AI-Buildability checks. Tiers the output to story complexity: trivial stories get a 1-screen draft, heavy stories get the full template.

Universal output rules (concise language, omit empty sections, compact DECISION REQUIRED) apply — see CLAUDE.md → OUTPUT STYLE.

---

## Process

### 1. Load context
Invoke `/sub:context-loader`. If 🚫 or ⚠️, emit a single line recommending `/onboard-project`, then proceed — do not stop. Treat every missing critical field (Jira key, platform, API contracts, NFR defaults) as an `⚑ Open` block inline within the story.

### 2. Require prior discovery for non-trivial work
If the request touches multiple platforms, multiple systems, or playback / auth / monetization, and no Feature Discovery Report exists, stop and recommend `/feature-discovery` first.

### 3. Clarification gate
If platform scope, user need, design reference (UI), or integration contracts (system-boundary stories) are missing, ask **at most 3** questions. No `[TBD]` platform fields.

### 4. Playback gate
For any playback story, confirm Player SDK, DRM provider, license server pattern, CDN, manifest format. If unconfirmed, raise an Open block and pause.

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
1. [Title] — [tier] — [platforms]
```

Skip the index for an all-Minimal batch — go straight to the stories.

### 7. Generate each story per its tier
Use the per-tier template. Don't pad sections — omit them when empty.

### 8. Self-validate
Internally run AI-Buildability checks against each story (concrete spec, no embedded decisions, single deployable outcome, platform/SDK explicit, integration contracts referenced for boundary stories, ACs executable as tests, design resolved for UI stories).

- All clean → emit stories only.
- Any failure → emit stories + a short **Issues found** appendix listing only the failed checks per story.

Don't emit a per-story pass-table. Don't emit a verdict block. The story itself, plus the issue list when issues exist, is the artefact.

---

## Templates

### Minimal tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from project-context.md, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [single platform]
**Labels:** [platform label(s) + domain label — see label generation rule]

**Acceptance Criteria**

Scenario 1: [Happy path]
Given [precondition with concrete data]
When [single action]
Then [observable outcome]

[Add Scenario 2 only if the happy path doesn't cover the obvious failure or alternate.]

**Design:** [Figma frame link OR "no design needed" — see design rule]
```

Drop NFRs, Integration Contracts, Test Considerations, and Platform-Specific Considerations entirely for Minimal stories. Defaults inherited from `project-context.md` are sufficient.

### Standard tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from project-context.md, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform:** [list]
**Labels:** [platform label(s) + domain label — see label generation rule]
**Dependencies:** [PROJ-XXX] *(omit if none)*

**Integration Contracts** *(omit if no system boundary)*
- API endpoints: [linked spec]
- Events: [name + parameter schema]
- Auth assumptions: [token type, refresh behaviour]

**Acceptance Criteria**

Scenario 1: [Happy path]
Given [concrete data]
When [single action]
Then [observable outcome with concrete expected value]

[Add scenarios for materially distinct behaviours: error/failure if the story crosses a boundary or accepts user input; edge cases for obvious boundaries (empty / max / network loss); alternate paths for entitlement / role / locale variation. No vague verbs ("works correctly", "handles gracefully", "as expected"). Each scenario must be executable as a test.]

**NFRs** *(include only sub-bullets that diverge from defaults in `project-context.md` — omit the section if all defaults apply; include a sub-bullet if `project-context.md` has no value for it)*
- Performance: [Numeric SLA]
- Accessibility: [non-default specifics]
- Localization: [if multilingual]
- Error handling: [timeouts, retry, fallback]
- Analytics: [event name + parameters]

**Design:** [Figma frame link OR "no design needed" — see design rule]

**Open** *(omit if none)*
⚑ Open: [one-liner] — defaulting to [X] unless changed.
```

### Heavy tier

```
## [Story Title]
> [KEY]-DRAFT | Epic: [name from project-context.md, or TBD if no epic covers this area]

**As a** [user type], **I want** [goal] **so that** [reason].

**Platform Scope:** [list — include per-platform notes only when behaviour diverges]

**Labels:** [platform label(s) + domain label — see label generation rule]
**Dependencies:** [PROJ-XXX] *(omit if none)*

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

Scenario 2: [Error / failure — required for boundary stories]
...

Scenario 3: [Edge case — required when boundaries are obvious]
...

Scenario 4: [Alternate path — required when entitlement / role / locale changes behaviour]
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

**Open** *(omit if none — use Short form unless decision is material to scope, cost, cert, or architecture)*
⚑ DECISION REQUIRED
What: ...
Why it matters: ...
Options: ...
Recommended default: ...
```

---

## Issues Appendix *(emit only when self-validation finds failures)*

```
### Issues Found
- [Story Title]: [specific failure — e.g., "AC scenario 2 uses 'handles gracefully' (vague verb)"]
- [Story Title]: [specific failure — e.g., "Integration contract missing — story calls auth API but no endpoint linked"]
```

Don't list passing checks. The fix list is the artefact.

---

## Quality Rules

- **Tier defaults to the template**, but PM can override.
- **Omit empty sections.** Don't print "—" or "N/A" placeholders.
- **ACs executable as tests.** Concrete data in Givens; observable outputs in Thens. Banned verbs ("works correctly", "handles gracefully", "as expected", "appropriately", "if needed", "properly") fail the scenario.
- **Error and edge scenarios mandatory** when the story crosses a system boundary or accepts user input.
- **Integration contracts referenced (linked), not described.** AI codegen needs the contract to call correctly.
- **Don't dictate implementation.** No "use Redux", no "store in IndexedDB". AC defines what; codegen decides how.
- **For playback stories**, name the DRM provider and license server pattern. If not selected, raise an Open block.
- **Granularity = clarity, not capacity.** Decompose when a story mixes decision boundaries, not when it feels big.
- **Self-validation is part of generation.** Don't make the PM run `/validate-stories` to find buildability issues — surface them in the Issues appendix here.
- **Jira key format.** Use the project key from `project-context.md`. Format: `[KEY]-DRAFT` — e.g., `STREAM-DRAFT`. Never append a number; this is a draft placeholder, not a real issue ID.
- **Label generation rule.** Produce one platform label per platform in scope, plus one domain label. Platform labels: `ios` | `tvos` | `android` | `android-tv` | `fire-tv` | `web` | `samsung` | `lg`. Domain labels: `playback` | `auth` | `epg` | `search` | `browse` | `profile` | `monetization` | `onboarding` | `settings` | `analytics`. Format: lowercase, hyphen-separated, comma-delimited list. Example: `ios, tvos, playback`. If domain is ambiguous, use the most specific match; if none fit, use the story's primary feature noun lowercased.
- **Epic name.** Read from `project-context.md` if an epic map is defined for the feature area. Write `TBD` only when no epic covers this area.
- **Design field rule.** Write `no design needed` only for non-visual stories (background processes, API integrations, analytics instrumentation). Any story with a user-facing UI requires a Figma link. If no link is available, raise `⚑ Open: Figma frame not supplied — defaulting to 'design required before implementation'.`
- Drafts only. Never auto-create via `/sub:jira-write`.
