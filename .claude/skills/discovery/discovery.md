---
name: discovery
description: This skill should be used when the user asks to "run discovery", "start discovery", "analyze SOW", "kick off discovery", "help me with discovery", "new project", "I have a new SOW", "project intake", "scope this", "scope analysis", "project scoping", "full discovery", or needs comprehensive discovery analysis for an OTT/streaming project. This skill handles full 5-phase project-level discovery (SOW Intake, PRD Creation, Epic Definition, Cross-Functional Reviews, Project Planning). For single-feature scoping, use the feature-discovery skill instead. Use this skill to guide structured 5-phase discovery: SOW Intake -> PRD Creation -> Epic Definition -> Cross-Functional Reviews -> Project Planning.
version: 2.0.0
---

# Discovery Skill for OTT Projects

This skill guides comprehensive discovery workflows for OTT/streaming projects, taking a PM from raw pre-kickoff materials (SOW, scope docs, RFPs) through to execution-ready deliverables.

## When to Use This Skill

Use discovery skill for:
- **Project kickoff**: New client engagement, new feature initiative, complex scope requiring systematic analysis
- **SOW analysis**: Extract requirements from contracts, statements of work, or RFPs
- **Multi-platform scoping**: Projects affecting multiple platforms (iOS, Android, CTV, Web, etc.)
- **Stakeholder alignment**: Ensure cross-functional agreement (Design, QA, Dev, PM, Risk) before development
- **Timeline estimation**: Build realistic project plans with capacity constraints

## 5-Phase Discovery Process

### Phase 1: SOW Intake & Scope Analysis (1-2 days)
**Goal:** Extract complete project scope from raw materials.

**Actions:**
0. **Load project foundation:** Invoke `.claude/sub/context-loader.md` to load the client's Jira project foundation (team, contacts, Jira config, platform scope) into context before proceeding.
1. Receive SOW, RFP, scope doc, or project brief
1.5. **Auto-Pull Context:** Before asking clarifying questions, automatically search for existing context:
   - **Google Drive:** Use `mcp__claude_ai_Google_Drive__search_files` to search the Drive folder (see `.claude/config.yml` for the Drive folder ID) for files related to the project/SOW keywords. Download and review any matches using `mcp__claude_ai_Google_Drive__read_file_content`. If no results are returned, skip and note in the output.
   - **Slack channels:** Invoke `.claude/sub/slack-signal-scan.md` with `time_window=14d` and extract relevant project/SOW signals from the returned results. If the channels are unreachable or return no results, skip and note in the output.
   - **Meeting notes:** Scan `product-development/product/meetings/[Client]/meeting-notes/` files from the last 14 days (standup/, weekly-client-sync/, team-bi-weekly/) for SOW/project keyword matches. If the folder is empty or no matches are found, skip and note in the output.
   - Present a summary of what was found and merge into the intake context before proceeding to clarifying questions.
2. Ask clarifying questions on 6 critical areas (referencing `references/intake-questionnaire.md` for structured question sets):
   - **Platform scope**: Which platforms? Which OS versions?
   - **Content catalog**: How many titles? Which content formats? Licensing constraints?
   - **Playback stack**: Which player SDK? DRM provider? CDN? Manifest format?
   - **Auth & entitlements**: Which auth system? Entitlement logic? Device limits?
   - **Analytics & monetization**: Which KPIs? Ads? In-app purchase? Subscription?
   - **Team & timeline**: Team size? Hard deadlines? Dependencies?
