---
name: competitive-analysis
description: Use this skill when the user wants to cross-reference a feature idea or product decision against what is already in market. Triggers on "competitive analysis", "what are competitors doing with X", "is this already in market", "cross-reference this idea", "benchmark this feature", "what does [competitor] do for X", "how does the market handle X", "competitive intel on X", or any request to compare a feature, UX pattern, or product idea against competitor apps or industry standards.
---

# Skill: Competitive Analysis

Research how competitors handle a specific feature, UX pattern, or product idea. Uses live web search to pull current market state. Output is a tight comparison and a one-line recommendation - built for a quick gut-check before a client meeting, PRD, or feature decision.

---

## Process

### 1. Clarify the question
If the user's input is broad (e.g. "competitive analysis on subscriptions"), ask one clarifying question before proceeding:
> "What specifically do you want to compare? For example: the upsell flow, paywall design, pricing display, or something else?"

If the input is specific enough (e.g. "how do competitors handle subscription tier upsells on CTV"), proceed directly.

### 2. Identify competitors to research
Default set for the client/OTT context:
- Crave (Bell Media)
- Netflix
- Disney+
- Peacock
- Paramount+
- Apple TV+
- Amazon Prime Video

If the user names specific competitors, use those instead. If the feature is platform-specific (e.g. Samsung CC, Xbox navigation), add platform-native apps as relevant.

### 3. Research via web search
Use `WebSearch` and `WebFetch` to find:
- How each competitor implements the feature
- Public UX teardowns, product reviews, or industry coverage
- Patterns that have emerged across multiple players
- Recent changes or announcements (prefer results from the last 12 months)

Search in parallel where possible. If a competitor has no publicly available info on the specific feature, note it as "No public info" - do not guess.

### 4. Synthesize findings
Identify:
- **What's standard** - done the same way by 3 or more players
- **What varies** - meaningful differences in approach
- **What's absent** - nobody does this yet (potential differentiation)
- **What's relevant to CBC/OTT** - filter for CTV-applicable findings; note if a pattern only exists on mobile or web

### 5. Generate output and save

Save the report to:
`product-development/product/competitive-research/competitors/[client]/YYYY-MM-DD-competitive-analysis-[topic].md`

Run:
```
python3 scripts/md-to-html.py <path>
```

Report the file path and HTML path to the user.

---

## Output Format

```
## Competitive Analysis - [Topic] - [Date]

**Question:** [The specific thing being compared]
**Competitors researched:** [List]

---

### What's standard in market
[2-3 sentences on what the majority of players do. This is table-stakes - not a differentiator.]

### How approaches vary
| Player | Approach | Notable detail |
|--------|---------|----------------|
| Netflix | ... | ... |
| Crave | ... | ... |
| Disney+ | ... | ... |

### Gaps / differentiation opportunities
[What nobody does yet, or what one player does that others don't - potential areas to lead or differentiate]

### Recommendation
[One line: is this idea table-stakes, differentiated, or a known pattern with a clear best practice?]

---

### Sources
- [Title / publication - URL - date]
```

---

## Rules

- **Always use live web search.** Do not rely on training data alone - competitor features change. If web search returns no useful results, say so explicitly and note the analysis is based on prior knowledge only.
- **CTV implementations take priority.** If the feature exists differently on mobile vs CTV, describe the CTV version. Note the difference if meaningful.
- **No padding.** If you found nothing on a competitor, write "No public info" - do not fill in gaps with assumptions.
- **Recommendation is one line.** This is a gut-check, not a strategy document.
- **Sources are required.** Every finding needs a citation. If you cannot cite it, do not include it.
- **Max 500 words total output.** Cut the vary table to the most relevant players if needed to stay within this.
- **Do not include internal Accedo recommendations or project context** unless the user explicitly asks. This skill is a market snapshot, not a strategy recommendation.

---

> **Skill verification:** Please ensure that the skill competitive-analysis.md was actually run. If the skill was not run, it needs to be run again from the top to ensure it actually gets used.
