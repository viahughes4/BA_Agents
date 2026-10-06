---
name: confluence-lookup
description: Use this skill when the user asks any question that requires looking up information in Confluence. Triggers include "check Confluence", "look up the spec", "what does Confluence say about", "find the release notes for", "when was X released", "what's in the build plan", "check the QA report", "what are the test accounts", "find the PRD for", "look up the certification process", or any question about release history, build schedules, QA sign-offs, feature specs, test cases, or internal project docs for any client.
version: 2.0.0
---

# Skill: Confluence Lookup

General-purpose Confluence navigation skill. Supports multiple client spaces. Go directly to the correct page using the known space tree - do not do a broad CQL search unless the specific page ID is unknown.

---

## MCP Tools

| Tool | When to use |
|------|-------------|
| `mcp__plugin_atlassian_atlassian__getConfluencePage` | Fetch a specific page by ID (preferred - use when page ID is known) |
| `mcp__plugin_atlassian_atlassian__searchConfluenceUsingCql` | Search by keyword or version number when page ID is not known |
| `mcp__plugin_atlassian_atlassian__getConfluencePageDescendants` | Explore children of a section when the exact child page is unknown |

**Always use `contentFormat: "markdown"`** for readable output.

---

## Process

### 1. Identify the client
The user will typically specify the client (e.g. "CBC", "VIDAA"). If unclear, ask.

### 2. Load the client config below
Each client has its own section with: Cloud ID, space key, root page ID, space tree, and routing rules.

### 3. Route using the client's routing table
Go directly to the known page ID. Only fall back to CQL search when:
- The version/page is not in the known list
- The section has many children and the exact one is unknown

### 4. Fetch and answer
Extract only the relevant information. Always surface the Confluence URL so the user can open the page directly.

### 5. If page is a parent index
Use `getConfluencePageDescendants` to list children, then fetch the matching child.

### 6. Error handling
If a page fetch returns an error or empty result, fall back to CQL search using the known space key and keywords. If CQL also fails, note the failure to the user and provide the Confluence space URL so they can search manually.

---

## Rules

- **Go direct.** Use page IDs when known. CQL search is a fallback, not the default.
- **Never fetch the root page to navigate** - use the space tree below instead.
- **Summarize, don't dump.** Extract the specific answer; cite page title and URL.
- **One fetch is the goal.** If you know the page ID, one tool call should be enough.

---

---

# Client: CBC

**Cloud ID:** `accedobroadband.jira.com`
**Space key:** `CBC`
**Root page ID:** `2490630473`
**Root URL:** `https://accedobroadband.jira.com/wiki/spaces/CBC/overview`

## client Space Tree

