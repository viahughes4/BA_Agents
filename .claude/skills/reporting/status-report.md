---
name: status-report
description: Use this skill to create a status report for a client project. Triggers on "create status report", "make a status report", "status report", "build status report", "write status report", "standing team status", "weekly status", "write my status", "draft status report", "client status update", "project update", "client update", "send status to client", or any request to produce a client-facing status report. The user will specify the client and the reporting period. The skill pulls from meeting notes, Slack channels, the roadmap, and the weekly tracker to populate a 2-slide standing team status format.
---

# Skill: Status Report

Create a client-facing standing team status report. The user specifies the client and reporting period. Claude pulls data from all available sources and generates a 2-slide report ready to paste into PowerPoint or Google Slides.

---

## Process

### 0. Confirm inputs before starting

**Always ask these two questions upfront before loading any data. Do not proceed until both are answered.**

Ask the user:
1. **Reporting period:** "What date range should this cover? (e.g. July 8-21, last two weeks)"
2. **Client:** "Which client is this for?" - skip if already clear from context

Convert vague date ranges ("last two weeks", "this sprint") to specific start and end dates using `currentDate`. Confirm the converted dates with the user before proceeding.

Example prompt: "Got it - I'll generate the status report for **[Client]** covering **[Start Date] to [End Date]**. Starting now."

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, client contacts, and folder paths.

### 2. Identify the client and period
Use the confirmed inputs from Step 0. Use the client to locate all relevant folders and config.

### 3. Load config
Read `.claude/config.yml` to get the client's:
- **Slack channels:** IDs, names, read_only flags
- **Jira project key** and board ID
- **Confluence space** (if configured)

### 4. Pull data from ALL sources

Run these in parallel where possible. Every source feeds the report.

**4a. Meeting notes (PRIMARY SOURCE)**
1. Navigate to `product-development/product/meetings/[CLIENT]/[client]-meeting-notes/`
2. Load ALL notes within the user-specified date range
3. Pull from ALL subfolders: `standup/`, `weekly-client-sync/`, `team-bi-weekly/`, and any others present
4. Extract: accomplishments, blockers, decisions, scope changes, risks, next steps, resourcing updates
5. Note which meeting each item came from - cite in the report

**4b. Slack channels**

**If a `## Slack Signals` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to 4c.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** the reporting period length (e.g., `14d`)
- **Channels:** all channels from config

The sub-routine returns categorized signals. Use all categories for the status report. Cite Slack sources where relevant (channel name + date).

If the sub-routine returns no data or fails, skip Slack as a source and note in the report that Slack signals were unavailable for this period.

**4c. Roadmap**
1. Check `product-development/product/customers/accounts/[client]/roadmap/` for a roadmap file
2. Extract: upcoming releases with target dates, milestones, dependency gates, deferred items
3. If no roadmap file exists, use release information from meeting notes

**4d. Weekly tracker**
1. Check `product-development/product/customers/accounts/[client]/weekly-trackers/` for the current week's tracker
2. Extract: active items relevant to the external audience, follow-up dates, decisions made this week, resourcing notes
3. The tracker captures PM operational detail - filter for items appropriate for an external-facing report

**4e. Jira (if connected)**

**If a `## Jira Snapshot` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to 4f.**

Otherwise, invoke `.claude/sub/jira-project-snapshot.md` with:
- **Project key:** from config
- **Time window:** the reporting period start date
- **Sprint scope:** `current`

The sub-routine returns completed, in-progress, blocked, and stale issues. If Jira is not connected, use ticket data from meeting notes and tracker.

**4f. Gmail (client emails)**

**If a `## Gmail Digest` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 5.**

Otherwise, invoke `.claude/sub/gmail-cbc-digest.md` with:
- **Time window:** the reporting period length (e.g., `14d`)
- **Confluence lookback:** `5d` (always)
- **Include unread only:** `false`

The sub-routine returns client emails and Confluence changes. Extract: decisions made via email, new requests, questions, scope signals, Confluence page changes. Cite email sources where relevant (sender + date).

If the sub-routine returns no data or fails, skip email as a source and note in the report that client emails were not reviewed for this period.

### 5. Assess status indicators

