---
name: validate-stories
description: Use this skill to review, check, or validate user stories or tickets before sending to engineering or QA. Triggers on "check these tickets before Andy picks them up", "review this story for me", "is this ticket good enough to send to dev", "review CBC-XXXX", "check my stories", "validate these", "are these tickets ready", "review before I assign", or any request to assess ticket or story quality. Also replaces validate-ac - use this for AC-only checks too. Paste ticket text, provide Jira keys, or describe what to check.
---

# Skill: Validate Stories

Review user stories and tickets for quality before they go to engineering or QA. Works on pasted story text, Jira ticket keys, or tickets from this session.

Two modes:
- **Fast** (default for single tickets) - AC quality + buildability check only. Quick answer: ready or not, and what to fix.
- **Full** (default for batches of 3+) - all 5 lenses including platform completeness, cert risk, and NFR quality. Use before a release or when handing a full batch to dev.

Universal output rules apply - see CLAUDE.md for formatting guidelines.

---

## Process

### 1. Determine mode
Default to Full. If `$ARGUMENTS` includes `fast`, run Lenses 1-2 only.

### 2. Accept input in any of three forms
- Pasted story text (single or multiple)
- Jira issue ID(s) - invoke `.claude/sub/jira-project-snapshot.md` or use `mcp__plugin_atlassian_atlassian__getJiraIssue` for individual ticket lookups
- Output of `/generate-stories`

### 3. Load context
Invoke `.claude/sub/context-loader.md`. Use it to verify platform scope claims, locale requirements, system selections.

### 4. For Jira IDs, fetch
Use `mcp__plugin_atlassian_atlassian__getJiraIssue` to retrieve full story content (description, ACs, labels, links, parent epic). If the Jira fetch fails or the issue is not found, skip that ticket, note the error inline (e.g., CBC-XXXX: could not retrieve - check key or permissions), and continue with remaining tickets.

### 5. For batches (>= 3 stories), emit summary table first
Per-story full reports only for ⚠️ and 🚫 verdicts. ✅ Buildable stories appear as a single one-line summary at the bottom (e.g., "✅ Buildable: <jira.project_key>-101, <jira.project_key>-104, <jira.project_key>-107"). Do not emit a per-story report for a passing story.

### 6. Run lenses
Fast mode: Lens 1 + Lens 2 only.
Full mode: all 5 lenses.

Be uncompromising on Lens 1 and Lens 2 - these are the gate. Lens 3-5 surface risk, not block.

---

## The 5 Validation Lenses

### Lens 1: AI-Buildability *(primary gate)*
- **Spec concrete:** no vague verbs ("works correctly", "handles gracefully", "as expected"), no `[TBD]` fields
- **No embedded decisions:** no unresolved DECISION REQUIRED in the story body; if present, the story is not buildable until answered
- **Single deployable outcome:** the story produces one shippable result on one platform with one decision boundary (split if not)
- **Platform & SDK explicit:** specific platform list, named SDK versions where the story interacts with the platform layer
- **Integration contracts referenced:** API endpoints / event schemas / token assumptions linked to actual specs, not described in prose, when the story crosses a system boundary
- **Design resolved:** Figma frame referenced for any UI story, or explicit "no design needed"
- **ACs executable as tests:** concrete inputs, observable outputs, every Then assertion can become an automated or manual test

### Lens 2: AC Quality
- **Format:** Accepts either hybrid format (plain summary sentence + Given/When/Then scenarios) or pure Gherkin. Plain bullet lists without Given/When/Then fail this lens.
- **Plain summary:** if present, should state the overall expected behaviour in one sentence - readable at a glance
- **Happy path:** present and well-formed with concrete data
- **Observable Thens:** every Then states what the user/system observes - not "the system handles X"
- **Concrete data:** Givens have actual values, not "some user" or "valid input"
- **No implementation prescription:** no "use Redux", no "call endpoint /v1/..." (the *contract* belongs in Integration Contracts, not in the AC verb)
- **Required scenario coverage:**
  - Happy path: always
  - Error / failure: required when the story crosses a system boundary or accepts user input
  - Edge cases: required when the story has obvious boundaries (empty, max, network loss, concurrent action)
  - Alternate paths: required when entitlement, role, or locale changes the behaviour - especially user states (Guest, Member, Unsubscribed, Not Eligible)
- **Loading / empty / error states** addressed where they meaningfully diverge from defaults

### Lens 3: Platform & Technical Completeness
- Platform scope explicit (no "All Platforms" without listing)
- D-pad navigation addressed for any TV platform (Fire TV, Samsung, LG, Apple TV, Roku)
- Player SDK and DRM named on any playback story
- Token refresh and session expiry covered in any auth story
- Concurrent stream limits called out for sign-in / playback start stories
- IAP vs web checkout distinction explicit for any purchase story
- Named analytics events (with full parameter set) on any analytics story
- GDPR / CCPA / ATT / COPPA flags raised on any data, tracking, or kids-content story
- CSAI vs SSAI explicit on any ad story

### Lens 4: Certification Risk
Spot-check for cert blockers using the platform-specific rules in `/cert-checklist` (do not duplicate the full checklist here - defer to it for thorough scope-level review). Flag anything that would obviously fail cert: ATT prompt missing on iOS tracking, Sign in with Apple missing where required, D-pad gaps on TV stories, Widevine L1 missing for Android HD/4K DRM.