3. Flag **all platform-specific blockers** (DRM fragmentation, D-pad navigation, certification cycles, memory constraints)
4. Produce: **Intake Summary** (1-2 pages) with scope, constraints, open questions, platform risk register
   - Save to: `product-development/product/customers/accounts/[client]/` as `YYYY-MM-DD-intake-summary-[project].md`
   - Run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/YYYY-MM-DD-intake-summary-[project].md`
   - Report the HTML path to the user.

**Output:**
- Intake Summary document
- Platform Risk Register (high-level)
- Decisions Ready for Review Gate
- **Risk Register Seed** - risks identified during intake, formatted in risk-register entry format for direct import via `/risk-register`:
  > **Risk**: [title] | **Type**: Timeline / Technical / External / Resource | **I**: 1-3 | **L**: 1-3 | **Score**: I×L | **Owner**: [name or TBD] | **Mitigation**: [Prevention] / [Contingency] / [Trigger to escalate]

---

### Phase 2: PRD Creation + Review Gate (2-3 days)
**Goal:** Convert scope into a structured PRD with full traceability and cross-functional sign-off.

**Actions:**
1. Draft PRD sections (5-10 pages):
   - **Overview** (client, vision, success metrics)
   - **Platform & Device Scope** (explicit OS version ranges, DRM per platform, D-pad navigation flow)
   - **Feature Backlog** (100+ features mapped to catalog with effort estimates)
   - **Content & Playback** (catalog size, formats, DRM, CDN, player SDK specifics)
   - **Auth & Entitlements** (flow, timing, failure handling, device limits)
   - **Analytics** (KPI definitions, event taxonomy, dashboards)
   - **Monetization** (ads, IAP, subscription logic)
   - **Localization** (language support, metadata translation, regional formatting)
   - **Certification & Compliance** (platform requirements, timeline, legal holds)
   - **Technical Constraints** (memory, performance targets, SLAs, dependencies)

2. Trace every PRD section to SOW clause or intake answer (cite sources)

3. **Parallel 4-Reviewer Gate:**
   - [ ] **Design Review**: Platform UX parity, 10-foot UI compliance (text size, focus rings, safe zones), D-pad navigation flow
   - [ ] **QA Review**: Test scope, platform fragmentation risks, edge cases per platform
   - [ ] **Dev Review**: Technical feasibility, dependencies, player SDK availability, DRM complexity
   - [ ] **Risk Review**: Platform constraints, certification blockers, timeline risks, capacity constraints

   If a reviewer is unavailable, flag the gate as pending and proceed with a documented assumption.

4. Iterate based on reviewer feedback. Surface **all blockers as Open**.

5. Save PRD to: `product-development/product/PRDs/[client]/YYYY-MM-DD-prd-[project].md`
   - Run: `python3 scripts/md-to-html.py product-development/product/PRDs/[client]/YYYY-MM-DD-prd-[project].md`
   - Report the HTML path to the user.

**Output:**
- Complete PRD (5-10 pages, fully traced)
- Reviewer Sign-off Checklist (all 4 gates green)
- Consolidated Decision Log (all blockers resolved or escalated)

---

### Phase 3: Epic Definition + Sizing (2-3 days)
**Goal:** Break PRD into deliverable epics with story-point sizing and platform-specific acceptance criteria.

**Actions:**
1. Extract 15-30 epics from PRD (1 epic = 1-3 sprint deliverable)
   - Map each epic to PRD section(s)
   - Define epic-level acceptance criteria (testable, platform-specific)
   - Size each epic (use OTT benchmarks: see references/sizing-benchmarks.md)

2. **Epic sizing benchmarks** (per platform):
   - iOS/Android: 3-8 points (touch, standard SDK)
   - CTV (Tizen, webOS, Fire TV, Roku): 5-13 points (D-pad nav, memory constraints, custom playback)
   - Web: 2-5 points (faster iteration, more browser support)
   - Multi-platform (3+ platforms): Add +30% effort per platform after first

3. Identify **cross-platform dependencies** (e.g., "API must support X before mobile can ship")

4. Build **Phased Delivery Plan**:
   - Phase 1 (MVP): 3-5 critical epics, ~20-30 points
   - Phase 2: Next tranche of features
   - Phase 3+: Nice-to-haves, optimization

5. Create **Epic Tracking Sheet** (Google Sheet or Jira):
   - Epic name, PRD source, description, AC, size (story points), platform-specific notes, dependencies
   - **If creating epics in Jira:** Present the full draft in chat (title, description, acceptance criteria, story points) for each epic and wait for explicit user approval before writing to Jira. Project: CBC, Board: 1824 at accedobroadband.jira.com.

6. Save Epic Backlog to: `product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-epic-backlog-[project].md`
   - Run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-epic-backlog-[project].md`
   - Report the HTML path to the user.

**Output:**
- Epic Backlog (15-30 epics, fully sized)
- Phased Delivery Plan (phases with point allocations)
- Epic Tracking Sheet (shared with team)
- **Downstream Ready Flags** - for each epic, include:
  - `/generate-stories` readiness: Ready / Needs Clarification / Blocked - with recommended story tier (Minimal/Standard/Heavy)
  - Platform-split recommendation: which platforms need separate stories vs. shared implementation
  - Integration contracts identified: named API contracts, SDK versions, service dependencies per epic
  - Risks in `/risk-register` format for direct import:
    > **Risk**: [title] | **Type**: Timeline / Technical / External / Resource | **I**: 1-3 | **L**: 1-3 | **Score**: I×L | **Owner**: [name or TBD] | **Mitigation**: [Prevention] / [Contingency] / [Trigger to escalate]

---

### Phase 4: Epic Review Gate + Final Adjustments (1-2 days)
**Goal:** Validate epic sizing, acceptance criteria, and phased delivery against team capacity.