```
client / Radio-Canada Home (2490630473)
│
├── Internal Project Information (2490568903)
│   ├── Onboarding (2489780146)
│   ├── Project Management Documentation (5235572757)
│   │   ├── client Standing Team Build Release Plan (4979359942)  ← release schedule
│   │   ├── [Internal] client QA Reports (4971593852)             ← internal QA sign-offs
│   │   ├── Internal Governance Call Meeting Notes (2491383837)
│   │   ├── Key Documents (2489976223)
│   │   ├── Retros (3753017393)
│   │   ├── Lessons Learned (5438242867)
│   │   └── Paralympics / Olympics (5443289134)
│   ├── Technical Solution (2490568934)
│   ├── client Feature Details (5236031497)                       ← feature specs
│   │   ├── Pause Ads (5391810708)
│   │   ├── Admin Panel Improvements (5400526853)
│   │   ├── MyLineup (5431394392)
│   │   ├── Promo Banner (5432639603)
│   │   └── Comscore Analytics (5431885861)
│   ├── QA (Internal) (2490570038)
│   ├── Product Design (2505212660)
│   ├── Support (2490570114)
│   │   ├── Project Quick Guide
│   │   ├── Process of Monthly Report
│   │   ├── Outstanding Customer Support Ticket
│   │   ├── Support Package
│   │   └── Support Release
│   └── Archive - Meeting Notes 2022-2023 (2490630999)
│
├── External Project Information (2490567784)
│   ├── Project Management (2490567798)
│   │   ├── Project Resources and Project Charter (2491285578)
│   │   ├── Project Plan (2490567840)
│   │   ├── Meeting Notes (2490567848)
│   │   │   ├── Kick-off Meeting (2490567856)
│   │   │   ├── Daily Meeting Minutes (2500427777)
│   │   │   ├── Demo Meeting Minutes (2571599875)
│   │   │   └── Backlog Grooming (2546630657)
│   │   ├── Project Scope (2490567872)
│   │   │   ├── Supported Devices (2490567886)
│   │   │   └── PRD (2499838374)
│   │   ├── Weekly Status Reports (2490568028)
│   │   ├── Vacation Calendar (2489715157)
│   │   └── CBC/Radio-Canada Client Feedback (2505768967)
│   ├── Technical Documentation (2489648506)
│   ├── QA (2490568102)                                        ← external QA docs
│   │   ├── QA Strategy (2504425498)
│   │   ├── Test Cases (2490568287)
│   │   ├── Test Accounts (2490568226)                        ← login credentials
│   │   └── QA Reports (2490568156)
│   │       ├── QA REPORTS (OLYMPICS) (3432480959)
│   │       ├── QA Reports - Phase 1 (2913042511)
│   │       ├── QA Project reports - Phase 2 PDF (2912518201)
│   │       ├── QA Reports - 2024 (3377070210)
│   │       └── 2025 (4798054813)
│   ├── Technical Build (2490568331)
│   │   ├── How-to Articles (2490568372)
│   │   │   ├── How to install a Tizen build through USB (2490568380)
│   │   │   ├── How to properly test a TV app on a webpage (2575204444)
│   │   │   └── How to setup closed captions styles on Gem (2666561840)
│   │   └── Error Handling (2490568770)
│   ├── Platform Certification Processes (2511603368)
│   └── CTV Release/Build Notes (3547758612)                  ← ALL release notes
│       ├── client CTV Release Notes (3574628356)                ← human-readable, by version
│       │   ├── [Samsung][LG][X1][Rogers][Xbox] Release Notes - 1.19.2  (5678071811)
│       │   ├── [Samsung][LG][X1][Rogers][Xbox] Release Notes - 1.19.1/1.20.0 - May (5632983041)
│       │   ├── [Samsung][LG][X1][Rogers][Xbox] Release Notes - 1.19.0  (5479006209)  ← NOT released to PROD
│       │   ├── [LG][X1][Rogers][Xbox] Release Notes - 1.18.2            (5639372801)
│       │   ├── [Samsung][LG][X1][Rogers][Xbox] Release Notes - 1.18.1   (5478219777)
│       │   ├── [Samsung][LG][X1][Rogers][Xbox] Release Notes - 1.18.0 Paralympic Games (5341544449)
│       │   └── ... (older versions back to 1.10.x - use CQL search)
│       └── client CTV Build Notes (3574857793)                  ← raw RC build notes
│           └── Changelog (3547594816)
│
└── Build Archive (2607939776)
    ├── QA Releases (2607317949)
    ├── UAT Builds (2973729507)
    └── PROD Builds (2607808801)
```

## client Routing Rules

| Question type | Page ID | Notes |
|---|---|---|
| Release schedule / build dates | `4979359942` | Build Release Plan |
| Internal QA sign-off by version | `4971593852` | [Internal] client QA Reports |
| External QA reports | `2490568156` | Then check children by year |
| Feature spec (Pause Ads, MyLineup, etc.) | `5236031497` | client Feature Details - then fetch specific child |
| Test accounts / login credentials | `2490568226` | Test Accounts |
| Test cases | `2490568287` | Test Cases |
| Certification process | `2511603368` | Platform Certification Processes |
| How-to / install guides | `2490568372` | How-to Articles |
| PRD / project scope | `2499838374` | PRD |
| Status reports | `2490568028` | Weekly Status Reports |
| Team contacts / roster | `2490630473` | Root home |
| Vacation calendar | `2489715157` | - |
| Client feedback | `2505768967` | - |
| Release notes by version | CQL search | `space = "CBC" AND title ~ "X.XX.X" AND type = page` |

## client Known Release Pages (fast lookup - no search needed)

| Version | Page ID | Status |
|---------|---------|--------|
| 1.19.2 | `5678071811` | Released - Apr 27, 2026 |
| 1.19.1/1.20.0 | `5632983041` | Drafting - May 2026 |
| 1.19.0 | `5479006209` | NOT released to PROD - superseded by 1.19.2 |
| 1.18.2 | `5639372801` | LG/X1/Rogers/Xbox - Apr 20, 2026 |
| 1.18.1 | `5478219777` | All platforms - Mar 2026 |
| 1.18.0 | `5341544449` | Paralympic Games release |

**Release notes title pattern:** `[Samsung][LG][X1][Rogers][Xbox] Release Notes - client - X.XX.X - [Month]`

---

---

# Adding a New Client

To add a new client space, append a new section following this template:

```markdown
---

# Client: [CLIENT NAME]

**Cloud ID:** `[instance].jira.com`
**Space key:** `[SPACE_KEY]`
**Root page ID:** `[root page ID]`
**Root URL:** `https://[instance].jira.com/wiki/spaces/[SPACE_KEY]/overview`

## [Client] Space Tree

[Map the space tree here with page IDs - run getConfluencePageDescendants on the root to populate]

## [Client] Routing Rules

| Question type | Page ID | Notes |
|---|---|---|
| ... | ... | ... |
```

Run `/confluence-lookup` after adding a new client to confirm routing works.

---

> **Skill verification:** Please ensure that the skill confluence-lookup.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
