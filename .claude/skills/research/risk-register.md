---
name: Risk Register
description: This skill should be used when the user asks to "create a risk register", "identify risks in my roadmap", "analyze roadmap for risks", "generate risk register", "assess project risks", "build risk matrix", "add risks to register", or wants to "review risks" for a project or timeline. Supports roadmap risk analysis, risk scoring, mitigation planning, and risk register generation.
version: 2.0.0
---

# Risk Register Skill

## Purpose

Structured identification, scoring, and tracking of project risks. Supports creating new registers, adding risks from feature discovery, and reviewing/updating existing registers.

---

## Files

- **Template:** `scripts/risk-register-template.md`
- **Categories:** `references/risk-categories.md`
- **Methodology:** `references/methodology.md`

---

## Risk Scoring

Uses the same **Impact x Likelihood** scale as `/feature-discovery` - scores are directly portable between skills.

**Priority = Impact x Likelihood (1-3 each)**

| Level | Impact | Likelihood |
|-------|--------|------------|
| 1 | Minor - workaround exists, cosmetic, one person blocked | Unlikely (<30%) |
| 2 | Significant - feature delayed 1-2 weeks, quality concerns | Reasonable chance (30-60%) |
| 3 | Critical - project blocked, launch delayed, team crisis | Very likely (>60%) |

**Priority buckets:** 🔴 Critical 7-9 · 🟠 High 4-6 · 🟡 Medium 2-3 · 🟢 Low 1

**Risk entry format:**
> **Risk**: [title] | **Type**: Timeline / Technical / External / Resource | **I**: 1-3 | **L**: 1-3 | **Score**: I×L | **Owner**: [name] | **Mitigation**: [Prevention] / [Contingency] / [Trigger to escalate]

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load and validate project context for the active client.

If context-loader is unavailable or returns no data, surface a warning and ask the user to provide project name, team, and key milestones manually before continuing.

### Optional: Slack context
If the user has provided a Slack message, thread, or channel link, read `.claude/sub/slack-context-extractor.md` and follow its process in full. Merge urgency signals, blockers, and decisions into the risk identification context. If no Slack input is provided, skip this step.

### 2. Determine the task

| User request | Action |
|---|---|
| "Create a risk register for [project]" | Run full identification interview (Step 3), then generate register |
| "Add these risks to my register" | Skip interview; go directly to Step 5 |
| "Update my register - these items changed" | Load existing register, re-score affected risks, regenerate |
| "Risks from feature discovery" | Import directly from discovery report using existing I×L scores |

### 3. Risk identification interview (new registers only)

Gather:
- **Project / roadmap**: What is being built? Key milestones?
- **Team**: Size, skill gaps, availability constraints
- **Dependencies**: External blockers, integration points, third-party timelines
- **Assumptions**: What is assumed to work? What could go wrong?

Ask targeted questions. Build a risk picture before scoring.

### 4. Categorise risks

| Category | Examples |
|---|---|
| Schedule / Timeline | Milestone delays, certification windows, compressed sprints |
| Resource / Capacity | Team bandwidth, skill gaps, eng lead availability |
| External Dependency | Third-party backends, client API readiness, vendor delays |
| Technical | Architecture unknowns, legacy migration, platform quirks |
| Specification / Scope | Unclear requirements, missing designs, scope creep |
| Compliance / Cert | Platform certification, DRM, IAP rules |

Reference: `references/risk-categories.md`

### 5. Score and prioritise

Score each risk: I x L (1-3 each). Apply priority buckets above. Risks from `/feature-discovery` already carry I×L scores - import them directly.

### 6. Develop mitigations

For each risk:
- **Prevention**: What reduces likelihood?
- **Contingency**: Backup plan if it occurs
- **Trigger**: When to escalate

### 7. Determine register structure

**New project:** Create a master register with sections by feature/workstream.
- Each section covers one feature or delivery milestone
- Risks from `/feature-discovery` go into the matching feature section
- General project risks (team capacity, cert timeline) go into a "Project-Level" section

**Adding to existing register:**
- Identify the correct section (feature match or create new section)
- Assign the next available R-XXX ID (unique across all sections)
- Update the Summary Dashboard counts

### 8. Generate or update the register

Use `scripts/risk-register-template.md` as the base structure.

**Save location (client-based):**
| Client | Path |
|--------|------|
| client | `product-development/product/customers/accounts/[client]/risk-register/YYYY-MM-DD-[client]-risk-register.md` |
| VIDAA | `product-development/product/risk-register/YYYY-MM-DD-vidaa-risk-register.md` |
| JMMI | `product-development/product/risk-register/YYYY-MM-DD-jmmi-risk-register.md` |

File naming for new clients: `YYYY-MM-DD-[client]-risk-register.md`

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```
This generates a styled HTML version in an `html/` subfolder alongside the markdown.

If the file save or HTML export fails, report the error to the user with the full path attempted.

---

## Output Structure

Every register must contain:

1. **Header:** Last updated, project, team
2. **How to Use:** Scoring key, risk entry format
3. **Summary Dashboard:** Risk counts by section and severity
4. **Risk Sections:** One section per feature/workstream, risks in severity order
5. **Action Items by Week:** Combined across all sections, sorted by date
6. **Risk Owners & Escalation:** Who owns each risk and when to escalate
7. **Review Schedule:** Dates, reviewers, focus areas

---

## Importing Risks from Feature Discovery

When `/feature-discovery` produces risks in the Technical Flags & Risks section:

1. Each risk has an explicit I×L score - use it directly, do not re-score
2. Match the feature to its section in the risk register (or create a new section)
3. Assign the next R-XXX ID
4. Update the Summary Dashboard
5. Add action items for blocked/active risks to the appropriate week

---

## Quality Rules

- **Every risk must have an owner.** Unowned risks don't get actioned.
- **Use I×L integer scores (1-3).** No qualitative-only ratings ("High impact" without a number).
- **Scores must match the feature-discovery scale.** Never produce a register with a different scoring system - all PM skills share one scoring standard.
- **Blocked risks must have a clear escalation trigger.** Don't list a risk as BLOCKED without stating what resolves the block.
- **Keep it actionable.** Aim for 10-20 risks max. If a register exceeds 25 risks, split into separate feature sections.
- **Review dates are mandatory.** Every risk gets a `Next Review` date.
- **Never silently re-score.** If you change an I×L score from a feature discovery report, call it out explicitly.

---

## Key Concepts

### Risk vs. Issue
- **Risk**: Something that MIGHT happen; needs prevention/contingency
- **Issue**: Something that HAS happened; needs resolution
- This register tracks risks only - current issues belong in Jira

### Status Values
| Status | Meaning |
|--------|---------|
| 🔴 ACTIVE | Risk is live; mitigation in progress |
| 🔴 BLOCKED | Waiting on an external input to proceed |
| 🟡 MONITOR | Low probability; watching passively |
| 🟢 RESOLVED | Risk is closed; no longer applicable |

---

## Step 9. Quality check before closing

Before finishing, confirm the following are present in the output:
- [ ] All required output sections (Header, How to Use, Summary Dashboard, Risk Sections, Action Items, Risk Owners, Review Schedule)
- [ ] Every risk has an owner, an I×L score, and a Next Review date
- [ ] Summary Dashboard counts match the actual risk entries
- [ ] No em dashes in any output text
- [ ] HTML export was generated successfully
