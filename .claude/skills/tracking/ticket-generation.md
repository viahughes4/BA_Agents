---
name: ticket-generation
description: Use this skill to generate Jira-formatted tickets for standing team tasks, ad-hoc improvements, and operational work items that are not user stories. Trigger when the user asks to "create a ticket", "write up a task", "create an epic", "create a sub-task", "create an improvement ticket", "write a ticket", "new ticket", "make a ticket", "log a ticket", "file a ticket", "create a bug", "write up a bug", or any request to document work that does not follow a user story format. Examples: "create a ticket for the loading state fix", "write up an improvement ticket for nav bar", "I need an epic for RDI", "create a sub-task under <jira.project_key>-1234".
---

# Skill: Ticket Generation

Generate Jira-formatted tickets for standing team tasks, ad-hoc improvements, epics, and sub-tasks. These are not user stories - use `/generate-stories` for that.

---

## MANDATORY - Jira Description Rule (read before doing anything else)

**The client Jira project has a default description template that overwrites any description passed during ticket creation.** This means `createJiraIssue` will NEVER save the description, regardless of format or content.

The only working pattern is a mandatory two-step process:

1. Call `createJiraIssue` to create the ticket (description will be blank/template - this is expected)
2. Immediately call `editJiraIssue` with `contentFormat: "adf"` and the full description as an ADF document object

**This applies to EVERY ticket, EVERY time.** There are no exceptions. Skipping step 2 results in a blank description in Jira.

Confirmed via testing Jul 7 2026. Root cause: Jira project-level template config on the client instance.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load and validate project context.

If context-loader returns an error or incomplete context, note the gap to the user and proceed with available information. Do not halt the skill silently.

### Optional: Slack context
If the user has provided a Slack message, thread, or channel link alongside their input, read `.claude/sub/slack-context-extractor.md` and follow its process in full. Merge the returned context block into the working context before proceeding. If no Slack input is provided, skip this step.

### 2. Ask ticket type
If the user has not specified the ticket type, ask:

> **What type of ticket is this?**
> - **Task** - a defined piece of work with clear output (bug fix, feature work, integration task)
> - **Sub-task** - a smaller unit of work under a parent ticket
> - **Improvement** - an enhancement to something that already exists (UX, performance, reliability)
> - **Epic** - a high-level container grouping related work across multiple tickets

Wait for the answer before proceeding.

### 3. Clarification gate
Before generating, check for blockers by ticket type:

- **All types:** If the description of work or affected area is unclear, ask at most 2 targeted questions.
- **Sub-task:** If no Parent key (e.g., `<jira.project_key>-1234`) has been provided, ask for it before proceeding. Parent is required - do not generate a sub-task without it.
- **Improvement:** If the current vs. desired state is unclear, ask one clarifying question to establish it.
- **Epic:** If Goals or Scope are completely undefined, ask the user to describe the business objective before generating.

Do not generate until the minimum information is in hand.

### 4. Generate the ticket
Use the template for the selected type below. Omit any section that genuinely does not apply - do not leave placeholder text or "N/A" in the output.

### 5. Self-validate
Before presenting to the user, check:
- Description clearly states what the work is and why
- Every AC statement describes a specific, observable behavior or appearance - no vague language
- Sub-task has a Parent key populated - never leave this blank
- Improvement includes User Impact and either a Measurement section or a note that it was omitted intentionally
- Technical Notes are only included if there is something meaningful to capture
- Design Details only included if a Figma link is available or design review is relevant
- Epic has Goals, Scope, and Definition of Done defined - not granular AC

### 6. Confirm before creating
Present the full drafted ticket to the user. Then ask:

> "Does this look right? I'll create it in Jira once you confirm."

**Do not call the Jira create tool until the user explicitly approves.** Wait for a clear "yes", "create it", "looks good", or equivalent before proceeding.

If the user requests changes, revise the draft and re-present. Repeat until approved.

### 7. Create in Jira
Once the user confirms, create the ticket using the Jira create tool with the correct `projectKey`, `issueTypeName`, `parent` (if sub-task), `summary`, `description`, and any `additional_fields` (assignee, priority).

**CRITICAL - Description format:** Always pass the description using `contentFormat: "adf"` with a structured ADF document. Never use `contentFormat: "markdown"` - it silently fails to save in Jira. The minimum ADF structure is:

```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {"type": "paragraph", "content": [{"type": "text", "text": "Your description text here"}]}
  ]
}
```

For multi-section descriptions, use heading nodes (`{"type": "heading", "attrs": {"level": 3}, "content": [...]}`) and additional paragraph nodes. For code blocks use `{"type": "codeBlock", "attrs": {"language": "text"}, "content": [...]}`.

**CRITICAL - ADF patch step (always required):** After every ticket creation, immediately do a second call to update the description, unconditionally. Do not check first - just always patch. Jira silently drops or overrides ADF descriptions on creation; the PUT/editJiraIssue step is the only reliable way to persist them. Skipping this step will result in a blank or template description every time.

