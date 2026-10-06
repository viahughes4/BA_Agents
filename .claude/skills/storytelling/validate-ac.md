---
name: validate-ac
description: Redirects to validate-stories. Use validate-stories for all AC and story validation - it handles both fast AC-only checks and full story reviews. This skill is kept for backwards compatibility only.
---

# Redirected to validate-stories

This skill has been merged into `validate-stories` which handles both AC-only checks (fast mode) and full story validation (full mode).

Run `/validate-stories` instead. It accepts the same inputs: pasted AC, Jira ticket keys, or story text - and returns a faster, cleaner result.

# Skill: Validate AC

Single-story AC quality check. Same rule set as `/validate-stories fast` - use this skill when you only have ACs (not a full story) or want a quick single-story pass.

For batch validation, use `/validate-stories fast` directly.

Universal output rules apply: see CLAUDE.md.

---

## Process

1. Accept: pasted ACs, full story, or Jira ID. For Jira IDs, invoke `.claude/sub/jira-project-snapshot.md` with `ISSUE <id>`.
2. Determine **risk surface**:
   - **Boundary**: crosses a system boundary (auth, playback, payment, analytics, sync, network call) or accepts user input.
   - **Local UI**: purely client-local UI affordance with no system call and no user input beyond a tap.
3. Apply the AC Quality rules (same as `/validate-stories` Lens 2):
   - Given/When/Then format, not bullet lists
   - Concrete data in Givens (no "some user", "valid input")
   - Single observable action in When
   - Observable Thens with concrete expected values
   - **Banned vague verbs** in Thens: "works correctly", "handles gracefully", "as expected", "appropriately", "if needed", "properly". Each occurrence is an automatic scenario FAIL.
   - No implementation prescription (no "use Redux", no "call /v1/...")
   - Required scenario coverage:
     - Boundary stories: happy + error + edge (when boundaries are obvious) + alternate (when entitlement / role / locale changes behaviour)
     - Local UI stories: happy may suffice
   - Loading / empty / error states for screen-touching stories where they diverge from defaults
4. For each failing scenario: name the problem, provide a corrected rewrite.

No preamble. Be fast.

---

## Output

```
## AC Quality: [Story Title or ID]
**Result:** PASS / PARTIAL / FAIL
**Risk surface:** Boundary / Local UI

### Issues *(omit if none)*
**Scenario [N]: [title]**
- Problem: [specific]
- Rewrite:
  Given [...]
  When [...]
  Then [...]

### Missing Scenarios *(omit if none)*
- [e.g., "Boundary story (playback): no error scenario for license-server failure"]
- Suggested:
  Given [...]
  When [...]
  Then [...]
```

---

## Quality Rules

- **PASS** = all scenarios well-formed, no banned vague verbs, required coverage matches risk surface.
- **PARTIAL** = format clean, but missing a required scenario type for the risk surface.
- **FAIL** = format issues, banned vague verbs, no GWT, or implementation prescription in the AC.
- **A single banned vague verb is a scenario-level FAIL.** Don't soften.
- Always provide a corrected rewrite - never just "this is wrong."
- Boundary stories with only a happy path cannot pass. Local UI stories with only a happy path can.
- For multi-lens validation (INVEST / cert / NFRs), use `/validate-stories full` instead.

---

> **Skill verification:** Please ensure that the skill validate-ac.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
