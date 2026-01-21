# Assumption Register

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## Purpose

This document tracks all assumptions underlying the Unofficial Communities platform. Each assumption includes its scope, evidence level, risk if false, and validation plan.

---

## Assumption Status Definitions

| Status | Definition |
|--------|------------|
| **Active** | Currently assumed to be true, system depends on this |
| **Validated** | Confirmed true through evidence |
| **Invalidated** | Proven false, mitigation in progress |
| **Retired** | No longer relevant to current system |

## Evidence Level Definitions

| Level | Definition |
|-------|------------|
| **High** | Empirical data, user research, or industry standard |
| **Medium** | Expert opinion, case studies, or analogous systems |
| **Low** | Intuition, hypothesis, or limited data |

---

## Assumptions

### ASM-001: WhatsApp Remains Primary Communication Channel
**Scope:** Market
**Evidence Level:** High
**Status:** Active

**Description:**
Brazilian communities will continue to use WhatsApp as their primary communication channel for the foreseeable future.

**Evidence:**
- WhatsApp has 99% penetration in Brazil
- No viable competitor gaining traction
- User behavior deeply entrenched

**Risk if False:**
- Core value proposition becomes irrelevant
- Integration work wasted
- Need to pivot to alternative platforms

**Validation Plan:**
- Monitor market share reports quarterly
- Track user-reported communication preferences
- Watch for Telegram/Discord growth in target demographics

---

### ASM-002: Community Leaders Will Pay for Tools
**Scope:** Market
**Evidence Level:** Medium
**Status:** Active

**Description:**
Community leaders and admins are willing to pay for tools that improve community health and engagement.

**Evidence:**
- Competitor pricing exists (Community.com, Discord Nitro)
- Early user interviews show interest
- B2B2C model validated in adjacent markets

**Risk if False:**
- Revenue model fails
- Need to find alternative monetization
- May need to pivot to pure B2B

**Validation Plan:**
- Pricing page A/B tests
- Conversion rate tracking
- User willingness-to-pay surveys

---

### ASM-003: Gamification Improves Community Health
**Scope:** Product
**Evidence Level:** Medium
**Status:** Active

**Description:**
XP, levels, streaks, and missions will improve member engagement without causing unhealthy addiction.

**Evidence:**
- Behavioral science literature (Fogg, SDT)
- Similar systems successful in Duolingo, fitness apps
- Early user feedback positive

**Risk if False:**
- Core feature set ineffective
- May cause user burnout
- Differentiation lost

**Validation Plan:**
- Track D1/D7/D30 retention
- Monitor session length distribution (watch for concerning patterns)
- User satisfaction surveys
- Compare engaged vs non-engaged cohorts

---

### ASM-004: 3D World Adds Value Beyond 2D
**Scope:** Product
**Evidence Level:** Low
**Status:** Active

**Description:**
A 3D virtual world provides meaningfully better social presence than 2D alternatives or text-only.

**Evidence:**
- Spatial audio research
- VRChat/Gather.town adoption
- User requests for "more immersive" experience

**Risk if False:**
- UC World development investment wasted
- Users may prefer simpler 2D/text interfaces
- Performance issues on low-end devices

**Validation Plan:**
- A/B test 3D vs non-3D user cohorts
- Track time spent in world vs other features
- User preference surveys
- Device capability analytics

---

### ASM-005: Brazilian Users Accept Portuguese-Only UI
**Scope:** Market
**Evidence Level:** High
**Status:** Active

**Description:**
Target users are comfortable with a Portuguese-only interface and do not require English or other languages.

**Evidence:**
- Target market is Brazil-only initially
- Competitors (Brazilian) are Portuguese-only
- No user requests for translation

**Risk if False:**
- Expansion limited
- May miss bilingual user segments
- i18n retrofit expensive

**Validation Plan:**
- Track user location data
- Monitor language-related support requests
- Survey for language preferences if expanding

---

### ASM-006: Single VPS Sufficient for Launch Scale
**Scope:** Technical
**Evidence Level:** Medium
**Status:** Active

**Description:**
A single Hostinger VPS (4 cores, 16GB RAM) can handle initial launch traffic and growth for 6-12 months.

**Evidence:**
- Current resource utilization: <20% CPU, <50% memory
- Load testing results (internal)
- Similar systems at comparable scale

**Risk if False:**
- Performance degradation
- Need emergency scaling
- User experience suffers

**Validation Plan:**
- Prometheus metrics monitoring
- Define scaling triggers (CPU >70%, response time >500ms)
- Document horizontal scaling plan

---

### ASM-007: Meta API Remains Stable
**Scope:** Technical
**Evidence Level:** Medium
**Status:** Active

**Description:**
WhatsApp Cloud API and Instagram Graph API will maintain backward compatibility and not have breaking changes.