**Actions:**
1. **Parallel 4-Reviewer Gate (same reviewers as Phase 2):**
   - [ ] **Design Review**: Each epic's AC includes UI mockup, platform-specific layouts (D-pad focus order, safe zones), state designs (loading, error, empty)
   - [ ] **QA Review**: Each epic's AC is testable on all platforms, edge cases identified, test scope clear
   - [ ] **Dev Review**: Sizing is realistic, blockers are clear, dependencies are documented
   - [ ] **Risk Review**: Platform risks per epic, certification blockers, timeline risks flagged

   If a reviewer is unavailable, flag the gate as pending and proceed with a documented assumption.

2. Adjust sizing based on feedback.

3. Confirm phased delivery plan against team capacity:
   - **Available capacity**: team size x sprint velocity (points per sprint)
   - **Timeline**: phase points / capacity = estimated weeks
   - **Dependencies**: external APIs, design assets, certification cycles must be on critical path

4. Flag **all timeline risks** using Impact x Likelihood (1-3 scale):
   - **High Risk**: Impact 3 x Likelihood 2+ (must address before starting)
   - **Medium Risk**: Impact 2 x Likelihood 1-2 (monitor weekly)

**Output:**
- Final Epic Backlog (fully reviewed, signed off)
- Epic Tracking Sheet (updated with reviewer feedback)
- Capacity Plan (team capacity vs. phased delivery)
- Timeline Risk Register (all risks scored, mitigations identified)

---

### Phase 5: Project Plan Creation (1 day)
**Goal:** Produce execution-ready project plan with sprints, capacity allocation, and risk mitigation.

**Actions:**
1. **Create Sprint Plan**:
   - Sprint 1-N: Assign epics to sprints based on capacity and dependencies
   - Each sprint: ~20-30 points (adjust per team velocity)
   - Flag dependency epics (epics that depend on external APIs or partner work)

2. **Create Project Timeline**:
   - Kickoff -> MVP delivery -> Phase 2 -> Phase 3
   - Add 20% buffer for testing, certification, and dependencies
   - Mark hard deadlines (client launch dates, platform cert cycles, campaign dates)

3. **Identify Critical Path**:
   - Longest sequence of dependent epics
   - Certification cycles (add 5-15 days depending on platform)
   - External dependencies (partner APIs, design assets, content ingestion)

4. **Risk Mitigation Plan**:
   - For each HIGH-risk epic, document mitigation (spike, architectural review, early design, POC)
   - Assign owner for each mitigation

5. **Stakeholder Readiness Checklist**:
   - [ ] Design assets on track (kickoff date, delivery date)
   - [ ] Backend APIs on track (contract signed, dev env ready)
   - [ ] QA environment ready (staging, test data)
   - [ ] Certification prep started (guidelines reviewed, test devices obtained)
   - [ ] Team capacity confirmed (dev, QA, PM, design alloc)

6. **Produce and save final artifacts:**
   - Project Charter (1-2 pages: client, vision, scope, constraints, timeline, success metrics)
     - Save to: `product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-project-charter-[project].md`
     - Run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-project-charter-[project].md`
   - Sprint Plan (sprints 1-N with epic assignments)
     - Save to: `product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-sprint-plan-[project].md`
     - Run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/sprints/YYYY-MM-DD-sprint-plan-[project].md`
   - Timeline Gantt (visual critical path, dependencies)
   - Risk Register (all risks scored, mitigations, owner)
     - Save to: `product-development/product/customers/accounts/[client]/risk-register/YYYY-MM-DD-risk-register-[project].md`
     - Run: `python3 scripts/md-to-html.py product-development/product/customers/accounts/[client]/risk-register/YYYY-MM-DD-risk-register-[project].md`
   - Readiness Checklist (stakeholder sign-off)
   - Report all HTML paths to the user.

**Output:**
- Project Charter
- Sprint Plan
- Timeline Gantt
- Risk Register (final)
- Stakeholder Readiness Checklist

---

## Key OTT Platform Edge Cases (Always Check)

These patterns apply to almost every OTT project. Do NOT wait for the PM to mention them - assume they apply and raise issues proactively.

### DRM Fragmentation
- **Not universal**: FairPlay (iOS), Widevine (Android/Web/Tizen/webOS), PlayReady (Xbox/MVPD), Roku native
- **License server routing**: If multi-platform, backend must route to correct DRM provider per device
- **Impact**: Backend complexity, player SDK selection, test device coordination

### Platform-Specific Input Models
- **Touch**: iOS, Android mobile - swipes, pinches, long-press
- **D-pad + Remote**: All CTV platforms - manual focus management, Back button behavior, navigation sequences must be defined
- **Keyboard + Pointer**: Web browsers
- **Gamepad**: Xbox, PlayStation

**Impact**: Navigation patterns designed for touch will NOT work on TV remotes without rework. Flag upfront.

