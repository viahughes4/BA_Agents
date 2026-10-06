---
name: project-schedule
description: Create, maintain, and report on a project schedule - the phased or release-based plan of work, dates, owners, estimates, and milestones for a client engagement. Triggers on "create a project schedule", "build a project schedule", "update the schedule", "add to the schedule", "generate a schedule", "schedule status", "give me the schedule", "sync the schedule with Jira", "set up release schedule", "schedule from PRD", "timeline from SOW". Works standalone - a PRD, SOW, Jira board, Slack channel, or roadmap are all optional inputs, never requirements.
model: sonnet
---

# Skill: Project Schedule Manager

Creates and maintains the single project schedule for an engagement, and reports on it - the one place a PM goes to answer "what's the plan, and where does it stand." This skill is self-contained: it does not assume any other skill, sub-routine, or file convention exists in the repo it's installed into. Anything it would benefit from that isn't guaranteed to exist is listed in `README.md` as a future addition, not assumed present.

---

## Scope boundary

This skill does not do discovery, does not write a PRD, and does not create Jira tickets or stories. It sits after scope is understood (however loosely) and before day-to-day ticket execution:

```
Requirements (SOW / PRD / roadmap / chat) → [ THIS SKILL: schedule ] → Sprint tickets / stories
```

It is not gated on any upstream artifact existing. A PM can invoke it with a finished PRD, a roadmap file, a stack of Jira tickets, or nothing more than a description typed into chat.

---

## Context this skill can use, if present

Before asking the PM anything, check opportunistically for a per-engagement context or config file already established in the host repo (client name, Jira project key, team roster, capacity). If nothing is found, ask the PM directly for just what's needed and move on - never block on missing infrastructure.

---

## Process

### 0. Validation gate

Before finalizing (saving or reporting) any schedule:
- No unresolved placeholder date is left unflagged.
- No assumption about scope, dates, capacity, or estimates has been silently invented - anything inferred is marked so the PM can correct it.
- The PM has seen the output before it's treated as final.

### 1. New schedule or update?

Search for an existing schedule for this engagement (ask the PM where to look if this is the first run in this project - see step 9 for how the location gets remembered).

- **Found:** ask - "Found an existing schedule for [project]. Update it, or rebuild from scratch?" Default recommendation is **update** - never delete history. Skip to step 10 (Ongoing maintenance).
- **Not found:** continue to step 2.

### 2. Intake Q1 - Project type

The most important branch in the skill. Ask:

> **What type of project is this?**
>
> **A. New App/Feature Delivery** - building or porting something from requirements through development, QA, certification (if applicable), and a production release. Has a defined end date.
>
> **B. Standing Team** - an existing application already in production, with ongoing feature development, fixes, and recurring releases. Open-ended, cadence-driven.

If a context file already has a populated project-type-like field, propose the mapping instead of asking cold, and ask the PM to confirm or correct it.

This decision governs everything downstream - which reference to follow (`references/new-app-delivery.md` or `references/standing-team.md`), whether the schedule has an end date, and how updates work later. Never apply the New App/Feature Delivery lifecycle to a Standing Team engagement, and never treat a Standing Team release as a one-time project.

### 3. Intake Q2 - Jira (optional)

> **Does this project have a Jira board or existing tickets?**
>
> - **Yes, board:** provide the board URL.
> - **Yes, tickets:** provide the project key or ticket links.
> - **Not yet:** the schedule uses feature groupings instead; tickets can be linked later.
> - **Not sure / no:** proceed without Jira.

Retain the project key if given. Its absence must never block schedule creation.

### 4. Intake Q3 - Slack (optional)

> **Any Slack channel(s) I should watch for schedule-relevant updates or risks?**

If given, retain the channel(s). If not, skip Slack checks entirely for this schedule - don't ask again unless the PM brings it up.

### 5. Intake Q4 - Scope source