**If using Atlassian MCP plugin tools:** After `createJiraIssue`, immediately call `editJiraIssue` with the same ADF description.

**If using direct REST API (Python):** After the POST to create the ticket, immediately do a PUT to `/rest/api/3/issue/{key}` with `{"fields": {"description": <adf_object>}}`. Then verify with a GET to confirm the description has content blocks. If the description is empty or shows a Jira template, re-patch it.

```python
# Always patch description after create
patch_payload = json.dumps({'fields': {'description': description_adf}}).encode()
req = urllib.request.Request(
    f'https://api.atlassian.com/ex/jira/{cloud_id}/rest/api/3/issue/{new_key}',
    data=patch_payload, headers=headers, method='PUT'
)
urllib.request.urlopen(req, context=ctx)

# Always verify
verify_req = urllib.request.Request(
    f'https://api.atlassian.com/ex/jira/{cloud_id}/rest/api/3/issue/{new_key}?fields=description',
    headers=headers
)
verify_resp = urllib.request.urlopen(verify_req, context=ctx)
verify_data = json.loads(verify_resp.read())
content_blocks = verify_data['fields'].get('description', {}).get('content', [])
if not content_blocks:
    raise Exception(f'Description patch failed on {new_key} - re-patch required')
```

**Always add the label `created-via-claude` to every ticket created.** This is a mandatory org requirement - no exceptions.

Return the created ticket key and URL.

### 8. Report to user in chat
After the ticket is created, report the ticket key and URL in chat. Do not send a Slack DM.

---

## Templates

### Task

```
## [Ticket Title]
> [JIRA-KEY]-DRAFT | Type: Task | Epic: [epic name, or TBD]

**Description**
[What is this work and why does it matter? 2-4 sentences. State the area, the current gap, and what needs to be done.]

**Platform:** [list]
**Priority:** [High / Medium / Low]
**Story Points:** [estimate] *(omit if not yet estimated - leave for sprint planning)*
**Sprint:** [target sprint, or "Backlog"] *(omit if not yet assigned)*
**Dependencies:** [PROJ-XXX] *(omit if none)*

**Acceptance Criteria**
- [The component should behave like X]
- [The screen should display X when the user does Y]
- [The state should update to Z when action A occurs]
- [Loading state: the skeleton/spinner should appear while X is loading]
- [Error state: the screen should display [message] if X fails]
*(Max 8-10 statements. If more are needed, suggest splitting into two tickets.)*

**Technical Notes** *(omit if not applicable)*
- [Implementation context, constraints, or considerations]

**Design Details** *(omit if not applicable)*
- Figma: [link, or "pending - to be added before dev pickup"]

**Open** *(omit if none)*
- [one-liner - defaulting to [X] unless changed]
```

---

### Sub-task

```
## [Sub-task Title]
> [JIRA-KEY]-DRAFT | Type: Sub-task | Parent: [PARENT-KEY - required]

**Description**
[What specific piece of work is this? 1-3 sentences. Reference the parent ticket context briefly.]

**Platform:** [list]
**Priority:** [High / Medium / Low]
**Story Points:** [estimate] *(omit if not yet estimated)*
**Assignee:** [name, if known] *(omit if not yet assigned)*

**Acceptance Criteria**
- [Specific, observable outcome for this sub-task only]
- [Scoped tightly - this should not duplicate parent ticket AC]
*(Max 5-6 statements for a sub-task.)*

**Technical Notes** *(omit if not applicable)*
- [Any implementation notes specific to this sub-task]

**Open** *(omit if none)*
- [one-liner - defaulting to [X] unless changed]
```

---

### Improvement

```
## [Improvement Title]
> [JIRA-KEY]-DRAFT | Type: Improvement | Epic: [epic name, or TBD]

**Description**
[What exists today and what is the problem with it? Then describe the desired improvement. 2-4 sentences.]

**Current state:** [What the experience or system looks like now]
**Desired state:** [What it should look like after this improvement]
**User impact:** [Who is affected, how frequently, and how severe - e.g., "All Samsung users on navigate-back from player; causes visible layout jank every session"]

**Platform:** [list]
**Priority:** [High / Medium / Low]
**Story Points:** [estimate] *(omit if not yet estimated)*
**Dependencies:** [PROJ-XXX] *(omit if none)*

**Acceptance Criteria**
- [The improved component should behave like X]
- [The updated state should display as Y]
- [The user should no longer experience Z]
*(Max 8-10 statements. If more are needed, suggest splitting.)*

**Measurement** *(omit if not applicable)*
- [How will we verify this improvement delivered value post-ship? Name the signal: error rate delta, load time reduction, support ticket volume, manual QA pass rate, etc.]
- [AC answers "does it work?" - Measurement answers "did it matter?"]

**Technical Notes** *(omit if not applicable)*
- [Any refactor notes, constraints, or approach considerations]

**Design Details** *(omit if not applicable)*
- Figma: [link, or "pending - to be added before dev pickup"]

**Open** *(omit if none)*
- [one-liner - defaulting to [X] unless changed]
```

