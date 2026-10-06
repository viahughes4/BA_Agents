---
name: roadmap-sync
description: Use this skill when the user wants to create, edit, update, or build a roadmap. Triggers include "edit roadmap", "update roadmap", "build roadmap", "create roadmap", "add to roadmap", "roadmap changes", "roadmap sync", "sync roadmap", "sync the roadmap", "roadmap review", "update the timeline", "mark as blocked", or when the user shares a Miro CSV, Gantt chart, screenshot, or any roadmap artifact. Works across all clients. Stores roadmap data as structured markdown in the client's repo folder.
version: 1.0.0
---

# Skill: Roadmap Sync

Ingests roadmap input in any format and stores it as structured markdown in the client's repo. Supports initial creation, incremental updates, and Gantt-level detail for complex features.

**Never write to any file without showing a diff and getting explicit user confirmation first.**

---

## Supported Input Formats

| Input | How to handle |
|-------|--------------|
| Miro CSV export | Parse rows into roadmap items |
| Screenshot (Miro, Smartsheet, any visual) | Read image, extract items and timeline |
| Pasted table or text | Parse structure from formatting |
| Verbal instruction | e.g. "move RDI E2E to August, mark Figma as blocked" - apply as targeted update |

---

## File Structure (per client)

```
[client-root]/
└── roadmap/
    ├── roadmap.md                    ← high-level roadmap (dev assignments, milestones, visual timeline)
    ├── release-schedule.md           ← standalone release plan (upcoming + past releases, scope, status, sequencing notes)
    ├── roadmap-client-facing.md      ← client-facing version (client only) - simplified, no internal notes
    ├── [feature]-gantt.md            ← detailed Gantt for features needing it (e.g. rdi-gantt.md)
    └── ...
```

**release-schedule.md** is the single source of truth for release timing. It is separate from roadmap.md so it can be referenced or shared standalone. When any release date, scope, or status changes, update BOTH roadmap.md (the one-line summary) AND release-schedule.md (the full detail). roadmap.md links to release-schedule.md rather than duplicating the full table.

**Client root paths:**

| Client | Root | Roadmap file | Client-facing file |
|--------|------|--------------|-------------------|
| client | `product-development/product/customers/accounts/[client]/` | `roadmap/roadmap.md` | `roadmap/roadmap-client-facing.md` |
| VIDAA | `product-development/product/customers/accounts/vidaa/` | `roadmap/roadmap.md` | N/A |
| JMMI | `product-development/product/customers/accounts/jmmi/` | `roadmap/roadmap.md` | N/A |

If the client folder or roadmap files don't exist yet, create them before proceeding.

**client note:** When updating `roadmap.md`, always check whether the same change should be reflected in `roadmap-client-facing.md`. Internal notes, blocked reasons, and resource details should not appear in the client-facing file.

---

## Roadmap Item Schema

Every item in `roadmap.md` must capture:

| Field | Description |
|-------|-------------|
| **Feature / Workstream** | Name of the feature or work item |
| **Quarter / Target** | e.g. Q2 2026, July 2026, H2 2026 |
| **Start → End** | Date window if known |
| **Status** | On Track / At Risk / Blocked / Complete / Not Started |
| **Blocked reason** | If blocked - reason and date flagged |
| **Owner** | Team or person responsible |
| **Jira ticket(s)** | Linked ticket keys if available |
| **Notes** | Any relevant context or decisions |

---

## Gantt Item Schema

For `roadmap/[feature]-gantt.md`, capture:

| Field | Description |
|-------|-------------|
| **Milestone** | Name of the milestone or task |
| **Start date** | Target start |
| **End date** | Target end |
| **Status** | On Track / At Risk / Blocked / Complete |
| **Dependencies** | What this depends on |
| **Owner** | Who is responsible |
| **Notes** | Blockers, risks, decisions |

---

## Process

### 1. Load context and identify client
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths. If context-loader is unavailable or returns no project context, ask the user to confirm the client name, Jira project key, and team roster before proceeding. If the client is not clear from context, ask before proceeding.

