---
name: ask
description: Ask a question and get a cited answer from your own repo. Triggers on "ask my repo", "what do we know about", "what was decided on", "history on", "what's the status of", "when did we", "who owns", "what happened with", or any question that should be answered from project documentation rather than general knowledge.
---

# Skill: Ask My Repo

Answer any question about the client's Jira project using only real data from the repo - standup notes, risk register, tracker, meeting notes, RDI docs, roadmap, and PRDs. Every answer is cited with the source file and relevant context so you can verify it.

---

## Process

### 1. Parse the question

Identify the key subject(s) from the question. Common subjects and where to look:

| Subject | Where to look |
|---------|--------------|
| RDI (features, risks, decisions) | `RDI/risk-register/`, `RDI/weekly-updates/`, `RDI/discovery/`, standup notes |
| Ticket / Jira issue | Weekly tracker, standup notes, risk register |
| Person (Andy, Dario, Eduardo, Alex, Remi, Loic, etc.) | Tracker, standup notes, meeting notes |
| Risk or blocker | `risk-register/CBC-Risk-Register-June-2026.md`, `RDI/risk-register/RDI-Risk-Register-2026.md` |
| Decision made | Standup notes, meeting notes, tracker "Decisions" sections |
| Roadmap item | `roadmap/roadmap.md`, `roadmap/roadmap-client-facing.md` |
| Release / build | Standup notes, tracker, `sprints/` |
| Feature (long press, CC, analytics, etc.) | Standup notes, tracker, meeting notes, PRDs |
| Action item / follow-up | Weekly tracker `Follow-Up Schedule`, meeting notes `Next Steps` |

### 2. Read relevant files

Based on the subject, read the most relevant files. Do NOT read everything - be targeted.

**For time-bounded questions** ("when did", "last week", "recently"): start with the most recent files and work backwards.

**For decision questions** ("what was decided", "was it confirmed"): prioritize meeting notes Decisions sections and tracker.

**For status questions** ("what's the status of", "is X done"): start with the weekly tracker, then Jira if needed.

**For risk/blocker questions**: go directly to the risk registers.

**For person questions** ("what is Andy working on", "what did Remi say"): standup notes + tracker.

Relevant file paths:
- Weekly trackers: `product-development/product/customers/accounts/[client]/weekly-trackers/`
- Standup notes: `product-development/product/meetings/[Client]/meeting-notes/standup/`
- Meeting notes: `product-development/product/meetings/[Client]/meeting-notes/`
- Risk register: `product-development/product/customers/accounts/[client]/risk-register/[Client]-Risk-Register.md`
- RDI risk register: `product-development/product/customers/accounts/[client]/RDI/risk-register/RDI-Risk-Register.md`
- RDI weekly updates: `product-development/product/customers/accounts/[client]/RDI/weekly-updates/`
- Roadmap: `product-development/product/customers/accounts/[client]/roadmap/roadmap.md`
- PRDs: `product-development/product/PRDs/[client]/`

### 3. Synthesize and cite

Build the answer from what you found. For each claim:
- State the fact clearly
- Cite the source: `[file name, relevant section or date]`
- If multiple sources say different things, surface the conflict and note which is more recent

**If nothing found:** say so explicitly - "I couldn't find any reference to [topic] in the repo. This may not be documented yet."

**Never fabricate or infer** beyond what the documents actually say. If something is implied but not stated, flag it as an inference.

### 4. Output format

```
## Answer: [restate the question briefly]

[Answer in plain language, 2-5 sentences]

**Sources:**
- [file: section/date] - "[relevant quote or summary]"
- [file: section/date] - "[relevant quote or summary]"

**Note:** [any caveats, conflicts, or "this may be outdated since X date"]
```

---

## Quality rules

- **Cite everything.** If you can't point to a source, don't assert it.
- **Use the most recent source.** If standup notes from Jun 15 say something different from a risk register entry from Jun 8, surface the conflict and note the standup is more recent.
- **Be honest about gaps.** "Not documented" is a valid answer.
- **One clear answer, not a dump.** Don't paste entire file sections - synthesize and quote the key line.
- **Flag staleness.** If the most recent source is >2 weeks old, note it may be outdated.

---

> **Skill verification:** Confirm that the ask SKILL.md was run. If not, run from the top.