### Memory & Performance Constraints
- **High-end**: Nvidia Shield (few constraints)
- **Mid-tier**: Modern Samsung Tizen, LG webOS, Xbox Series X/S, PS5
- **Low-end**: Roku sticks, older Samsung/LG, Fire TV Stick 1st gen - texture limits, GC pauses, memory pressure

**Impact**: Asset optimization, player streaming quality limits, loading state timings.

### Certification Timelines
- **Fast**: Apple iOS/tvOS (2-5 business days), Google Play (1-3 days)
- **Slow**: Samsung Tizen, LG webOS (5-15 business days)
- **Critical**: Certification cycles are on critical path. Add 4 weeks before launch for submission + review + rework + resubmission.

### Bilingual & Localization
- **Assume required for multi-region projects** (Canada = EN/FR, others = region-specific)
- **String extraction**: Text-heavy features need early l10n scoping
- **Metadata translation**: Content titles, descriptions, metadata sources
- **Layout testing**: Text expansion, RTL languages

### Content Delivery Pipeline
- **Manifest formats**: HLS (m3u8) is near-universal; DASH (mpd) for some Android/CTV deployments. Confirm which format(s) the backend delivers.
- **ABR strategy**: Client-driven (player SDK selects quality) vs. server-driven (manifest controls). Affects player SDK integration complexity.
- **CDN warm-up**: First launch in a new region may require CDN origin-pull; impacts TTF (time to first frame). Plan for geographic rollout.
- **Geo-fencing**: Content rights enforcement via IP-based geolocation. Must be enforced server-side, not client-side. Flag any feature that bypasses or interacts with geo-restriction logic.
- **Content ingestion for new content types**: Adding a new content type (clips, extras, behind-the-scenes) requires CMS schema updates, metadata mapping, and potentially new API endpoints. Flag as a backend dependency.

### Feature Flag & Progressive Rollout
- **Flag granularity**: Per-platform, per-user, per-region, per-brand (Gem vs. TouTV). The flag infrastructure must support the granularity the rollout needs.
- **Rollback strategy**: Feature flags must support instant rollback without an app update. If a feature is behind a flag, the off-state must be tested as thoroughly as the on-state.
- **A/B testing**: If the feature needs A/B testing, the flag infrastructure must support variant assignment, event tracking per variant, and statistical significance calculation.
- **Accedo Control specifics**: For CBC/Accedo projects, Accedo Control is the typical flag infrastructure. Confirm it supports the required flag granularity before committing to a rollout plan.

### Regression Risk Patterns
- **Shared navigation (LRUD)**: The custom LRUD singleton manages focus across all screens. Any new screen or component that registers/deregisters focus nodes can regress navigation on existing screens. Flag if the feature adds new focusable elements.
- **Player lifecycle states**: The player has complex lifecycle states (loading, buffering, playing, paused, seeking, error, ended, ad break). Any feature that interacts with the player must handle all states. Regression risk is highest for features touching player overlays, controls, or state transitions.
- **Redux store coupling**: The single Redux store with modular reducers means new features can accidentally mutate or depend on state from other modules. Flag any feature that reads or writes to existing Redux modules beyond its own.
- **Platform-specific code path abstraction**: The Accedo XDK/VDK abstracts platform differences, but new features may require platform-specific code paths. Each new platform-specific path is a regression surface on every future release.

---

## References & Supporting Materials

For detailed information on OTT platforms, scoping, and estimation, consult:
- **`references/ott-platform-matrix.md`** - Platform capabilities, DRM, player SDKs, certification rules
- **`references/sizing-benchmarks.md`** - Story point estimation rules and platform-specific effort ranges
- **`references/epic-templates.md`** - Pre-built epic definitions for common OTT features (EPG, playback, auth, search, etc.)
- **`references/intake-questionnaire.md`** - Questions to ask during Phase 1 intake
- **`references/review-gate-templates.md`** - Checklists for Phase 2 and Phase 4 cross-functional reviews

For comprehensive discovery guidance including mandatory behavioral rules and edge case awareness, refer to the **discovery-agent** in `.claude/agents/discovery-agent.md`.

---

## Quick Start

To run discovery on a new project:

1. **Provide SOW or scope doc** to the discovery-agent
2. **Wait for Phase 1 output** (Intake Summary + Risk Register)
3. **Review and approve** Phase 1 before proceeding
4. **Phase 2**: PRD creation with parallel reviews (Design, QA, Dev, Risk)
5. **Phase 3**: Epic definition and sizing
6. **Phase 4**: Epic review gate and capacity planning
7. **Phase 5**: Final project plan and readiness checklist

---

**Version:** 2.0.0 | **Last Updated:** 2026-06-10

---

> **Skill verification:** Please ensure that the skill discovery SKILL.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