### 2. Load existing roadmap (if any)
Read `[client-root]/roadmap.md` and any relevant Gantt files. Understand the current state before processing input. If roadmap.md does not exist yet, skip the diff comparison and treat all parsed input as new items. Proceed to Step 3.

While loading, flag any items where:
- End date is in the past AND status is not Complete, Done, or Blocked → **potentially stale**
- Start date is in the past AND status is Not Started or To Do → **should have started**

Surface these as a brief warning before presenting the diff so the user can decide whether to update them as part of this sync.

### 3. Parse input
Extract all items from the provided artifact (CSV, screenshot, text, or verbal). Map each item to the schema above. If a field is missing, mark it as N/A.

### 4. Generate diff
Compare parsed input against existing roadmap. Show the user exactly what will change:

```
## Roadmap Diff - [Client] - [Date]

**New items:**
- [Feature] - Q3 2026 - Not Started

**Updated items:**
- [Feature]: Start date moved from June - August
- [Feature]: Status changed On Track - Blocked (Figma not delivered)

**Removed items:**
- None

**No changes:**
- [list unchanged items]
```

### 5. Wait for confirmation
Do not write anything until the user explicitly confirms. If they want adjustments, revise the diff first.

### 6. Validate Mermaid syntax before writing
Before writing any file, verify the Mermaid gantt block passes these checks:
- `axisFormat` line contains only `%b %Y` - no appended text, filenames, or image paths
- No special characters in task or section names: no `·`, `-`, `:`, `(`, `)`, `/`, `&`
- No orphan lines inside the gantt block (lines that are not section headers, task entries, or directives)
- All dates use `YYYY-MM-DD` format
- No task name contains a `-` character (use a space instead)

If any check fails, fix the Mermaid before writing.

### 7. Write files
On confirmation, update the following files as needed:
- `roadmap.md` - update the one-line release summary and any milestone/dev assignment rows
- `release-schedule.md` - update the full release table rows (scope, status, target, key dependency). This is the primary place for release detail.
- `roadmap-client-facing.md` (client only) - check whether client-facing dates or scope need updating
- Any relevant Gantt files

**release-schedule.md update rule:** Any change to a release target date, scope, or status must be reflected in release-schedule.md. Never update roadmap.md release rows without also updating release-schedule.md.

### 8. Generate HTML export
Run `python3 scripts/md-to-html.py <saved-file-path>` for every .md file written (roadmap.md, release-schedule.md, roadmap-client-facing.md, and any Gantt files). This produces a styled HTML version in an `html/` subfolder. Report the HTML path to the user.

---

## Roadmap Item Schema - Additional Fields

Every roadmap item should also capture (where known):

| Field | Description |
|-------|-------------|
| **North Star** | Which client goal this maps to: 👥 Goal 1 (User Retention), ▶️ Goal 2 (Content Discovery), 📺 Goal 3 (Longer Sessions), or 🔧 Maintenance |
| **Narrative** | How to frame this item to client in terms of the north star (especially for maintenance tickets) |

---

## Output Format - roadmap.md

```markdown
# [Client] Roadmap - Last updated: [date]

## Release Goals & Upcoming Milestones

> What are we trying to ship, by when, and what's blocking us? Updated weekly.

### Next 2 Weeks

| Date | Goal | Status | Owner | Blocker |
|------|------|--------|-------|---------|
| [date] | [release or milestone] | [status] | [owner] | [blocker or -] |

### Release Plan

| Release | Target | Scope | Status | Key Dependency |
|---------|--------|-------|--------|----------------|
| [release name] | [date] | [what ships] | [status] | [dependency] |

### What's Not Done Yet

| Item | Why It Matters | Owner | ETA |
|------|---------------|-------|-----|
| [item] | [north star connection] | [owner] | [date or TBD] |

---

## Visual Timeline

[Mermaid Gantt - see Mermaid Rules below]

## Developer Assignments

| Feature | Target | Start → End | Status | Jira | North Star | Narrative |
|---------|--------|-------------|--------|------|-----------|-----------|
| [name] | [Q/date] | [start] → [end] | [status] | [key] | [goal emoji] | [how to frame to CBC] |

## [Quarter / Period]

| Feature | Target | Start → End | Status | Owner | Jira | Notes |
|---------|--------|-------------|--------|-------|------|-------|
| [name] | [Q/date] | [start] → [end] | [status] | [owner] | [key] | [notes] |

## Blocked Items

| Feature | Blocked Since | Reason | Owner |
|---------|--------------|--------|-------|
| [name] | [date] | [reason] | [owner] |
```