> **What should I use to build this schedule's scope?**
>
> - A PRD or SOW (point me to it, or paste it in)
> - The existing roadmap/backlog
> - Jira tickets (already covered above, if provided)
> - I'll describe the features/releases directly
> - I don't know yet - just set up the structure

If a PRD/SOW is given, check it specifically for a stated project start and end date. If it doesn't state them, ask the PM directly - don't leave them blank and don't guess. Merge multiple sources if more than one is offered.

### 6. Intake Q5 - Estimates

The schedule can't be split into sprints or release windows without knowing how big each piece of work is. For whatever's in scope:

1. **If it has a linked Jira ticket:** use the ticket's own estimate (story points or whatever unit the team already uses). Don't ask the PM to re-estimate something already sized.
2. **If it doesn't:** ask the PM for a **feature/epic-level** estimate, not task-level - that's what a PM actually has before dev breakdown happens. The skill then breaks each feature into a task list itself (see `references/new-app-delivery.md` / `references/standing-team.md`) sized to roughly sum to the estimate given.
3. **If the PM has neither and has provided a generic estimation reference** for this domain, use it as a last-resort baseline - always flagged in the output as "baseline estimate, not team-confirmed," never presented as firm.

If an engineering estimate catalog is in play (source priority step 3 in `references/estimates.md`), ask once, up front, before any catalog matching: **"Will this build use Accedo Assemble/Elevate, or is it Ground-up (fully custom)?"** These are different bodies of work in the catalog, not interchangeable - this answer decides which half of the catalog is even eligible to match against. See `references/estimates.md`'s Step 0 for the full handling.

Ask once, up front: **"Story points or ideal days?"** If points, also ask for team velocity (points per sprint). If days, use the developer-capacity-per-sprint figure gathered during schedule build (see the type-specific reference) directly.

### 6b. Confirm estimate matches before building

Before laying out a single sprint or task, compile a short list of every in-scope item that doesn't have a solid estimate: anything still `TBD` (no match found at all) and anything matched only via a weak, generic, or proxy match (not an exact/near-exact catalog hit). Present this list to the PM as its own checkpoint - not scattered as flags the PM only discovers while reading a finished 7-sprint schedule:

```
Before I build this, here's what doesn't have a confident estimate:

Modality: [Assemble/Elevate | Ground-up] - catalog matches below are restricted to this track only.

No match found:
- [Feature] - checked [categories checked], within [modality]

Weak/proxy match only:
- [Feature] - matched to "[catalog requirement]" ([confidence]) - is this the right comparison, or is there a better one?

Platform exceptions found for in-scope platforms (per estimates.md):
- [Feature] on [Platform]: base [X]h → platform-adjusted [Y]h ([ratio]), Keep?: [Y/N]

Want to correct any of these before I build the schedule, or proceed with what's here (all still flagged in the output)?
```