---

### Epic

```
## [Epic Title]
> [JIRA-KEY]-DRAFT | Type: Epic

**Description**
[What is this epic delivering and why? 2-4 sentences covering the business goal and user value.]

**Goals**
- [Outcome 1: what success looks like for the business or user]
- [Outcome 2]

**Scope**

*In scope:*
- [Feature / workstream]

*Out of scope:*
- [What this epic explicitly does not cover - be specific to prevent scope creep]

**Platform:** [list]
**Priority:** [High / Medium / Low]
**Target quarter / release:** [Q3 2026 / v1.21.0 / etc.]

**Stakeholders**
- Internal: [name, role - e.g., Roberto Chavez, Eng Lead]
- Client: [name, role - e.g., Remi Beaupre, client PO]

**Discovery / PRD:** [link to feature brief, PRD, or Confluence page - or "pending"]

**Key Deliverables**
- [ ] [Ticket or workstream - link when available]
- [ ] [Ticket or workstream]

**Definition of Done**
- [Binary, unambiguous statement of when this epic is closeable in Jira]
- Example: "All child tickets resolved, release shipped to all active platforms, client sign-off received"

**Success Metrics**
- [How will we know this epic delivered business value? Distinct from DoD - this is about outcomes, not closure criteria]
- Example: "RDI live content loads within 3s on Samsung INT5 with no playback errors in smoke test"

**Dependencies**
- [Upstream: what must exist or be completed before this epic can close]

**Risks**
- [Known risk or uncertainty - include mitigation if one exists]

**Open** *(omit if none)*
- [one-liner - defaulting to [X] unless changed]
```

---

## AC Writing Rules

Applies to Task, Sub-task, and Improvement tickets. Not used for Epics.

- Write in present tense: "The button should..." / "The screen should..." / "The label should..."
- Describe behavior and appearance: what the user sees or what the system does
- Be specific: name the component, state, or condition
- Cover edge states: include at least one AC statement for loading, error, and empty states where applicable
- Do not use Gherkin (Given / When / Then) format
- Do not write AC for implementation details: describe what the user sees, not how the code works
- Banned phrases: "works correctly", "handles gracefully", "as expected", "appropriately", "looks good", "functions properly"
- Each statement should be independently verifiable by QA without knowledge of the code
- Sub-task AC: max 5-6 statements, scoped to that unit of work only - do not restate parent ticket AC
- Task / Improvement AC: max 8-10 statements - if more are needed, suggest splitting into two tickets

## Template Quick Reference

| Field | Task | Sub-task | Improvement | Epic |
|-------|------|----------|-------------|------|
| Description | yes | yes | yes | yes |
| Current / Desired State | no | no | yes | no |
| User Impact | no | no | yes | no |
| Platform / Priority | yes | yes | yes | yes |
| Story Points | optional | optional | optional | no |
| Sprint | optional | no | no | no |
| Assignee | no | optional | no | no |
| Parent Key | no | **required** | no | no |
| Target Quarter / Release | no | no | no | yes |
| Acceptance Criteria | yes (8-10) | yes (5-6) | yes (8-10) | no |
| Measurement | no | no | optional | no |
| Technical Notes | optional | optional | optional | no |
| Design Details | optional | no | optional | no |
| Stakeholders | no | no | no | yes |
| Discovery / PRD | no | no | no | yes |
| Goals + Scope | no | no | no | yes |
| Key Deliverables | no | no | no | yes |
| Definition of Done | no | no | no | yes |
| Success Metrics | no | no | no | yes |
| Dependencies / Risks | optional | no | optional | yes (split) |
| Open Questions | optional | optional | optional | optional |

---

## Rules

- Never create or update a Jira ticket without showing a full draft first and getting explicit approval. No exceptions.
- Always add the label `created-via-claude` to every ticket created in Jira. This is a mandatory org requirement.
- Never generate a sub-task without a Parent key - it is required before generation begins.
- Banned AC phrases: "works correctly", "handles gracefully", "as expected", "appropriately", "looks good", "functions properly". Rewrite any AC that uses these.
- Do not use em dashes in any output: use a hyphen (-) or colon (:) instead.
- If the user says "create it" without seeing a draft, show the draft first. Pasting content is not approval.
- After ticket creation, report the ticket key and URL in chat only. Do not send a Slack DM.
- Do not fabricate Jira keys. Use the project key returned by `.claude/sub/context-loader.md` with the `-DRAFT` suffix for drafts.
- If the ticket type is unclear after two clarifying questions, default to Task and note the assumption at the top of the draft.
- **CMP/Didomi reminder (surface to PM, do not add to ticket):** Before finalizing any ticket, check: does this feature touch navigation, modals, player controls, consent flows, or any screen that overlays the app? If yes, remind Vanessa: "This may interact with CMP/Didomi - confirm CMP navigation regression is in QA scope for this ticket." This is a PM prompt only, not ticket content.

---

> **Skill verification:** Please ensure that the skill ticket-generation SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
