---
name: release-day-runbook
description: Use this skill to create a timed release day runbook that unifies pre-release, release, and post-release steps. Triggers on "release day", "release runbook", "launch day checklist", "release day plan", "deploy checklist", or any request to create a structured runbook for a release window.
---

# Skill: Release Day Runbook

Unifies release-prep + cert-checklist into a timed runbook covering pre-release, release window, and post-release steps. Produces a checklist document ready to follow on release day.

---

## Before You Start

Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, platform list, and folder paths.

---

## Process

### 1. Identify the release

The user must provide:
- **Version number:** e.g., `v1.20.0`
- **Release date:** specific date or "today"
- **Platforms in scope:** which platforms are shipping (or "all")

If any are missing, ask.

### 2. Pull release data

**If a `## Jira Snapshot` block is already present in context, use it.**

Otherwise, invoke `.claude/sub/jira-project-snapshot.md` to get current sprint status, blocked issues, and completed work. If the snapshot is unavailable or returns no results, note "Jira data unavailable" in the Open Items section and proceed with manual inputs from the user.

Also check for existing release-prep or cert-checklist outputs:
- Look in `product-development/product/customers/accounts/[client]/sprints/` for `[version]-release-prep.md`
- If found, load it and use its data (what's shipping, open items, cert readiness)
- If not found, run the relevant queries inline

### 3. Build the runbook

Generate a timed checklist organized into 3 phases:

```markdown
# Release Day Runbook - [Version] - [Date]

**Version:** [version]
**Release Date:** [date]
**Platforms:** [list]
**Release Manager:** [from project foundation or ask]
**Status:** [Pre-Release / In Progress / Complete]

---

## Pre-Release Checklist (Before Release Window)

### Cert Confirmations
- [ ] Samsung Tizen - cert status: [status]
- [ ] LG webOS - cert status: [status]
- [ ] Xbox - cert status: [status]
- [ ] Comcast X1 - cert status: [status]
- [ ] Xumo - cert status: [status]
[Only include platforms in scope]

### QA Sign-Off
- [ ] Regression testing complete - signed off by: ___
- [ ] Platform-specific testing complete for: [platforms]
- [ ] No P0/P1 bugs open against this release
- [ ] Test results documented in: [link or "TBD"]

### Release Notes
- [ ] Release notes drafted: [path or "TBD"]
- [ ] Release notes reviewed by PM
- [ ] Client-facing release notes sent to: [contact or "TBD"]

### Code & Build
- [ ] Release branch created and tagged
- [ ] Build artifacts generated for all platforms
- [ ] Build verified in staging/pre-prod

### Stakeholder Communication
- [ ] Internal team notified of release window
- [ ] Client notified of release window and expected downtime (if any)
- [ ] Support team briefed on changes

---

## Release Window Steps

### Deploy Sequence
[Order based on platform dependencies and cert schedules]

- [ ] **[Platform 1]** - Deploy build [version] -> verify -> confirm
  - Verify: [specific check - e.g., app launches, content loads, DRM playback]
  - Rollback plan: [specific rollback steps]

- [ ] **[Platform 2]** - Deploy build [version] -> verify -> confirm
  - Verify: [specific check]
  - Rollback plan: [specific rollback steps]

[Continue for each platform]

### Go / No-Go Decision Points
- [ ] After [Platform 1] deploy: Go / No-Go for remaining platforms
- [ ] If any platform fails: escalation path -> [contact]

---

## Post-Release Steps

### Monitoring (First 24 Hours)
- [ ] Error rate monitoring - baseline: [current rate or "TBD"]
- [ ] Crash rate monitoring per platform
- [ ] Key user flows verified in production
- [ ] Client-reported issues triaged

### Documentation Updates
- [ ] Roadmap updated - version marked as released
- [ ] Release notes finalized and published
- [ ] Known issues documented

### Client Notification
- [ ] Client notified that release is live
- [ ] Post-release summary sent (what shipped, known issues, next steps)

### Retrospective Inputs
- [ ] Any release-day issues logged for retro
- [ ] Deployment timing and sequence notes captured

---

## Open Items - Must Resolve Before Release

| Item | Status | Owner | Blocker? |
|------|--------|-------|----------|
[From Jira snapshot - blocked issues and open items in the release]

---

## Contacts

| Role | Name | Reach via |
|------|------|-----------|
| Release Manager | [name] | [Slack/phone] |
| QA Lead | [name] | [Slack/phone] |
| Client Contact | [name] | [Slack/email] |
| Escalation | [name] | [Slack/phone] |
```

### 4. Save the runbook

Save to: `product-development/product/customers/accounts/[client]/sprints/[version]-release-runbook.md`

After saving, immediately run:
```bash
python3 scripts/md-to-html.py <saved-file-path>
```

### 5. Report to user

Confirm:
- File path saved
- HTML path
- Number of pre-release items (how many checked vs. unchecked)
- Number of open blockers
- Platforms covered

---

## Rules

- **Checklist format is mandatory.** Every item must be a `- [ ]` checkbox. The PM checks these off during release day.
- **Platform-specific steps.** Each platform gets its own deploy + verify step. Do not combine platforms.
- **Rollback plans are required.** Every deploy step must have a rollback plan, even if it's "revert to previous version."
- **Open items are blockers until resolved.** If the Jira snapshot shows blocked issues against this release, they appear in the Open Items section prominently.
- **No Jira writes.** Do not update ticket statuses or release versions.
- **No sends.** Save to file only. Do not post to Slack or send emails.
- **Populate from real data.** Use release-prep output, cert-checklist results, and Jira data. Do not invent cert statuses or QA results.

---

> **Skill verification:** Please ensure that the skill release-day-runbook SKILL.md was actually run. If the skill was not run, it needs to be run again from the top.
