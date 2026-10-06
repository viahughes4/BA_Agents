# Project Context - Whitelabel PRD
> Source: Accedo Broadband — Whitelabel OTT Project Scoping Sheet
> Last imported: 2026-05-09

## Purpose

This file is a **reference catalog** of features, systems, and requirements commonly delivered across whitelabel OTT projects at Accedo Broadband. Each entry lists the **possible values** — `Yes / No / N/A` for binary capabilities, numeric ranges for measurable values, and option lists for category fields.

Use this as the source of truth for **what is possible** during scoping, gap analysis, and story generation. Per-project specifics (the values selected for a given engagement) are captured separately during `/onboard-project`.

---

## Systems Requirements

### OVP (Online Video Platform)

| Requirement | Possible Values |
|---|---|
| Video Resolutions | SD / HD / 4K (any combination) |
| HDR Supported | Yes / No |
| HDR Standards | HDR10 / Dolby Vision / HLG (any combination) |
| Resolution Split | Numeric % across SD / HD / 4K |
| HDR Library Coverage | 0–100% |
| Adaptive Bitrate Streaming | HLS / DASH / Smooth Streaming (any combination) |
| Hours Streamed per User per Month | Numeric |
| Monthly Growth — Live Assets | Numeric (hours / month) |
| Monthly Growth — VOD Assets | Numeric (hours / month) |
| Annual Storage | Numeric GB / year |
| Hours of Encoding (Annual) | Numeric |
| FAST Channel Creation / Syndication | Yes / No / N/A |
| Hours of Live Events Annually | Numeric |
| Live / FAST Channel Count | Numeric (with per-year growth rate) |
| Stream Starts per Month | Numeric |
| DRM Licenses (Annual) | Numeric |
| Captioning Minutes Annually | Numeric |

### CDN

| Requirement | Possible Values |
|---|---|
| Provider | Akamai / CloudFront / Fastly / Edgio (Limelight) / Azure Media Services / Other |
| Annual Bandwidth Usage | Numeric GB / year |
| Tokenized URL Signing | Yes / No |
| Infrastructure Analytics | Yes / No |

### Privacy & Compliance

| Requirement | Possible Values |
|---|---|
| GDPR | Required / N/A |
| CCPA | Required / N/A |
| COPPA | Required / N/A |
| ATT (iOS) | Required / N/A |
| Data Residency Requirements | List of regions / None |

### IDM (Identity Management)

| Requirement | Possible Values |
|---|---|
| Number of User Accounts | Numeric |
| Authentication Method | OAuth2 / OIDC / Email + Password / SSO / Social Sign-In (any combination) |
| Sign in with Apple (iOS) | Required (if 3rd-party auth on iOS) / N/A |
| Parental Controls | Yes / No / N/A |
| Multi-Profiles | Yes / No / N/A |
| Child Profiles | Yes / No / N/A |
| Access Model | Gated / Ungated / Mixed |
| First-Line Customer Support | Yes / No |
| Concurrent Stream Limits | Numeric (typically 1–5) |

### SMS (Subscription Management)

| Requirement | Possible Values |
|---|---|
| Merchant of Record | Yes / No |
| Tax Calculations | Yes / No |
| Subscription Tiers | Free / Single Tier / Multi-Tier |
| Free Trials | Yes / No |
| Coupons / Discounting | Yes / No |
| Regions Supported | List (e.g., North America / Europe / Asia Pacific / Australia & NZ) |
| Languages Supported | List of language codes (e.g., EN / EN+FR / EN+ES+FR) |
| Churn Analytics | Yes / No |

### Recommendations

| Requirement | Possible Values |
|---|---|
| Content Recommendations | Yes / No |
| Recommendation Types | Top 10s / More Like This / Trending Now / Personalized / Editorial / Recently Added (any combination) |

### Analytics

| Requirement | Possible Values |
|---|---|
| QoS | Yes / No |
| QoE | Yes / No |
| Content Performance | Yes / No |
| User Behaviour | Yes / No |
| QoS Metrics | Bitrate / Rebuffering Ratio / Startup Time / Errors (any combination) |
| Provider | Conviva / Mux / NPAW (YOUBORA) / Adobe / Custom |

### Advertising

| Requirement | Possible Values |
|---|---|
| DAI Strategy | CSAI / SSAI / Both / N/A |
| Video Ads | Yes / No |
| Display Ads | Yes / No |
| Ad Tag Formats | VAST / VMAP / VPAID / SIMID (any combination) |
| Ads Served per Month | Numeric |
| Dynamic Ad Use Cases | Interactive / Native / Shoppable / N/A |
| Ad Reporting | Yes / No |

### Video Player

