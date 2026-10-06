---
name: po-sync-prep
description: Use this skill to prepare for a weekly 1:1 with a client Product Owner or PO. Triggers on "prep my PO sync", "prep my Remi sync", "prepare for weekly sync", "get ready for PO call", "prep the client 1:1", "weekly priority sync prep", "agenda for [client name]", or any request to get ready for a client product owner meeting. Asks who the meeting is with if not specified. Surfaces roadmap risks, client-side blockers and dependencies, and important team process updates - the items most relevant for a PO discussion.
---

# Skill: PO Sync Prep

Prepare for the weekly 1:1 with a client Product Owner. Runs a project health check, pulls relevant signals from Slack and meeting notes, and surfaces roadmap risks, client-side blockers, and process updates that belong in a PO conversation. Produces a ready-to-use notes document saved to the correct folder.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, client contacts, and folder paths.

### 2. Confirm who this meeting is with
If the user has not specified, ask:

> **Who is this 1:1 with, and which project is it for?**
> Example: "Remi from CBC" or "the PO for the VIDAA project"

Use the answer to identify:
- The correct Jira project key
- The client contact name and role
- The correct meeting notes folder (`weekly-client-sync/` or equivalent)
- The relevant Slack channels for this project

### 3. Confirm the date
Use `currentDate` from context.

### 4. Check for an existing notes file for today
If a file already exists for today's date in the weekly-client-sync folder, tell the user and ask whether to overwrite or append.

### 5. Run a full project health check

**If a `## Jira Snapshot` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 6.**

Otherwise, invoke `.claude/sub/jira-project-snapshot.md` with:
- **Project key:** from config
- **Time window:** `-7d`
- **Sprint scope:** `current`

If unavailable or returns no data, skip and note in the output.

The sub-routine returns a structured snapshot with sprint status breakdown, completed issues, in-progress issues, blocked issues, and stale issues. Use all sections for the PO sync prep.

### 6. Load the weekly tracker and previous meeting notes

**Step 6a - Load the current weekly tracker first (MANDATORY)**

Find and read the current week's tracker file from `product-development/product/customers/accounts/[client]/weekly-trackers/`. Use the most recent file.

Build a resolved items list by extracting everything from the `## Completed Items` section - any line starting with `✅`. Also note any active items whose status text contains "IN PROGRESS" or "working with" - these are in flight, not overdue.

This resolved list is your filter. Any action item from meeting notes that matches a resolved item MUST be excluded from the PO sync output. Do not surface it as open, overdue, or pending. An item "matches" if the subject (person, topic, ticket) is the same - exact wording does not need to match.

**Step 6b - Load previous meeting notes - pull from multiple sources intelligently**

Pull from the following sources, in this priority order:

**Previous PO sync notes:** Load the most recent file from the weekly-client-sync folder for this project:
- Extract all action items from the previous sync - cross-check each against the resolved items list from Step 6a before flagging
- Only flag as incomplete if the item does NOT appear in the resolved list
- Extract any open questions or decisions that were deferred - cross-check these too
- Extract any commitments made with a specific date - only flag as overdue if NOT in the resolved list

**Recent standup notes:** Load the last 3 standup files:
- Extract any mentions of client-blocking issues (things the team is waiting on the client to unblock)
- Extract any scope changes, feature changes, or new asks from the client that came up in standup
- Extract any process concerns the team raised

**Recent team planning notes:** Load the most recent team planning / bi-weekly file if it exists within the past 14 days:
- Extract any roadmap items, capacity concerns, or risk flags raised by the team

### 7. Scan Slack for client-relevant signals

**If a `## Slack Signals` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 7b.**

Otherwise, invoke `.claude/sub/slack-signal-scan.md` with:
- **Time window:** `7d`
- **Channels:** all channels from config

If unavailable or returns no data, skip and note in the output.

From the returned signals, focus on: Client Signals, Blockers, Decisions, Brain Captures, and Unanswered Questions - these are most relevant for a PO sync.

### 7b. Pull cbc-ctv GitHub status

**If a `## GitHub Digest (cbc-ctv)` block is already present in context, use it directly and skip this step.**

Otherwise, invoke `.claude/sub/cbc-github-digest.md`.

If unavailable or returns no data, skip and note in the output.

From the returned digest, surface for the PO sync:
- Any PRs that have been approved and green for 3+ days but not merged - these are delivery blockers worth flagging
- Any PRs with failing CI tied to features the PO cares about
- Total merge-ready count as a signal of pipeline health

This is valuable for the PO because it shows whether approved work is actually shipping, or whether there's a bottleneck between "done" and "in production."

### 7c. Scan Gmail for client emails

**If a `## Gmail Digest` block is already present in context (pre-loaded by batch-runner or another skill), use it directly and skip to Step 8.**

Otherwise, invoke `.claude/sub/gmail-cbc-digest.md` with:
- **Time window:** `7d`
- **Confluence lookback:** `7d`
- **Include unread only:** `false`

If unavailable or returns no data, skip and note in the output.

From the returned digest, focus on: emails needing response, Confluence changes relevant to current work, and any scope signals from client emails.

### 8. Filter for PO sync-relevant content

A PO 1:1 is the right time to raise: **roadmap risks, client-side blockers or dependencies, important team process updates, and decisions the PO needs to make.**

From all sources, identify items that match these categories:

**Roadmap risks:** anything that threatens a delivery date, a committed feature, or a sprint target:
- Blocked tickets that impact upcoming delivery
- Stale tickets that suggest a feature is drifting
- Sprint overload or velocity concerns
- Dependencies from the client that haven't materialized

**Client-side blockers and dependencies:** things the team cannot move forward on without the client:
- Tickets blocked on client / client action
- Missing design assets, test accounts, content, or decisions
- Third-party dependencies the client controls

**Team process updates worth raising with the PO**:
- Resourcing changes (new team member, someone leaving, vacation coverage for an upcoming commitment)
- Process changes that affect the client's workflow (release notes format, QA process changes, communication changes)
- Any scope dispute or mis-alignment that needs to be resolved at the PO level

**Open action items from the client:** things the PO committed to that haven't been delivered.

Do NOT include in the PO sync prep:
- Individual developer performance or velocity details
- Internal team process debates not yet resolved
- Minor bug updates that don't affect upcoming delivery

### 9. Draft the proposed agenda

Build a proposed agenda organized by priority. Flag items where the PO needs to make a decision or provide something as `[ACTION NEEDED FROM PO]`.

Suggested structure:
1. Follow-ups from last sync - open action items from both sides
2. Delivery update - what shipped; what is in QA; upcoming delivery date
3. Roadmap risks - specific items that may affect timeline or scope
4. Client-side blockers - what the team is waiting on from the client
5. Team / process updates - anything the PO needs to know about how the engagement is running
6. PO's items - space for new asks or updates from the client side

### 10. Generate and save the notes file

Use the template below. Pre-populate the agenda and carry-forward items. Leave narrative sections blank for Vanessa to fill in during the call.

**IMPORTANT - Save location:**
Save to the PO sync PREP folder, NOT the weekly-client-sync folder.
The `weekly-client-sync/` folder is **read-only** - only real meeting notes pulled from Google Drive go there after the call. Never save prep files there.

Save prep files to:
`product-development/product/meetings/[Client]/meeting-notes/po-sync-prep/YYYY-MM-DD-[po-name]-vanessa-po-sync-prep.md`

Example: `meeting-notes/po-sync-prep/2026-06-08-remi-vanessa-po-sync-prep.md`

After saving the markdown file, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```
This generates a styled HTML version in an `html/` subfolder alongside the markdown.

Then upload the markdown file to Google Drive folder `1ficiL4i0DJTKwpb770YoE8h1RtNEkaHe` using `mcp__claude_ai_Google_Drive__create_file`. Note the returned Drive file ID for reporting in Step 11.

### 11. Report to user
Confirm:
- File path saved
- HTML file path generated
- Drive file ID (from the Google Drive upload)
- Number of open action items from previous sync (Vanessa's + client's)
- Number of roadmap risks identified
- Number of client-side blockers
- Any Slack signals surfaced
- Any sub-routines that were skipped due to unavailability

---

## Output Template

```markdown
# [PO Name] / Vanessa - [Project] Weekly Priority Sync

**Date:** [Month DD, YYYY]
**Attendees:** Vanessa Hughes, [PO Name] ([Client])
**Type:** Weekly Client Sync

---

## Proposed Agenda

1. Follow-ups from [previous sync date]
   [List open action items from both sides]

2. Delivery update
   [What shipped / what's in QA / next delivery date]

3. Roadmap risks
   [Specific items that may affect timeline or scope]

4. Blockers requiring [Client] input
   [Things the team can't move forward without the client]

5. Team / process updates
   [Resourcing changes, process updates, anything the PO needs to know]

6. [PO Name]'s items
   [Leave blank - PO adds on call]

---

## Summary

[Leave blank - fill in after call]

---

## Decisions

[Leave blank - fill in during call]

---

## Next Steps

| Owner | Action |
|-------|--------|
| | |

---

## Details

[Leave blank - expand key topics during/after call]

---

## Prep Notes *(remove before sharing)*
> Generated [timestamp]
>
> **Open action items from [previous sync date]:**
> - Vanessa: [list or "none"]
> - [PO Name]: [list or "none"]
>
> **Roadmap risks identified:**
> [list]
>
> **Client-side blockers:**
> [list - ticket key + what the team needs]
>
> **Stale in-progress tickets (10+ days no update):**
> [list or "none"]
>
> **Slack signals this week:**
> [summary or "none"]
>
> **Email signals this week:**
> [summary or "none"]
>
> **Confluence changes this week:**
> [summary or "none"]
>
> **Overdue commitments:**
> [list or "none"]
```

---

## Rules

- Agenda items must come from real data - Jira, Slack, or meeting notes. Do not invent items.
- Every item where the PO needs to act is flagged `[ACTION NEEDED FROM PO]`
- Open client action items from the previous sync always appear first in the agenda
- **NEVER surface an item as open, overdue, or pending if it appears in the weekly tracker's Completed Items section (✅ lines).** The tracker is the source of truth - it overrides meeting notes. If it is marked done in the tracker, it is done. Full stop.
- **NEVER surface an active tracker item as "overdue/unconfirmed" if its status says IN PROGRESS or "working with [person]".** That means it is in flight, not missed.
- If Vanessa made a commitment with a specific date and it is overdue AND it is NOT in the resolved list, surface it prominently
- Internal team concerns (velocity, individual performance) are never included in this prep
- Prep Notes are for Vanessa's reference only - remind her to remove before sharing

---

> **Skill verification:** Please ensure that the skill po-sync-prep SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