Wait for a response before continuing to step 7. If the PM says proceed, carry the same flags into the schedule itself (per `estimates.md` and `document-format.md`'s Confidence column) rather than treating this checkpoint as resolving them.

### 7. Build the schedule

Hand off to the type-specific reference:
- **New App/Feature Delivery** → `references/new-app-delivery.md`
- **Standing Team** → `references/standing-team.md`

Both produce the same underlying document shape - see `references/document-format.md` for the shared structure, color-coding convention, and the two required sections (detailed table + summary rollup).

If the PM explicitly asked for certification and/or client UAT, or the PRD clearly requires them, include those phases. If genuinely unclear on a New App/Feature Delivery project, ask directly rather than guessing either way.

### 8. Validate before saving

- Total duration vs. any stated deadline - if the schedule runs longer, flag it: "Schedule shows [X]; deadline is [Y]. Options: descope, extend, or add capacity." Never silently compress dates to fit.
- Every client-side or third-party dependency has a named owner and a target date.
- QA time is present within every sprint/release, never zeroed out.
- A buffer/contingency phase exists near the end of a New App/Feature Delivery schedule.

### 9. Save

First time in this project: ask the PM where to save the schedule. Record that location in the schedule file's own header so future invocations find it without asking again. After that, update the same single file in place - never create a new dated snapshot file for a routine update. Only start a genuinely new file for a materially new engagement.

Every save also writes a companion HTML file next to the markdown (same name, `.html`) - see `references/html-export.md`. This is what's actually screenshotted or pasted into a slide deck: real color-coded rows instead of emoji markers, plus an inline SVG milestone timeline built from the schedule's 🟩 milestones.

### 10. Ongoing maintenance

Support all of the following without re-running full intake:
- Add an upcoming release or phase
- Move a feature/task between releases or phases
- Add or remove a task, with or without a linked ticket
- Adjust a development or QA window
- Update a date
- Mark a task done / carry it forward
- Reassess whether planned scope fits the available window

**Save behavior:** a change to dates, scope, or milestones gets summarized back to the PM for a quick confirm before writing ("Moving Feature X to October, pushing the release milestone to 10/15 - save this?"). A pure status toggle (mark done, mark blocked) saves directly - low-risk, easily reversed.

### 11. Every invocation - opportunistic checks

Every time this skill runs (not just when the PM explicitly asks for a sync), run both of the following before doing whatever else was asked. Follow `references/jira-slack-checks.md` for full detail. In short:

- **Jira check** (only if a project/board was given in intake, only for tasks with a linked ticket): compare actual ticket status against what the schedule implies is expected. Surface mismatches; never auto-apply.
- **Slack check** (only if a channel was given in intake): scan for anything schedule-relevant - a mentioned slip, a raised risk, a decision that changes scope. Surface it; never auto-apply.

Present anything caught as a short flagged list before continuing: "Before we get to that - I noticed [X] in Jira/Slack. Want me to reflect that in the schedule?"

### 12. Reporting

"Give me the schedule," "what's the status," "schedule for the client" - all point to the same document. There is no separate redacted client copy; the schedule's summary rollup (see `references/document-format.md`) is written to be shareable with anyone by default, since it never carries ticket-level or risk detail to begin with.

---

## Quality rules

- Dates are specific (`2026-09-30`), never vague ranges.
- Every dependency and blocker has a named owner and a target date.
- Recommendations are actionable: "do X by [date], else descope Y" - not just "this is at risk."
- Never delete history - completed phases/releases stay in the document.
- Never invent an end date for an open-ended Standing Team engagement.
- Jira ticket status or a Slack mention never moves a schedule date or changes a commitment without the PM confirming.
- If the PM says "track this" with no ticket yet, add it anyway.
- An estimate sourced from a generic baseline (not a ticket or PM input) is always labeled as such.
- Complex or high-risk work always carries a 15% buffer on its estimate, and a realistic buffer phase (also ~15% of dev duration) is always included before any launch/release, to account for unexpected edge-case bugs - see `references/estimates.md`.

---

## Related files

- `references/new-app-delivery.md` - phase/sprint structure, task breakdown, estimate bin-packing, QA/cert/buffer handling for defined-end projects
- `references/standing-team.md` - release-cadence structure, per-release estimate handling, cert-per-platform handling for open-ended engagements
- `references/estimates.md` - estimate source priority, unit handling, and how feature-level estimates become sprint-fit tasks
- `references/document-format.md` - shared document shape, color-coding convention, the detailed table and the summary rollup
- `references/jira-slack-checks.md` - the opportunistic drift/signal check that runs on every invocation
- `references/html-export.md` - the companion HTML file written alongside every markdown save, including the milestone timeline
- `references/engineering-estimate-catalog.md` - Accedo's measured effort-per-requirement reference, ships with the skill (New App/Feature Delivery only)
- `references/engineering-base-catalog-raw.md` - the full unprocessed source export the catalog above is curated from
- `README.md` - what this skill needs from a host repo, what it doesn't require, and future additions