| Requirement | Possible Values |
|---|---|
| Player Type | Native / Open Source / Third Party |
| Native Examples | AVPlayer (iOS) / ExoPlayer or Media3 (Android) |
| Third-Party / OSS Examples | Bitmovin / THEOplayer / Shaka Player / Video.js / JW Player / Brightcove / Samsung PSDK / LG webOS pipeline / Amazon IVS |
| Manifest Format | HLS / DASH / Smooth Streaming (any combination) |
| DRM Systems | FairPlay / Widevine (L1 / L3) / PlayReady (any combination) |

### Front-End

| Requirement | Possible Values |
|---|---|
| Design Provided By | Customer / Accedo / Hybrid |
| Quality Assurance | Yes / No |
| QA Automation | Yes / No |
| Unit Testing | Yes / No |
| Unit Testing Coverage | 0–100% |
| Form Factors | Mobile / Desktop / 10-foot (any combination) |
| Platforms Supported | iOS / Android / Web / Roku / Samsung Tizen / LG webOS / Fire TV / Apple TV / Vizio / HiSense / Meta Quest / AVP / Google TV / Fire Tablet / X1 (any combination) |
| Remote Application Configuration | Yes / No (Accedo Control) |
| Development | Yes / No |

### Operations

| Requirement | Possible Values |
|---|---|
| Hosting | Yes / No |
| Support and Maintenance | Yes / No / Tiered SLA |

### Migration

| Requirement | Possible Values |
|---|---|
| Existing Systems Reused | List of system names / None |
| Replacement Systems | List of systems being replaced / None |
| Contract Expiry for Replaced Systems | Numeric months |
| Migration Timeline | Numeric months |

---

## Feature Catalog

A reference list of features commonly delivered across whitelabel OTT engagements. **Typical Effort** is a per-platform dev-day range — wide ranges reflect variance by platform complexity (e.g., Trickplay is fast on Roku, slower on Apple TV / Fire TV).

### Set Up / Architecture

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Architecture | Per-platform architecture review and confirmation | 2–3 |
| Project Setup | Project instance and device testing setup | 2–3 |
| Performance Optimizations | Memory management, load time, general responsiveness | 5–6 |
| Base Network Layer | Networking mechanism for all backend service calls | 3–4 |
| Caching Mechanism | On-device data storage and business logic | 2–3 |
| Service Layer & Data Modelling | Service layer components and data models | 4 |
| Error Handling | Human-readable error handling for network / service failures | 3 |
| App CMS Integration | Accedo Control integration for branding, theming, swimlane curation | 4–12 (up to 50+ for complex skinning) |
| Localization | Multi-language support | 3–4 |
| Localization (Advanced) | Right-to-left language support | 2–4 |

### Customer Management

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Registration | Sign-up via email / password or shortcode (TV devices) | 2–4 |
| Authentication — D2C | Email / password or shortcode sign-in | 2–3 |
| Authentication — TVE | TV Everywhere sign-in | 2–4 |
| Authorization / Entitlement | Paywall / premium content access restrictions | 4–5 |
| Multi-Profiles | Create / edit / manage profiles; profile-level personalization | 2–3 |
| Parental Controls | Age-rating PIN restriction within child profiles | 4 |
| PIN Entry Control | Parental PIN entry and storage (server / device) | 1–3 |
| Account Management | Manage email / password / payment (mobile / web only) | 2–4 |
| Device Management | Manage signed-in devices | 2 |
| Consent Management | User toggles for data tracking / storage | 2–3 |
| GDPR / CCPA / COPPA Compliance | Region-specific data security policy compliance | 2–4 |
| Favorites | Add / remove content from favorites list | 3–4 |

### Video Playback

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Video Player | Native or third-party player integration | 2–10 |
| Transport Controls | Play / Pause / FF / RW + platform-specific controls | 2–3 |
| Player Metadata UI | Overlay with content metadata during playback | 2 |
| Live TV Playback | Live asset playback | 2–3 |
| Live Startover | Skip to start of a live programme | 2–4 |
| Live Catchup | Seek backwards in live and return to live | 2–3 |
| VOD Playback | VOD asset playback | 2 |
| Video Bookmarking | Resume watching / continue watching | 3–4 |
| Closed Captioning | On-screen subtitles | 2–3 |
| Multiple Audio Track Support | Descriptive audio / SAP / surround sound | 2–3 |
| Picture-in-Picture | Mini-player when navigating or after close | 2–4 |
| Binge Mode | Autoplay next episode in series | 0.5–2 |
| Trickplay | Thumbnail scrubbing on progress bar | 1–5 |
| Concurrency Management | Limit concurrent streams per account | 2 |
| DRM | Copyright protection and access rights enforcement | 2 |
| Background Playback | Continued playback while navigating | 2–4 |
| Chromecast / AirPlay | Cast support (iOS, Android, Web) | 2–4 |
| Multi-View | Multiple simultaneous video instances | 5–10 |
| Overlays | Semi-transparent overlays during playback (stats, polls) | 2–4 |
| Vertical Video Support | TikTok / Shorts-style vertical playback (mobile only) | 4–8 |