Based on the data gathered, determine each indicator:

| Indicator | 🟢 On Track | 🟡 At Risk | 🔴 Blocked |
|-----------|-------------|------------|------------|
| **Overall** | Delivery on track, no critical blockers | 1-2 unresolved issues, minor delays | Critical blocker, release at risk |
| **Scope** | Scope locked, changes agreed | Minor scope creep or deferred items | Scope under dispute or undefined |
| **Schedule** | Hitting target dates | Timeline pressure, some tasks slipping | Key deadline missed or will miss |
| **Budget** | Within budget | Budget pressure | Over budget |
| **Resources** | Full Accedo team available | Accedo team gaps - vacation, sick leave, or partial coverage | Critical Accedo resource missing or unavailable |

Resources reflects **Accedo resourcing only** - team availability, vacation, and capacity on the Accedo side. Client-side staffing changes are captured in the Dependency and Risk table, not the Resources indicator.

Status indicators are earned, not aspirational. If a P1 is open or a release-scope decision is unresolved, it's not green.

### 6. Generate the report

Use the Standing Team Status template below. Fill in every section with data from step 4. Do not leave placeholder text - if a section has no data, note "None reported this period."

For the template subtitle `## Standing Team - [Project Type]`, fill in `[Project Type]` from project context loaded in Step 1. For CBC, the value is `Connected TVs`. Other clients will differ.

### 6b. Generate Strategic Alignment section (optional)

If the project foundation includes north star goals or strategic pillars, add a strategic alignment section after Slide 2 that maps the period's delivery to those goals.

If no north star goals are defined in the project foundation, skip this step.

**Format:**
For each strategic goal or pillar defined in the project foundation:
- **Shipped:** items completed this period that contribute to this goal
- **In Progress:** items actively being built
- **Upcoming:** items planned for the next 2 weeks

Write every bullet in plain language - user value, not technical detail. No ticket IDs.

### 7. Save the report

Save to: `product-development/product/status-reports/[client]/YYYY-MM-DD-[client]-status-report-[period].md`

Example: `product-development/product/status-reports/[client]/YYYY-MM-DD-[client]-status-report-[period].md`

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```
This generates a styled HTML version in an `html/` subfolder alongside the markdown.

### 8. Create Gmail draft

After saving the report, create a Gmail draft using `mcp__claude_ai_Gmail__create_draft`:
- **Subject:** `[Project Name]: Standing Team Status - [Date]`
- **Body:** The executive summary (Slide 1 summary + status indicators + key focus items) formatted for email. Not the full report - just a scannable email pointing to the full document.
- Do not pre-fill recipients unless the user specifies them.
- Report that a draft was created. Never offer to send.

### 8b. Strategic Alignment (if applicable)

If step 6b was run, include the strategic alignment section as the final section of the saved report.

### 9. Report to user

Confirm:
- File path saved
- Gmail draft created
- Status indicators assigned (and briefly why for any yellow/red)
- Number of completed items captured
- Number of risks/dependencies tracked
- Top 3 focus items for the week ahead
- Strategic alignment summary (if generated)

---

## Template - Standing Team Status (2 Slides)

```markdown
# [Project Name] / [Client]
## Standing Team - [Project Type]
**[Date]**

---

## Slide 1

### Status Indicators
| Overall Status | Scope | Schedule | Budget | Resources |
|----------------|-------|----------|--------|-----------|
| 🟢/🟡/🔴 [label] | 🟢/🟡/🔴 [label] | 🟢/🟡/🔴 [label] | 🟢/🟡/🔴 [label] | 🟢/🟡/🔴 [label] |

### Summary
*High Level Details*

