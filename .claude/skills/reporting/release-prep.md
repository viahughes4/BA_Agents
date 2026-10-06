---
name: release-prep
description: Use this skill for pre-release document generation and cert readiness checks. For go/no-go day-of coordination, use release-day-runbook instead. Triggers include "prep the release", "release notes for v1.X", "release prep", "prep for cert", "what's in this release", "what shipped in v1.X", "release checklist", "prepare the release", "get ready for release", "pre-release review", "release readiness check", or any request to summarize what's going out in an upcoming version.
version: 1.0.0
---

# Skill: Release Prep

Generates release notes from completed Jira tickets, runs a cert readiness check, and flags any open blockers - all in one pass. Output is a clean release prep document ready to share internally or with CBC.

**Never post to Jira, Slack, or Confluence without explicit instruction.**

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths.

---

## Inputs

The user must provide:
1. **Version number** - e.g. `v1.20.0`
2. **Release date** (or "TBD")

Optional:
- **Slack context** - if the user pastes a Slack thread or channel link, read `.claude/sub/slack-context-extractor.md` and merge any relevant signals (blockers, go/no-go signals, last-minute changes) into the output.

---

## Process

### 1. Pull ticket data from Jira

Run `.claude/sub/jira-project-snapshot.md` to pull sprint and ticket data for the client board. Filter results to fix version = [version] and group by type:
- **Bug Fixes** - issues of type Bug
- **Features & Improvements** - Stories, Tasks, Improvements
- **Platform / Infrastructure** - anything tagged as platform, dependency, or infrastructure

### 2. Identify open blockers

From the snapshot output, filter for issues where status = In Progress, To Do, Open, or Blocked and fix version = [version] OR linked to the release.

Flag these as **Open Items - Review Before Release**.

### 3. Run cert readiness check

Load `.claude/skills/cert-checklist/SKILL.md` and apply the cert checklist against the completed ticket list and open items. Identify:
- Any cert-required items that are not yet resolved
- Any platform-specific risks (Samsung, LG, Xbox, X1, Xumo)

If the cert-checklist skill is unavailable or returns no results, note this in the Cert Readiness section as: Cert check could not be completed automatically - manual review required. Continue generating the rest of the document.

### 4. Generate release prep document

Save to: `product-development/product/status-reports/[client]/YYYY-MM-DD-[version]-release-prep.md`

Use this structure:

```markdown
# [Version] Release Prep - [Date]

## Release Summary
- **Version:** [version]
- **Target date:** [date]
- **Status:** [On Track / At Risk / Blocked]
- **Overall cert readiness:** [Ready / Needs Review / Blocked]

---

## What's Shipping

### Bug Fixes
| Ticket | Summary | Platform |
|--------|---------|----------|

### Features & Improvements
| Ticket | Summary | Owner |
|--------|---------|-------|

### Platform / Infrastructure
| Ticket | Summary | Owner |
|--------|---------|-------|

---

## Open Items - Review Before Release
| Ticket | Summary | Status | Risk |
|--------|---------|--------|------|

---

## Cert Readiness
| Check | Status | Notes |
|-------|--------|-------|

---

## Blockers
| Item | Reason | Owner | Resolution Path |
|------|--------|-------|----------------|
```

After saving the document, run `python3 scripts/md-to-html.py product-development/product/status-reports/[client]/YYYY-MM-DD-[version]-release-prep.md` and report the generated HTML path to the user.

### 5. Present output

Show the document in chat. Ask if the user wants to:
- Post to Slack (DM or channel)
- Push to Confluence
- Update the roadmap status for this version

Do not take any of these actions without explicit confirmation.

---

## Quality Rules

- **Pull from Jira first** - do not invent ticket content. If Jira data is unavailable, note it and proceed with what's available.
- **Flag cert blockers clearly** - if any cert-required item is unresolved, mark the overall cert readiness as "Needs Review" or "Blocked".
- **Platform column matters** - client ships to Samsung, LG, Xbox, X1, Xumo. Tag the platform for each bug fix so cert reviewers can prioritize.
- **Never fabricate ticket summaries** - if a ticket is sparse, use the ticket title only.

---

> **Skill verification:** Please ensure that the skill release-prep.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