### Analytics

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Analytics Base | Base integration for analytics network calls | 3–6 |
| Pageview Events | User pageview analysis | 5 |
| User Interaction Events | User behaviour and interaction tracking | 5 |
| Content Performance Events | Viewing patterns, popularity, completion rates | 3–5 |
| Playback / QoS / Heartbeat Events | Performance and quality metrics | 2–3 |

### Advertising

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Dynamic Ad Insertion (DAI) | CSAI or SSAI integration | 4–10 |
| Display Advertisements | Ads visible as swimlanes / icons (mobile, web) | 2–4 |
| Video Advertisements | Pre / mid / post-roll via VAST tags | 3–5 |
| Interactive Advertising | Gamified or actionable ads via VPAID / SIMID | 4–8 |

### In-App Purchase

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Payment / Billing | IAP support for subscription purchases | 4–8 |
| Multi-Tier Subscriptions | Multiple tiers (hybrid ad loads, quality, concurrency) | 3–5 |
| Free Trials | Trial creation and management | 2–3 |
| Coupons | Marketing coupon redemption (web-based) | 2–3 |
| e-Commerce | Physical merchandise purchase / discovery | 5–10+ |

### User Experience

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Scroll Controls | Vertical swimlane scroll + horizontal content scroll | 2 |
| Swimlane Types | Carousel / hero banner / etc. | 2 |
| Global Navigation Menu | Main menu with dynamic placement | 8–12 |
| Navigational Context Mechanism | Indicate current location within app | 2–3 |
| Settings | System settings + account management | 3 |
| Version / About Page | App version display | 1 |
| Video Quality Selection | Good / Better / Best | 1–2 |
| Language Option | Language selection | 2–3 |
| Logout / Deactivate | Sign out | 1–2 |
| Image Style Support | JPG / PNG / SVG | 1 |
| Splash Screen | Static or animated logo on app load | 1–2 |
| Movie Detail Page | Metadata / visuals / related content | 3 |
| Series Detail Page | Metadata / visuals / seasons / related content | 2 |
| Live Event Detail Page | Metadata / visuals / related content | 2 |
| Home Page | Main discovery experience (hero, swimlanes, favourites, continue watching) | 3 |
| Shows Browse Page | Series browse via Accedo Control | 2 |
| Movies Browse Page | Movies browse via Accedo Control | 2 |
| Poster Tiles | Movie / Series / Live Event / Sports tiles | 2 |
| Channel Page | Channel metadata / visuals / current + past live | 2 |
| Onboarding | First-launch feature walkthrough | 2 |
| Custom Widgets | Bespoke UI widgets | Varies |
| Gamification | Polls / quizzes / challenges | 4–8 |
| Rewards | Points-based redemption | 3–6 |
| Offline Downloads | Download / manage / play offline (mobile only) | 5–10 |
| Watch Parties | Co-viewing for live events / shows (mobile, web) | 5–10 |
| Shorts Experience | Short-form vertical video discovery (mobile only) | 5–10 |
| User Generated Content | Upload / moderation / management / flagging | 8–15+ |

### EPG

| Feature | Description | Typical Effort (days) |
|---|---|---|
| EPG | Full-screen live TV guide | 2–3 |
| EPG Advanced Filters & Favouriting | Genre / type filters; favourite channels | 2–3 |
| EPG Live Preview | PiP preview of live content (CTV) | 3 |
| Mini-guide | Overlay guide during live TV playback | 1–3 |

### DVR

| Feature | Description | Typical Effort (days) |
|---|---|---|
| DVR | Cloud or on-device recording of live content | 2–3 |
| DVR Reminders | Notifications for recording start / completion | 1–2 |
| DVR Commands | Record / delete / set recording for X time | 0.5–1 |

### Content Discovery

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Deeplinking | Launch app from notifications and marketing emails | 3–5 |
| Universal Search | OS-level content discovery integration | 5 |
| Push Notifications | Marketing and social notifications (mobile only) | 2–4 |
| Social Sharing | Share to Facebook / Twitter / etc. (mobile, web) | 2–3 |
| Search | Asset search with recommendations + history | 2–4 |
| Advanced Search | Search filtering and history | 2–4 |
| Voice Search | Remote voice search (CTV) | 0–8 (Apple TV often 0 with built-in) |
| Recommendations | Top 10s / More Like This / Recommended | 2–5 |
| Recommendations (Post-Completion) | Full-screen binge-mode recommendations | 2–4 |