**Evidence:**
- Meta's API versioning policy
- No major breaking changes in past 2 years
- Business-critical API for Meta revenue

**Risk if False:**
- Integration breaks
- Need emergency fixes
- User verification flow fails

**Validation Plan:**
- Monitor Meta developer announcements
- Subscribe to API changelog
- Maintain version pinning strategy
- Regular integration tests against API

---

### ASM-008: Users Will Complete Onboarding
**Scope:** Product
**Evidence Level:** Low
**Status:** Active

**Description:**
Users will complete the WhatsApp verification and profile setup flow before accessing features.

**Evidence:**
- Required for community verification (necessary)
- Similar flows exist in competitors
- No alternative identified

**Risk if False:**
- High drop-off rate
- Reduced user acquisition
- Need to simplify flow

**Validation Plan:**
- Track onboarding funnel conversion
- Identify drop-off points
- A/B test flow variations
- User feedback on friction points

---

### ASM-009: Community Size 50-500 Members Optimal
**Scope:** Product
**Evidence Level:** Medium
**Status:** Active

**Description:**
The platform is optimized for communities of 50-500 active members. Smaller communities lack critical mass; larger ones lose intimacy.

**Evidence:**
- Dunbar's number research
- WhatsApp group size limits (1024)
- Competitor target sizes

**Risk if False:**
- May miss larger community market
- May not serve small tight-knit groups
- Scaling calculations incorrect

**Validation Plan:**
- Track community size distribution
- Correlate size with health metrics
- User interviews across size segments

---

### ASM-010: Ethical Design Is Competitive Advantage
**Scope:** Market
**Evidence Level:** Low
**Status:** Active

**Description:**
Users and community leaders value ethical, non-manipulative design and will choose it over engagement-hacking alternatives.

**Evidence:**
- Growing awareness of dark patterns
- Regulatory pressure (LGPD, GDPR)
- Early user positive feedback on transparency

**Risk if False:**
- Competitors with dark patterns may win
- "Ethics" not a purchasing criterion
- Slower growth than aggressive competitors

**Validation Plan:**
- User research on decision factors
- Track win/loss reasons vs competitors
- Monitor regulatory environment

---

### ASM-011: Colyseus Scales to 50 Users Per Room
**Scope:** Technical
**Evidence Level:** Medium
**Status:** Active

**Description:**
Colyseus can handle 50 concurrent users per UC World room with acceptable latency (<100ms state sync).

**Evidence:**
- Colyseus documentation claims
- Similar deployments in industry
- Initial load testing

**Risk if False:**
- UC World performance issues
- Need alternative architecture
- Room size limits lower than planned

**Validation Plan:**
- Load testing with synthetic users
- Production monitoring of room sizes
- Latency percentile tracking

---

### ASM-012: Users Have Modern Browsers
**Scope:** Technical
**Evidence Level:** High
**Status:** Active

**Description:**
Target users have access to modern browsers (Chrome 90+, Safari 14+, Firefox 90+, Edge 90+) that support ES2020+, WebGL 2.0.

**Evidence:**
- Brazil browser market share data
- Target demographic (smartphone users) typically have auto-updating browsers
- WhatsApp web users already require modern browsers

**Risk if False:**
- UC World won't render
- JavaScript features fail
- Need polyfills or fallbacks

**Validation Plan:**
- Analytics on browser versions
- WebGL capability detection
- Graceful degradation for unsupported browsers

---

## Assumption Index

| ID | Description | Scope | Evidence | Status |
|----|-------------|-------|----------|--------|
| ASM-001 | WhatsApp Remains Primary Communication | Market | High | Active |
| ASM-002 | Community Leaders Will Pay for Tools | Market | Medium | Active |
| ASM-003 | Gamification Improves Community Health | Product | Medium | Active |
| ASM-004 | 3D World Adds Value Beyond 2D | Product | Low | Active |
| ASM-005 | Brazilian Users Accept Portuguese-Only UI | Market | High | Active |
| ASM-006 | Single VPS Sufficient for Launch Scale | Technical | Medium | Active |
| ASM-007 | Meta API Remains Stable | Technical | Medium | Active |
| ASM-008 | Users Will Complete Onboarding | Product | Low | Active |
| ASM-009 | Community Size 50-500 Members Optimal | Product | Medium | Active |
| ASM-010 | Ethical Design Is Competitive Advantage | Market | Low | Active |
| ASM-011 | Colyseus Scales to 50 Users Per Room | Technical | Medium | Active |
| ASM-012 | Users Have Modern Browsers | Technical | High | Active |

---

*Next Assumption ID: ASM-013*

<!-- Last Reviewed: 2026-01-20 - No updates needed -->
