---
name: cert-checklist
description: Use this skill when the user wants to check platform certification readiness before a release. Triggers on "check cert readiness", "run the cert checklist", "are we ready for certification?", "cert requirements for this release", "what do I need for Samsung cert?", "what do I need for Apple cert?", or any request to review release scope against platform certification requirements.
---

# Skill: Certification Checklist

Platform certification readiness check. Reviews release / sprint scope against per-platform cert requirements and flags gaps.

---

## Files

- **Canonical cert reference:** `OTT-requirements-reference.md` -> *Platform Certification Requirements* section. The list below is a working copy - if the catalog updates, treat the catalog as authoritative.
- **Project scope:** Loaded via `.claude/sub/context-loader.md` -> *Platform Launch Plan* and *Systems* (especially DRM, IDM, Ads). Determines which platforms and which cert requirements apply to this run.

---

## Process

### 1. Load context
Read `.claude/sub/context-loader.md` and follow its process to load project context, Jira key, team roster, and folder paths. If 🚫, stop and recommend `/onboard-project`; cert-checklist needs a platform list to be meaningful.

### 2. Accept release scope
- Sprint scope: invoke `.claude/sub/jira-project-snapshot.md` to get current sprint scope if Jira connected, else ask the PM.
- Or: PM-supplied list of features being released.

### 3. Determine platforms in scope
Filter platform cert checks to only the platforms in the release.

### 4. Run per-platform checks
For each requirement, rate:
- ✅ Addressed in sprint
- ⚠️ In scope but no evidence of cert-specific implementation
- 🚫 Missing
- N/A

### 5. Produce report
- Save the output as a `.md` file to `product-development/product/customers/accounts/[client]/sprints/` using the naming convention `YYYY-MM-DD-cert-checklist-[platform].md`.
- Run `python3 scripts/md-to-html.py <saved-path>` and report the HTML path to the user.

---

## Per-Platform Requirements
*(Working copy of `OTT-requirements-reference.md` -> Platform Certification Requirements. If they diverge, the catalog wins.)*

### Roku
- Trickplay thumbnails for VOD > 15 min
- Bookmarking for VOD > 15 min (persist 30+ days)
- Roku Voice Keyboard for email / PIN entry
- Roku Event Dispatcher for authenticated channels
- Voice control if avg > 5M hours/month
- Demand API if > 200K hours/month outside US
- RFI screen for SVOD/TVOD sign-in
- Launch-to-rendered: <= 15s (Stick+) / <= 20s (Express)

### Apple TV / iOS
- Sign in with Apple if any third-party auth offered
- ATT prompt if any ad / analytics tracking
- ATS: all endpoints HTTPS
- FairPlay DRM only (no Widevine / PlayReady)
- HIG compliance for UI patterns
- Background audio session for PiP / audio content
- Catalogue feed for Universal Search
- App content ratings and metadata complete

### Fire TV
- D-pad navigation on every interactive element (no touch fallback)
- Amazon IAP for in-app purchases (no third-party payment)
- Fire TV UX Guidelines compliance
- Deep link schema for Alexa voice commands
- VSK / Media Session API for voice (if catalog integrated)

### Samsung Tizen
- Text-to-Speech (TTS) support
- UI guidelines compliance (Samsung Seller Office checklist)
- PSDK integration for DRM and playback
- Memory budget compliance (texture memory audit)
- Tizen OS multi-generation support documented

### LG webOS / Lightning
- webOS version targeting in app manifest
- Lightning component lifecycle compliance
- D-pad focus management on nested components
- Texture memory within LG limits
- LG Seller Lounge submission requirements met

### Android / Google Play
- Target SDK current per Google Play annual requirement
- Google Play Integrity API for device attestation
- Widevine L1 for HD/4K DRM content
- Content ratings + Data Safety declarations complete

### Web
- WCAG 2.1 AA compliance
- EME / MSE for DRM
- CORS configuration for license + manifest endpoints
- Codec fragmentation handling (H.264 baseline; HEVC / AV1 fallback strategy)

---

## Output Format

```
## Certification Readiness: [Project] - [Platform(s)] - [Date]

### Release Scope
[Summary of features in scope for this release]

### Cert Risk Summary
**Overall risk:** Low / Medium / High
[One-sentence justification]

### Per-Platform Checklist

#### iOS / Apple TV
| Requirement | Status | Notes / Action |
|---|---|---|
| Sign in with Apple (if 3rd-party auth) | ✅/⚠️/🚫/N/A | ... |
| ATT prompt (if tracking) | ... | ... |
| ATS HTTPS-only | ... | ... |
| FairPlay only | ... | ... |
| HIG compliance | ... | ... |
| Background audio session | ... | ... |
| Universal Search feed | ... | ... |
| Content ratings / metadata | ... | ... |

#### Fire TV
| Requirement | Status | Notes / Action |
|---|---|---|
| D-pad full coverage | ... | ... |
| Amazon IAP | ... | ... |
| Fire TV UX guidelines | ... | ... |
| Alexa deep links | ... | ... |
| VSK / Media Session | ... | ... |

#### Samsung Tizen
[...]

#### LG webOS
[...]

#### Roku
[...]

#### Android / Google Play
[...]

#### Web
[...]

### Critical Risks (Must resolve before submission)
- [Platform - requirement]: [Action needed]

### Warnings (Resolve before release)
- [Platform - requirement]: [Action needed]

### Recommended Pre-Submission Actions (prioritized)
1. [...]
2. [...]
```

---

## Quality Rules

- Only check platforms actually in the release. Don't generate noise about Roku if Roku isn't shipping.
- 🚫 Missing on a cert-blocking item (ATT, Amazon IAP, Sign in with Apple, Widevine L1) -> Critical Risk.
- ⚠️ items can become 🚫 if not addressed in time; call this out explicitly.
- Be specific. "Confirm ATT prompt is implemented before any IDFA / tracking SDK call" beats "verify tracking compliance."
- If context-loader does not surface a DRM provider or platform list, flag that as a precondition gap before running this skill.
- Always state the cert authority (App Store, Google Play, Amazon, Samsung Seller Office, LG Seller Lounge, Roku) when calling out a risk.
- This skill consults the canonical cert list in `OTT-requirements-reference.md`. If you encounter a cert requirement not represented in the catalog, surface it to the PM and recommend updating the catalog.

---

> **Skill verification:** Please ensure that the skill cert-checklist.md was actually run. If the skill was not run, then it needs to be run again from the top to ensure it actually gets used.