### Accessibility

| Feature | Description | Typical Effort (days) |
|---|---|---|
| Text-to-Speech (TTS) | Visually-impaired navigation support (Web, Samsung, Vizio, X1) | 3–6 |
| WCAG 2.1 AA Compliance | Accessibility standards (web + applicable platforms) | 5–10 |
| Voice Commands | Playback / navigation voice commands (CTV) | 4–8 |

### PayTV

| Feature | Description | Typical Effort (days) |
|---|---|---|
| TIF Integration | Google TV Input Framework (Android TV) | 5–10 |
| Google Assistant Integration | Native OS assistant (Android TV / Google TV) | 3–5 |
| Emergency Alert System | Full-screen takeover for emergency alerts (legal requirement for Canadian Pay TV) | 3 |

---

## Platform Certification Requirements

Universal cert reference for whitelabel OTT projects.

### Roku
- Roku Event Dispatcher — required for SVOD / TVE authentication
- Roku Voice Control — required at 5M+ avg hours / month
- Roku Trickplay — required for VOD > 15 minutes
- RFI Screen for sign-in — required for authenticated SVOD / TVOD
- Roku Bookmarking — required for VOD > 15 minutes (persist 30+ days)
- Roku Demand API — required at 200K+ hours / month outside US
- Roku Voice Keyboard — required for email and PIN entry

### Apple TV / iOS
- Sign in with Apple — required if any 3rd-party auth offered
- ATT prompt — required for any ad / analytics tracking
- ATS — all endpoints HTTPS
- FairPlay DRM only (no Widevine / PlayReady)
- HIG compliance for UI patterns
- Background audio session for PiP / audio-only content
- Catalogue + availability feed for Universal Search
- Content ratings + metadata complete

### Fire TV
- D-pad navigation on every interactive element (no touch fallback)
- Amazon IAP for in-app purchases (no third-party payment)
- Fire TV UX Guidelines compliance
- Deep link schema for Alexa voice commands
- VSK / Media Session API for voice control (if catalog integration available)

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
- Widevine L1 for HD / 4K DRM content
- Content ratings + Data Safety declarations complete

### Web
- WCAG 2.1 AA compliance
- EME / MSE for DRM
- CORS configuration for license + manifest endpoints
- Codec fragmentation handling (H.264 baseline; HEVC / AV1 fallback strategy)

---

## Performance KPIs (Targets & Typical Ranges)

### Core App KPIs

| KPI | Target |
|---|---|
| Sign-in to Homepage (first launch) | ≤ 10s (≤ 15s on Roku per cert requirement) |
| Sign-in to Homepage (re-launch) | ≤ 5s |
| Sign-in to Playback | ≤ 60s end-to-end |
| App Freezes | None — efficient memory management |
| Homepage Load (best) | ≤ 2s after first launch |
| Homepage Load (acceptable upper bound) | 7–8s |
| Home Screen — tvOS / Apple TV | ≤ 6s |
| Home Screen — Samsung / Fire TV / Roku | ≤ 12s |

### EPG & Playback KPIs (consider for PRD inclusion)

| KPI | Description |
|---|---|
| EPG Load from Homepage | Time to load full EPG |
| EPG Responsiveness | Time to navigate up / down between channels |
| On Now Guide Load | Time to load on-now guide |
| EPG Load from On Now Guide | Time to load EPG from on-now |
| Mini-guide Navigation | Average time scrolling mini-guide |
| View All Pages | Time to fully render all swimlanes |
| VOD Playback Start | Time from selection to playback |
| Live Playback Start | Time from selection to live stream start |
| Skip Back / Forward | Time to execute skip |
| Live TV Return | Time to return to live from pause / catchup |
| DVR Page Load | Time from selection to DVR playback |

### Typical Platform Benchmark Ranges
*Reference data from prior Accedo deliveries. Actual results vary by device model, OS version, and connection.*

| KPI | Apple TV (HD 4th Gen) | Fire TV (Stick 4K) | Samsung 2021 | Roku Premiere |
|---|---|---|---|---|
| EPG Load (fresh) | 3–5s | 2–3s | 8–10s | 4–6s |
| EPG Load (subsequent) | 1–2s | 2s | 5–7s | 2–3s |
| EPG Responsiveness | < 0.05s | Instant | 1.5–2.5s | < 1s |
| VOD Playback Start | 5–6s | 17–20s | 25–30s | 5–7s |
| Live Playback Start | 5–6s | 6–7s | 23–25s | 5–7s |
| Skip Back / Forward | ~1s | ~1.5s | ~1.5s | 1s |
