---
name: new-feature
description: Scaffold a new major feature folder with the full RDI-style structure - discovery docs, risk register, testing plan, weekly updates, roadmap, and client sync notes. Triggers on "new feature", "scaffold feature", "create feature folder", "set up [feature name]", "start discovery for", or any request to initialize a new major feature workstream.
---

# Skill: New Feature Scaffold

Create the complete folder structure and starter files for a new major feature. Modeled after the RDI folder structure - everything in one place from the start: discovery, risk register, testing, weekly updates, roadmap, and meeting notes.

---

## Process

### 1. Get the feature name

If the user has not provided a feature name, ask:

> **What's the name of the new feature?**
> This will be used as the folder name. Use a short slug (e.g. "commonwealth-games", "live-dvr", "in-app-messaging"). Avoid spaces.

Sanitize the name: lowercase, hyphens instead of spaces, no special characters.

### 2. Confirm the scope

Ask one quick question if not already clear:

> **Is this a client-facing feature (visible to CBC/Julien) or an internal workstream?**
> - Client-facing: creates client-facing roadmap entry, weekly update template
> - Internal: creates lighter structure, no external-facing docs

### 3. Create the folder structure

Base path: `product-development/product/customers/accounts/[client]/[feature-name]/`

Create these subfolders and starter files:

#### `discovery/`
File: `YYYY-MM-DD-[feature-name]-discovery.md`
```
# [Feature Name] - Discovery

**Date:** [today]
**Owner:** Vanessa Hughes
**Status:** In Discovery

---

## Problem Statement

[What problem does this feature solve? Why is client asking for it?]

---

## Scope

**In scope:**
- TBD

**Out of scope:**
- TBD

**Open questions:**
- TBD

---

## Key Contacts

| Name | Role | Company |
|------|------|---------|
| | | |

---

## Reference Documents

- [Add links to any SOW, Figma, Confluence pages, or client docs]

---

## Notes

[Discovery notes as work progresses]
```

#### `risk-register/`
File: `[feature-name]-risk-register.md`
```
# [Feature Name] Risk Register

**Feature:** [Feature Name]
**Owner:** Vanessa Hughes
**Created:** [today]
**Last Updated:** [today]

---

## Risk Dashboard

| ID | Risk | Severity | Status | Owner |
|----|------|----------|--------|-------|
| R-001 | [First risk - TBD] | TBD | OPEN | Vanessa Hughes |

---

## Active Risks

### R-001: [Risk Title]

| Field | Value |
|-------|-------|
| **Category** | [Technical / External Dependency / Scope / Resource] |
| **Status** | OPEN |
| **Description** | TBD |
| **Impact** | TBD |
| **Owner** | Vanessa Hughes |
| **client Action Needed** | None |
| **Opened** | [today] |
| **Next Review** | [today + 1 week] |

---

## Resolved Risks

| ID | Risk | Resolved | Resolution |
|----|------|----------|------------|
```

#### `testing/`
File: `[feature-name]-testing-plan.md`
```
# [Feature Name] - Testing Plan

**Owner:** Alejandro Santarrosa (QA)
**Prepared by:** Vanessa Hughes
**Date:** [today]
**Environment:** TBD

---

## Before You Start - Prerequisites

- [ ] Test environment confirmed stable
- [ ] Test accounts available
- [ ] Build with feature available

---

## Test Sections

### Section 1 - [Area TBD]

| # | Test Case | Expected Result | Pass / Fail | Notes |
|---|-----------|----------------|-------------|-------|
| 1.1 | TBD | TBD | | |

---

## Known Limitations

| Item | Reason | Owner |
|------|--------|-------|

---

## Sign-Off

| Section | Status | Issues Raised | Verified By |
|---------|--------|---------------|-------------|
```

#### `weekly-updates/`
File: `README.md`
```
# [Feature Name] - Weekly Updates

Weekly status updates for [Feature Name]. One file per week, created each Friday.

## Format
Files: `YYYY-MM-DD-[feature-name]-weekly-update.md`
```

#### `roadmap/`
File: `[feature-name]-roadmap.md`
```
# [Feature Name] - Roadmap

**Created:** [today]
**Owner:** Vanessa Hughes

---

## Timeline

| Phase | Target | Status | Notes |
|-------|--------|--------|-------|
| Discovery | TBD | In Progress | |
| Development | TBD | Not Started | |
| QA | TBD | Not Started | |
| Certification | TBD | Not Started | |
| Delivery | TBD | Not Started | |

---

## Key Milestones

| Date | Milestone |
|------|-----------|
| TBD | Discovery complete |
| TBD | Development complete |
| TBD | QA start |
| TBD | Certification submission |
```

#### `CBC-Sync/`
File: `README.md`
```
# [Feature Name] - client Sync Notes

Meeting notes and prep docs for the client syncs related to [Feature Name].

## Naming Convention
`YYYY-MM-DD-[meeting-type]-prep.md` or `YYYY-MM-DD-[meeting-type]-notes.md`
```

### 4. Generate HTML for starter files

Run the HTML converter on the discovery doc and risk register:
```bash
python3 scripts/md-to-html.py [discovery-file-path]
python3 scripts/md-to-html.py [risk-register-file-path]
```

### 5. Report to user

Confirm everything created:

```
## [Feature Name] scaffolded

**Base path:** `product-development/product/customers/accounts/[client]/[feature-name]/`

**Created:**
- discovery/ - [filename] + HTML
- risk-register/ - [filename] + HTML
- testing/ - [filename]
- weekly-updates/ - README.md
- roadmap/ - [filename]
- CBC-Sync/ - README.md

**Next steps:**
1. Fill in the Problem Statement in the discovery doc
2. Add your key contacts
3. Add your first risk to the risk register
4. Add the feature to the roadmap in `roadmap/roadmap.md`
5. Say "update tracker" to add this feature to this week's active items
```

---

## Rules

- Always use today's date in file names
- Never overwrite existing files - if the folder already exists, report it and ask before proceeding
- Sanitize the feature name: lowercase, hyphens, no spaces or special characters
- Always generate HTML for discovery and risk register
- The tracker is NOT automatically updated - remind Vanessa to do it

---

> **Skill verification:** Confirm that new-feature SKILL.md was run. If not, run from the top.