If the batch covers a meaningful slice of release scope, recommend running `/cert-checklist` for the full readiness pass.

### Lens 5: NFR Quality
- **Present** (not omitted)
- **Specific** - numeric SLAs where applicable
- **Measurable** - testable thresholds
- **Realistic** for the platform (don't ask for 1s startup on a Tizen 2018 TV)
- **Accessibility** called out per platform
- **Error handling concrete** - timeouts, retries, fallback
- **Localization** explicit for multilingual UI stories

---

## Output (per story, when ⚠️ or 🚫)

```
## Validation Report - [Story Title] - [ID] - [Date]

**Verdict:** ✅ Buildable / ⚠️ Hold (revise) / 🚫 Block (major rework)
**Summary:** [One sentence]

### Lens 1: AI-Buildability
| Check | Result | Note |
|---|---|---|
| Spec concrete (no vague verbs/TBD) | ✅/❌ | ... |
| No embedded decisions | ✅/❌ | ... |
| Single deployable outcome | ✅/❌ | ... |
| Platform & SDK explicit | ✅/❌ | ... |
| Integration contracts referenced | ✅/➖/❌ | ... |
| Design resolved | ✅/➖/❌ | ... |
| ACs executable as tests | ✅/❌ | ... |

### Lens 2: AC Quality
- GWT format: ✅/❌
- Happy path well-formed with concrete data: ✅/❌
- Observable Thens: ✅/❌
- No implementation prescription: ✅/❌
- Required scenario coverage (error / edge / alternate): ✅/⚠️ - [if ⚠️, name the specific missing scenario, e.g., "no DRM license-failure scenario on a playback story"]
- Loading / empty / error states (where applicable): ✅/➖/❌
[Specific issues called out by scenario number]

### Lens 3: Platform & Technical Completeness
[Per applicable item: ✅/❌ + note]

### Lens 4: Certification Risk
[Cert-blocker flags only; defer to /cert-checklist for full review]

### Lens 5: NFR Quality
[Present? Specific? Measurable? Realistic?]

### Required Changes
1. [Specific, actionable]

### Suggested Improvements
- [Nice-to-have]

### DECISION REQUIRED
⚑ ...
```

---

## Batch Summary (when >= 3 stories)

```
## Validation Summary - [N] stories - [Date]

| ID | Title | Verdict | Critical Issues |
|---|---|---|---|
| <jira.project_key>-101 | ... | ✅ Buildable | - |
| <jira.project_key>-102 | ... | ⚠️ Hold | Missing error-state AC; no DRM named |
| <jira.project_key>-103 | ... | 🚫 Block | No platform scope; no ACs |

### Build-Ready Recommendation
- **Buildable now:** [count] - [comma-separated IDs]
- **Hold for revision:** [count] - [ID: one-line fix per item]
- **Block:** [count] - [ID: one-line reason per item]

*Recommended next step (one line - omit if obvious).*
```

---

## Quality Rules

### AC requirements vary by ticket type - apply judgment

Not all tickets require Given/When/Then scenarios. Apply the right standard for each type:

| Ticket type | AC requirement |
|------------|---------------|
| User story (feature/UI) | Required - hybrid format (plain summary + Gherkin scenarios per user state/path) |
| Task (implementation/refactor) | Not required - clear description of deliverable and integration contracts is sufficient |
| Bug | Not required - steps to reproduce + expected vs actual behaviour is sufficient. Flag only if expected behaviour is absent or ambiguous. |
| QA verification story | Not required - test scope, pass conditions, and what to verify is sufficient |
| Sub-task | Not required - scope defined by parent ticket |

**Never flag "no AC" on a bug, task, or sub-task.** Only flag missing AC on user stories and QA verification stories where the pass condition is genuinely unclear.

### When flagging issues, be specific

Every flag must state:
- What exactly is missing (name the specific scenario, state, or field)
- What the fix looks like (give a concrete example or rewrite)
- Why it matters (what would fail without it)

Never write "missing AC" or "no scenarios" as a standalone flag. Always say which scenario is missing and write an example of what it should look like.

**Bad:** "Missing error state AC"
**Good:** "Missing scenario for when user lacks rdi:premium scope - add: Given user does not have rdi:premium scope, When they tap the ICI RDI card, Then the subscription modal appears with the 'M'abonner a ICI RDI' CTA"

### Other rules

- **🚫 Block** = a Lens 1 fail on Spec concrete, No embedded decisions, or ACs executable. Do not soften. A story with `[TBD]` platforms, vague Thens, or unresolved DECISION REQUIRED is 🚫.
- **⚠️ Hold** = fixable issues; required changes must be specific and actionable with example rewrites.
- **✅ Buildable** = spec is clear enough for dev to implement and QA to verify. Does not require perfection.
- **For playback stories where DRM is material**, missing player SDK or DRM provider = automatic 🚫.
- **Never approve "All Platforms"** without an explicit list.
- **Apply judgment on scenario coverage.** A simple UI affordance with only a happy path can pass. A payment / playback / auth flow with only a happy path cannot. When you flag missing coverage, name the specific scenario and write the example.
- **Cert detail belongs in `/cert-checklist`.** Lens 4 is a spot-check, not a full review.
- **Skip per-story reports for passing tickets.** A passing ticket is one row in the summary table.

---

> **Skill verification:** Please ensure that the skill validate-stories.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