**When updating the roadmap, always refresh the Release Goals & Upcoming Milestones section to reflect:**
- Any new releases or target dates
- Any newly blocked items
- The "Next 2 Weeks" table based on current date
- The "What's Not Done Yet" table for anything without a confirmed ship date

---

## Output Format - [feature]-gantt.md

```markdown
# [Feature] Gantt - [Client] - Last updated: [date]

## Visual Timeline

[Mermaid Gantt - see Mermaid Rules below]

---

## Detail Table

| Milestone | Start | End | Status | Owner | Dependencies | Notes |
|-----------|-------|-----|--------|-------|--------------|-------|
| [name] | [date] | [date] | [status] | [owner] | [deps] | [notes] |
```

---

## Mermaid Gantt Rules

All roadmap and Gantt files must include a Mermaid diagram. Follow these rules exactly to avoid syntax errors:

```
gantt
    title [Client] Roadmap [Year]
    dateFormat YYYY-MM-DD
    axisFormat %b %Y

    section [Section Name]
    [Task name]     :[modifiers,] [id,] startDate, endDate
```

**Modifiers:** `done` (green), `active` (blue), `crit` (red/blocked), no modifier (gray)

**Milestone syntax:**
```
GATE - [Name]    :milestone, [id], [date], 1d
```

**Syntax rules - these will break the diagram if violated:**
- No special characters in section names or task names: no `·`, `-`, `:`, `(`, `)`, `/`
- Use `-` instead of em dash
- Use `and` instead of `&`
- Use `1d` for milestone duration - never `0d`
- Do not combine `done` with `milestone`
- Do not use `-` inside task names (e.g. write `getHasAccess TouTV` not `getHasAccess - TouTV`)
- `dateFormat` must be `YYYY-MM-DD` - all dates must match this format

**roadmap.md Gantt:** High-level by workstream. One bar per major feature/workstream. Sections = workstreams (RDI, Accessibility, Long Press, etc.).

**[feature]-gantt.md Gantt:** Detailed by phase. One bar per task/milestone. Sections = phases.

---

## Rules

- **Never update files without explicit user confirmation.**
- Always show the diff before writing.
- If the input is a screenshot, describe what you read before producing the diff so the user can catch any misreads.
- Preserve existing items not mentioned in the new input - do not delete unless the user explicitly removes them.
- If a verbal update is ambiguous (e.g. "push RDI back"), ask for the specific new date before applying.
- Always update the Mermaid Gantt when roadmap dates or statuses change.
- **Every roadmap entry must have at minimum:** Feature/Workstream, Quarter/Target, and Status. All other fields default to N/A if not provided - never leave a row half-empty or skip required columns.
- **Ambiguous dates must be resolved before writing.** If the user gives a vague timeframe ("next quarter", "soon", "later this year"), ask for a specific month before applying the change. Do not interpret quarter boundaries yourself.
- **Always save to the client-specific path** from the File Structure table above. Never save to a generic or unspecified location.
- **After saving, always generate HTML:** run `python3 scripts/md-to-html.py <saved-file-path>` immediately after writing the markdown file. This produces a styled HTML version in an `html/` subfolder alongside the markdown.


> **Skill verification:** Please ensure that the skill roadmap-sync.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