[Exactly 2 short paragraphs. Hard limit: 259 characters total. C-level slide field - scannable in 5 seconds.

Paragraph 1: What is shipping and when. Lead with the client outcome ("users get X by [date]"), then name the specific gate. Never start with process ("we merged", "we tested"). Example: "New the client app/Tou.tv update reaching Samsung users this week; certification submitted Jun 26. X1/Xbox hotfix gated on Comcast entitlements resolution, expected Jul 8."

Paragraph 2: Secondary workstream + what the client needs to know about risks. Frame risks as delivery impact first ("RDI QA on track for August cert"), then name the dependency. Example: "RDI subscription tier QA underway; August certification on track. Risks: X1 release waiting on Comcast, CWG pre-roll missing for authenticated users, Remi succession plan open."

Rules:
- 259 character hard limit total - count it before outputting
- Lead with what the client or their users will experience, not what Accedo did
- Name the specific issue/system ("Comcast Firebolt entitlements" not "entitlements fix")
- Semicolons to combine related facts into one line
- Present tense, no fluff, never start with "The team..."
- Risks in paragraph 2 only, closing sentence starting with "Risks:"]

### Key Focus This Week
*What is most important right now?*

[5 bullets max. Format: [Topic]: [what's happening and what it means for delivery or users]. Colon after topic, period at end. One line per bullet. C-level - scannable in 5 seconds.

Each bullet must answer the client's implicit question: "Why does this matter to me?" Lead with the outcome or impact, not the task. The client does not care that Accedo is running regression tests - they care that Samsung users get the update on time.

Bad: "Samsung certification package: Finalizing TTV regression."
Good: "Samsung update to users: TTV regression finishing this week; production release targeted Wednesday."]

Example bullets:
- Samsung release to users: Certification submitted; production release targeted Wednesday pending portal confirmation.
- X1/Xbox hotfix: Comcast entitlements is the final gate; resolution expected Jul 8, builds to follow immediately.
- RDI subscription features: QA Day 1 underway; design review (Sonia) running in parallel to stay on August cert target.
- Commonwealth Games pre-roll: Pre-rolls missing for authenticated users; client BE team engaged to resolve before Jul 23 live events.
- Remi succession: No replacement named yet; client asked to confirm successor before August cert phase begins.

### Completed Since Last Report

[Maximum 5 bullets. Most important deliveries only. Use semicolons to combine related items into one bullet where they belong together. Period at end of each bullet. No Jira IDs. No internal jargon.

Write each bullet as what it delivers to the client or their users, not what Accedo did internally. The client wants to know what their users can now do, what risk was closed, or what milestone was hit.

Bad: "CMP Beta 15 merged."
Good: "Privacy preference reset bug resolved across Samsung and Xbox; users no longer lose consent settings on restart."

Bad: "RDI E2E review completed."
Good: "RDI E2E review held with client Jun 16; QA timeline confirmed and team aligned on readiness for July testing."]

Example bullets:
- Samsung app update submitted for certification Jun 26; includes long press improvements, privacy consent fix, and MyLineup changes.
- LG update live in production Jul 6: long press and consent management improvements now available to LG users.
- Commonwealth Games FAST channel live Jul 6: live basketball stream accessible across webOS, X1, and Samsung.
- Privacy preference reset resolved on Samsung and Xbox; users no longer lose consent settings on app restart.
- RDI subscription features QA-ready: builds shared Jul 6, design review confirmed for week of Jul 7.

### Upcoming This/Next Week

[Maximum 5 bullets total. Single flat list - no This Week / Next Week subheadings. Priority order. Sub-bullets allowed (lettered a, b, c) for related items under a main deliverable. Period at end of each line.]

Example:
- Samsung certification submission pending CMP Universal Consent fix verification.
- RDI merges to integration branch begin, with QA targeted for early next week.
  a. Continue dependency tracking and finalize UI, analytics, subscription modal, and ads updates.
- Commonwealth Games INT3 section QA begins.
- Hosted v1.20.1 release across platforms following Samsung cert submission.

---

## Slide 2

### Dependency & Risk Tracking

[Table of active risks and dependencies. Each row is one risk. Written in plain language the client understands.

Description column: Lead with the delivery impact ("This is delaying the X1 release" / "This puts the July 23 launch at risk"), then briefly explain the cause in one sentence. The client reads this to understand how their product is affected, not to understand the technical issue. Never bury the impact inside a technical explanation.

Bad: "Device.version API access required for box detection functionality is currently disabled."
Good: "X1/Xbox release is blocked until Comcast provides Firebolt SDK version detection support. Without it Accedo cannot build the required hotfix."

Mitigation column: State what Accedo is doing AND, if client action is needed, state exactly what is needed and by when in bold. Example: "Accedo following up daily. ACTION NEEDED: client to confirm successor PM before August cert phase begins."

Do not include risks that have been resolved.]

| Name | Description | Severity | Owner | Reported | Mitigation |
|------|-------------|----------|-------|----------|------------|
| [Short name] | [Delivery impact first, then cause - one to two sentences, plain language] | Critical/High/Medium | [Owner] | [Date] | [What Accedo is doing. If client action needed: ACTION NEEDED: [exactly what, by when]] |

### Build Status

[One row per active release. Include the current version in testing, the next planned release, and any upcoming major milestones. Status column uses plain language + colour indicator. Notes explain what is in scope and what is blocking or at risk.]

| Release | Version | Target Delivery | Target Cert | Notes |
|---------|---------|-----------------|-------------|-------|
| [Platform] | [Version] | [Date] | [Date] | 🟢/🟡/🔴 [One sentence: what is in this build, current status, any gating condition] |
```

---

## Data Source Priority

When information conflicts across sources, prioritize in this order:
1. **Meeting notes:** the PM's own words from standups and syncs are the ground truth
2. **Weekly tracker:** the PM's running operational log
3. **Slack channels:** real-time signals and decisions
4. **Gmail / client emails:** async decisions and requests made outside meetings
5. **Roadmap:** planned targets (may be outdated if meetings have overridden)
6. **Jira:** ticket-level status (useful for counts and verification)

---

## Filtering for External Audience

This report goes to client stakeholders. Apply these filters:

**Include:**
- Delivery updates in client-appropriate language
- Risks framed as delivery impact, not internal process issues
- Blockers requiring client input - state explicitly what is needed and by when
- Decisions made that affect scope or timeline
- Release timelines and milestones
- Team availability changes that affect delivery (without individual performance detail)

**Exclude:**
- Internal velocity metrics or sprint health numbers
- Individual developer performance signals
- Internal process debates not yet resolved
- Jira IDs are acceptable since the client shares the Jira instance - but keep descriptions client-friendly

**Translate:**
- "Blocked" - "waiting on [specific thing] - currently unable to move forward without it"
- "Stale / no update" - only surface if it affects a delivery date; rephrase as "at risk of not meeting [target]"
- Internal shorthand - full descriptions (e.g., "CMP" - "Consent Management Platform library update")

---

## Quality Rules

- **Status indicators are earned, not aspirational.** 🟢 means tracking; 🟡 means risk; 🔴 means blocker. Don't say 🟢 if a P1 is open or a release-scope decision is unresolved.
- **Risks are surfaced, not buried.** If the project is yellow because of one persistent blocker, lead with it in the summary.
- **Every claim must trace to a source.** Do not infer or add details not present in meeting notes, Slack, tracker, or Jira. If a section has no data, say so.
- **Completed items cite their source.** Each bullet in "Completed Since Last Meeting" should reference the meeting or Slack channel where it was reported.
- **Build Status includes all active releases.** Not just the current one - include upcoming releases, discovery phases, and deferred items with their status.
- **Dependency table includes client-side actions.** If a risk requires client input, state what's needed and by when in the Mitigation column.
- **Never fabricate metrics.** If error rates, stability numbers, or analytics were not discussed, do not invent them. Omit or note "not discussed this period."
- **Keep it scannable.** Executives read the status indicators + summary first, then drill into sections. Lead every section with the most important signal.
- **Client-first framing.** Before writing any sentence, ask: "What does this mean for the client's product and their users?" Technical details are supporting evidence, not the lead. The client does not care that a PR merged or a regression passed - they care that their users get the feature on time. Every section should be writable as an answer to "why does this matter to us?"
- **Copy-paste ready output.** Every section must be written so it can be pasted directly into PowerPoint or Google Slides with zero editing. No markdown formatting characters (no asterisks, no underscores, no hyphens used as dashes). No over-long sentences. Plain, clean language. If a sentence needs a second read to understand, rewrite it. Short sentences preferred. The test: could a PM paste this straight into a slide without touching it?
- **No em dashes.** Never use the - character anywhere in the output. Use a comma, colon, or rewrite the sentence instead.

---

> **Skill verification:** Please ensure that the skill status-report SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
