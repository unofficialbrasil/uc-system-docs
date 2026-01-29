# Two-Pillar SaaS Strategy

**System:** Unofficial Communities
**Last Updated:** 2026-01-29
**Version:** 3.0
**Status:** Approved Strategy — Implementation Pending Gate Approval

---

## 1. Executive Summary

Unofficial Communities operates a dual-core SaaS architecture that separates **Social Engagement** (Pillar 1) from **Commercial Conversion** (Pillar 2), connected by an intelligence layer called **The Bridge**.

**Core Model:**
- **Pillar 1 — Social Engagement Ecosystem (UC World 3D):** Builds community belonging, autonomy, and competence through ethical gamification and spatial presence. Outputs behavioral intelligence at community and cohort level.
- **Pillar 2 — Branded Virtual Spaces (Managed Showrooms):** UC builds, operates, and continuously optimizes branded virtual environments within the platform. Qualifies users into leads via AI bot + optional form, then hands off to brand sales representatives.
- **The Bridge:** Intelligence layer that detects community-level readiness and user-level intent, routing qualified users to the right branded spaces at the right moment without manipulation.

**Business Model:**
- UC builds and operates all branded spaces (managed service)
- Revenue: per-qualified-lead pricing and/or monthly management fee (low initial pricing to build data moat)
- Value increases over time as the matching engine learns which communities produce the best leads for each brand

**Key Properties:**
- All behavioral data stays within UC's controlled environment (consent, age verification, LGPD compliance)
- No individual profiling — cohort aggregates determine community readiness; explicit user actions determine routing
- Both communities and branded spaces generate data that improves matching and qualification for the entire platform

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TWO-PILLAR ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PILLAR 1: Social Engagement          PILLAR 2: Branded Spaces         │
│  ┌─────────────────────────┐          ┌─────────────────────────┐      │
│  │ UC World 3D             │          │ Virtual Showrooms       │      │
│  │ • 7 interior zones      │          │ • Brand-customized env  │      │
│  │ • 6 portal zones        │          │ • AI bot qualification  │      │
│  │ • Gamification system   │          │ • Lead capture form     │      │
│  │ • Living Graph          │          │ • Performance analytics │      │
│  │ • Community portals     │          │ • A/B tested variants   │      │
│  └────────────┬────────────┘          └────────────┬────────────┘      │
│               │                                    │                    │
│               │         ┌──────────────┐           │                    │
│               └────────►│  THE BRIDGE  │◄──────────┘                    │
│                         │              │                                │
│                         │ • Community  │                                │
│                         │   readiness  │                                │
│                         │ • Intent     │                                │
│                         │   detection  │                                │
│                         │ • Community  │                                │
│                         │   ↔ Brand    │                                │
│                         │   matching   │                                │
│                         │ • Feedback   │                                │
│                         │   loops      │                                │
│                         └──────────────┘                                │
│                                                                         │
│  DATA FLOWS:                                                            │
│  Communities → behavioral signals → Bridge → routing decisions          │
│  Branded spaces → conversion data → Bridge → better matching           │
│  Platform-wide learning → improved qualification for all brands        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Pillar 1: Social Engagement Ecosystem

### 3.1 Purpose

Build community belonging, autonomy, and competence through ethical gamification and spatial presence. Generate behavioral intelligence that feeds the Bridge without individual profiling.

### 3.2 Existing Capabilities (Already Built)

| Capability | Description | Data Output |
|------------|-------------|-------------|
| Living Graph | Daily community-to-community proximity mapping | Community affinity scores, portal assignments |
| Proxemics | Zone-based density management (Hub, Lounges, Workshop, Quiet, Brand, Onboarding) | Zone dwell patterns, diversity metrics |
| Gamification | Missions, XP, streaks, levels, rankings | Mission completion rates, engagement depth |
| Portal System | 6-neighbor connectivity with explainability | Cross-community travel, bounce rates |
| Behavioral Framework | Fogg MAP + SDT constraints | Engagement health, social reciprocity |

### 3.3 Social Intelligence Generated

All signals are **community-level and cohort-level**, never individual profiles.

| Signal | Data Source | Interpretation |
|--------|------------|----------------|
| Mission Velocity | `mission_activity_daily` | Cohort % completing ≥5 missions in rolling 7d |
| Zone Diversity | `zone_dwell_daily` | Unique zones visited / total zones in 7d |
| Social Reciprocity | Co-presence + lounge usage | Community % with ≥2 lounge sessions in 7d |
| Brand Zone Opt-in | Portal/zone entry events | Explicit click to enter brand zone |
| Session Persistence | WhatsApp activity + world presence | D7 retention and repeat 3d+ activity |
| Cross-Community Flow | Portal travel events | Travel rates, bounce rates, visit diversity |

### 3.4 Three-Layer Graph Model

| Layer | Entities | Purpose | Guardrail |
|-------|----------|---------|-----------|
| Interest Graph | Users → Zones / Tags / Missions | Cold-start discovery, interest matching | Prefer explicit signals (zone visits, mission types); minimize behavioral inference |
| Social Graph | Users → Co-presences, co-participation | Belonging, trust, reciprocity | Aggregates only; no dyadic edges at individual level |
| Community Graph | Communities → Communities (6-neighbor portals) | Cross-community health, discovery, partnerships | Aggregate signals; never expose individual member names or IDs |

**Extension for Pillar 2:** The Community Graph adds a **community-to-brand affinity** dimension (see Section 5.3), computed from conversion data flowing back from branded spaces.

---

## 4. Pillar 2: Branded Virtual Spaces

### 4.1 Purpose

UC builds, operates, and continuously optimizes branded virtual environments within the platform. These spaces qualify high-intent users into leads via AI bot interaction and optional form capture, then hand off qualified leads to brand sales representatives.

### 4.2 Operating Model

**UC builds and operates all branded spaces.** Brands do not self-serve.

| Aspect | Model |
|--------|-------|
| Who builds the space | UC (customized to brand requirements) |
| Who operates it | UC (managed service) |
| Who configures the bot | UC (brand provides product knowledge, qualification criteria, escalation rules) |
| Who receives leads | Brand sales team (via handoff interface or CRM export) |
| Who optimizes | UC (continuous A/B testing, layout iteration, bot refinement) |
| Revenue model | Per-qualified-lead and/or monthly management fee |

**Early-stage pricing:** Low initial pricing or per-qualified-lead only. The priority is building the data moat — each brand's conversion data improves the matching engine for all brands.

### 4.3 Branded Space Types

UC supports both high-consideration and low-consideration purchase paths:

| Type | Example | Qualification Depth | Lead Goal | Key Metric |
|------|---------|--------------------:|-----------|------------|
| High-consideration | Car dealership, real estate, enterprise software | Deep (budget, timeline, location, specific needs) | Qualified appointment or test-drive booking | Lead-to-visit conversion rate |
| Low-consideration | E-commerce, digital products, subscriptions | Light (interest, size/preference, email) | Purchase intent or cart start | Showroom-to-purchase rate |

The bot conversation flow, form fields, and qualification criteria are configured per brand and purchase type.

### 4.4 Showroom Components

```
┌─────────────────────────────────────────────────────────┐
│                  BRANDED VIRTUAL SPACE                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────────────┐    ┌───────────────────┐          │
│  │  Virtual          │    │  AI Bot           │          │
│  │  Environment      │    │                   │          │
│  │  • Custom layout  │    │  • Brand-specific │          │
│  │  • Brand assets   │    │    personality    │          │
│  │  • Interactive     │    │  • 3 modes:      │          │
│  │    elements       │    │    educational,   │          │
│  │  • Zone design    │    │    consultative,  │          │
│  │                   │    │    sales-oriented │          │
│  └───────────────────┘    └────────┬──────────┘          │
│                                    │                      │
│                           ┌────────▼──────────┐          │
│                           │  Lead Capture     │          │
│                           │  Form             │          │
│                           │  • Max 5 fields   │          │
│                           │  • Explicit        │          │
│                           │    consent        │          │
│                           │  • Age gate (≥18, │          │
│                           │    level 2+)      │          │
│                           └────────┬──────────┘          │
│                                    │                      │
│                           ┌────────▼──────────┐          │
│                           │  Sales Handoff    │          │
│                           │  • Lead summary   │          │
│                           │  • Intent score   │          │
│                           │  • Context (no    │          │
│                           │    transcripts)   │          │
│                           │  • CRM export     │          │
│                           └───────────────────┘          │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### 4.5 AI Bot Architecture

The bot operates in three modes, determined by the user's intent stage (see Section 5.2):

| Mode | Trigger | Behavior | Data Collection | Hard Sell |
|------|---------|----------|-----------------|-----------|
| Educational | Exploratory intent | Educate, answer FAQs, explore fit | Minimal (metadata only) | No |
| Consultative | Qualitative intent | Identify needs, assess fit, suggest options | Moderate (intent keywords, question count) | No |
| Sales-Oriented | Transactional intent | Qualify lead, present offer, route to form | Full (qualification criteria) | Allowed (transparent) |

**Bot Session Logging — Metadata Only:**

```typescript
// What to log:
{
  sessionId: uuid(),
  botMode: 'educational' | 'consultative' | 'sales_oriented',
  userQuestionCount: 5,
  botAnswerCount: 5,
  intentKeywordsDetected: ['pricing', 'timeline'],
  userSentimentProxy: 'positive',  // Derived from actions, not NLP
  resolutionType: 'form_started' | 'question_answered' | 'timeout' | 'user_exit',
  dwell_seconds: 240
}

// What NOT to log:
// - Message text or conversation transcripts
// - Bot responses
// - Any identifiable content beyond keywords
```

**Bot Qualification Metrics:**

| Metric | Definition | Use |
|--------|------------|-----|
| question_count | Questions asked by user | Curiosity proxy |
| answer_depth | Bot answers / user questions ratio | Engagement depth |
| intent_signal_strength | Keyword extraction (pricing, timeline, etc.) | Intent classification |
| ui_interaction_count | CTA clicks, form field focuses | Explicit intent |
| dwell_time_sec | Time in showroom | Consideration depth |
| lead_quality_score | Composite (1-10) | Downstream CRM routing |

**Handoff to Sales Representative:**

The bot-to-human transition includes:
- Lead summary (qualification criteria met, intent score, sentiment proxy)
- Interaction metadata (question count, dwell time, keywords detected)
- No conversation transcripts (privacy constraint)
- Brand-specific context (which product/service the user explored)
- Fallback SLA: 2h for high-priority escalations, 24h for medium

### 4.6 Lead Capture Form — Privacy by Design

**Mandatory Controls:**
- Explicit consent checkbox (NOT pre-checked)
- One-click withdrawal (7-day window)
- 90-day retention (auto-delete if not exported to brand)
- 1-year retention (if exported, then hard delete)
- Age assurance level 2+ required
- Max 5 form fields

**Allowed Fields:** name, email, company, specific need, optional field (brand-configurable)

**Prohibited Fields:** phone (unless separate consent), title/role (discrimination proxy), age (already verified), location beyond country

**Confirmation email:** Sent immediately with withdrawal link and partner privacy policy reference.

---

## 5. The Bridge: Intelligence Layer

### 5.1 Community-Level Readiness

Communities are classified by readiness for branded space activation:

**Community Readiness Score (progressive weighting):**

```
Phase 1 (Pilot, <10 communities):
  ReadinessScore = 0.2(Engagement) + 0.2(Social) + 0.1(Intent) + 0.5(Safety)

Phase 2 (Growth, 20-30 communities):
  ReadinessScore = 0.25(Engagement) + 0.25(Social) + 0.2(Intent) + 0.3(Safety)

Phase 3+ (Scale, all eligible communities):
  ReadinessScore = 0.3(Engagement) + 0.3(Social) + 0.2(Intent) + 0.2(Safety)
```

Where:
- **Engagement** = D7_retention × mission_completion_rate × avg_session_persistence
- **Social** = entropy_social × reciprocity_proxy × zone_diversity
- **Intent** = brand_zone_opt_in_rate × cross_community_travel × lounge_utilization
- **Safety** = moderation_response_time × report_rate_inverse × leader_stability

**Decision Tiers:**

| Tier | Score | Action |
|------|-------|--------|
| Tier 1 (Green) | ≥ 0.75 (Phase 1), ≥ 0.70 (Phase 2), ≥ 0.65 (Phase 3) | Eligible for branded space routing |
| Tier 2 (Yellow) | 0.50 – threshold | Observe for 2 weeks, then re-evaluate |
| Tier 3 (Red) | < 0.50 | Not ready; engagement-only |

### 5.2 Intent Stage Framework

User intent is classified through multi-signal confirmation. Stage progression requires both aggregate signals AND explicit actions.

| Stage | Signals | Bot Mode | Offer Strategy | False Positive Risk |
|-------|---------|----------|----------------|---------------------|
| **Latent** | Engaged in Pillar 1 only | N/A | No offers | Low |
| **Exploratory** | Zone diversity ≥4 + brand zone ≥1 + D7 retention | Educational | Passive suggestion (1/month) | Medium (just exploring) |
| **Qualitative** | Exploratory + brand zone ≥2 (REPEATED) + active 3d+ | Consultative | Contextual card | Low (confirmed interest) |
| **Transactional** | Qualitative + explicit action (click/question) | Sales-Oriented | Optional form | Very Low (proof of intent) |

**Key Rules:**
- Require repeated signals (brand zone 2+ times) before suggesting showroom — one-time visits are exploratory curiosity, not buying intent
- Never auto-escalate based on passive signals alone — explicit user action required to reach Transactional
- Intent signals decay after 7 days of inactivity (user reverts to previous stage)

**Compliant User Routing:**

```typescript
async function routeUserToShowroom(
  userId: number,
  eventType: 'showroom.opt_in_clicked' | 'brand_zone_visited' | 'page_load'
): Promise<ShowroomRoutingDecision> {

  // Step 1: Hard gate — age assurance level 2+
  const identity = await identityRepo.get(userId)
  if (identity.ageAssuranceLevel < 2) {
    return 'age_revalidation_required'
  }

  // Step 2: Aggregate cohort data (no individual profiling)
  const cohort = await cohortMembershipRepo.findByUserId(userId)
  if (!cohort || cohort.communityReadinessScore < 0.75) {
    return 'engagement_only'
  }

  // Step 3: Route based on EXPLICIT USER ACTION only
  if (eventType === 'showroom.opt_in_clicked') {
    return 'tier_2_showroom'  // User explicitly clicked → consultative mode
  }

  if (eventType === 'brand_zone_visited') {
    const recentOffers = await showroomOfferRepo.countRecent(userId, '7 days')
    if (recentOffers >= 1) return 'no_offer_frequency_capped'

    const recentDismissals = await showroomOfferRepo.countDismissals(userId, '30 days')
    if (recentDismissals >= 2) return 'no_offer_user_opted_out'

    return 'suggestion_eligible'  // Can suggest via soft offer
  }

  return 'engagement_only'
}
```

**Anti-Patterns (Do Not Implement):**
- Do not score users based on zone visits alone
- Do not store behavioral profiles in analytics without consent
- Do not use conversation text for routing
- Do not expose individual scores to third parties

### 5.3 Community-to-Brand Affinity

The Living Graph is extended with a community-to-brand affinity dimension. This is the platform's core IP — it improves with every brand onboarded.

**How It Works:**

```
Community behavioral signals     Brand conversion data
(from Pillar 1)                  (from Pillar 2)
         │                                │
         ▼                                ▼
┌─────────────────────────────────────────────────┐
│          COMMUNITY-TO-BRAND MATCHING            │
│                                                  │
│  For each (community, brand) pair:               │
│                                                  │
│  Affinity = f(                                   │
│    behavioral_overlap,     ← mission types,      │
│                              zone patterns       │
│    conversion_history,     ← leads generated,    │
│                              quality scores      │
│    intent_signal_density,  ← brand zone opt-ins, │
│                              showroom entries    │
│    demographic_fit         ← age bands,          │
│                              community type      │
│  )                                               │
│                                                  │
│  Output: ranked list of communities per brand    │
│  + ranked list of brands per community           │
└─────────────────────────────────────────────────┘
```

**Bootstrapping (before conversion data exists):**

Use behavioral science research to establish minimum viable matching criteria:

| Signal | Research Basis | Minimum Threshold |
|--------|---------------|-------------------|
| Zone diversity | Spatial behavior literature: diverse exploration correlates with openness to new experiences | ≥4 zones in 7d |
| Brand zone opt-in | Explicit interest signal — strongest predictor of purchase intent in opt-in environments | ≥1 voluntary entry |
| Mission completion | Self-Determination Theory: competence satisfaction correlates with engagement quality | ≥60% completion rate |
| D7 retention | Standard SaaS engagement metric: 7-day retention predicts long-term value | Active on day 7 |
| Social reciprocity | Dunbar/social capital research: users with social bonds are more valuable leads | ≥2 lounge sessions in 7d |

As conversion data accumulates, the matching engine shifts from research-based thresholds to empirically validated weights.

### 5.4 Two-Level Feedback Loop

**Per-Brand Loop (the product brands pay for):**

```
Brand X's showroom performance
    │
    ├── Lead quality score trends
    ├── Conversion rates by community source
    ├── Bot interaction patterns (metadata)
    ├── Form completion/abandon rates
    │
    ▼
Optimization actions:
    ├── Layout A/B tests (7-day cycles)
    ├── Bot prompt adjustments
    ├── Form field optimization
    ├── Community routing priority changes
    │
    ▼
Outcome: "Brand X, your best leads come from Communities A and F.
          We've optimized your bot prompt — lead quality up 15%."
```

**Platform-Wide Loop (UC's competitive advantage):**

```
Aggregated data across ALL brands
    │
    ├── Which community profiles produce best leads (by brand type)
    ├── Which bot modes convert best (by intent stage)
    ├── Which offer types work (passive vs contextual vs community-led)
    ├── Which qualification criteria predict sales outcomes
    │
    ▼
Platform learning:
    ├── Improved community-to-brand matching for new brands
    ├── Better intent stage thresholds
    ├── Refined readiness score weights
    ├── Optimized default showroom configurations
    │
    ▼
Outcome: Each new brand onboarded starts with better defaults
         because the platform has learned from all previous brands.
```

### 5.5 Automated Optimization Rules

**Rule 1: Layout Adjustment**
```
IF avg_dwell_sec < 90 THEN
  Action: Increase visual hierarchy of CTA
  IF form_start / entrance < 0.1 THEN move CTA above fold
  A/B test: 7 days
  Success: avg_dwell_sec increases ≥15%
```

**Rule 2: Bot Prompt Adjustment**
```
IF intent_signal_strength_avg < 0.5 THEN
  Action: Change bot opening from generic to specific
  A/B test: 7 days
  Success: intent_signal_strength_avg increases
```

**Rule 3: Form Field Optimization**
```
IF form_abandon_rate > 0.4 THEN
  Action: Reduce required fields
  Test: 7 days
  Constraint: lead_quality_score must not decrease
```

**Rule 4: Sentiment-Aware Optimization**
```
IF bounce_rate > 0.4 THEN
  Action: simplify_layout (users confused)
ELSIF sentiment_score_avg >= 0.6 AND dwell_time_30_120s_pct > 0.5 THEN
  Action: no_change (users satisfied + efficient — don't add friction)
```

**Rule 5: Winner Selection**
```
FOR EACH showroom:
  winner = best variant over 14 days by lead_quality_score, then form_completion
  Deploy winner to all new sessions
```

---

## 6. Transition Mechanics

### 6.1 Three-Phase User Journey

**Phase 1: Signal Accumulation (Pillar 1)**

User in a high-readiness community naturally:
- Completes 5+ missions in 7 days
- Visits 4+ unique zones
- Enters Brand Zone (explicit opt-in)
- Stays in community ≥7 days
- Participates in social interactions

System state: Intent signals accumulate. No redirection.

**Phase 2: Readiness Detection (Nightly Job)**

Daily `intent-readiness-build` job detects user meets Qualitative cohort criteria:
- Community readiness score ≥ threshold
- Zone diversity ≥ 4
- Brand zone opt-ins ≥ 2 (repeated)
- D7 retention = true
- No safety flags

System state: User flagged as Qualitative. No UX change yet.

**Phase 3: Conversion Offer (Non-Coercive)**

User sees one of three opt-in triggers:

| Option | Friction | Trigger | UX |
|--------|---------|---------|-----|
| A: Passive Suggestion | Lowest | Completes mission or enters Brand Zone | Light notification: "You seem interested in [brand]. View showroom?" |
| B: Contextual Card | Medium | 3rd brand zone visit | Floating card at edge of screen (not center), dismissible |
| C: Community-Led | Social | Leader manual recommendation | "[Leader] thought you'd like to learn about [brand]" |

All options: one-click accept or dismiss, reversible, no persistence after dismissal.

### 6.2 Transition Guardrails

| Guardrail | Rule |
|-----------|------|
| Frequency Capping | Max 3 showroom offers per user per month |
| Dismissal Respect | If dismissed 2x, wait 14 days |
| Post-Visit Cooldown | After showroom visit, no suggestions for 30 days |
| Intent Decay | 7 days of inactivity reverts user to previous stage |
| Explicit Opt-Out | User can disable all conversion offers in profile settings |
| Minors Protection | No conversion offers or lead capture for age < 18 (hard enforcement) |
| Age Gate | Showroom access requires age assurance level 2+ |

---

## 7. Ethical Framework

### 7.1 Go/No-Go Decision for Any Feature

| Question | If "No" |
|----------|---------|
| **Transparency:** Can you explain this to a user and they understand it? | Don't deploy |
| **Reversibility:** Can they opt out or go back? | Don't deploy |
| **Autonomy:** Are they making a real choice? | Don't deploy |
| **Harm:** Could this damage community or wellbeing? | Don't deploy |

### 7.2 Forbidden Patterns

| Pattern | Why Forbidden | Enforcement |
|---------|---------------|-------------|
| Individual Profiling | Violates privacy, enables manipulation | All data aggregated to cohort level |
| Fake Social Proof | Damages trust | No simulated activity; AI bots labeled as AI |
| Urgency Manipulation | Exploits vulnerability | No countdown timers, limited-time offers, pressure language |
| Dark Patterns | LGPD violation | No difficult opt-out, disguised links, pre-checked boxes |
| Behavioral Addiction | Damages community health | Missions for competence, not compulsion; no streak punishments |
| Minors Targeting | Illegal + unethical | Age gate enforced; zero conversion offers for age < 18 |
| Conversation Logging | Privacy violation | Bot logs metadata only; transcripts never stored |

### 7.3 Monthly Compliance Audit (90 Minutes, 15-Point Checklist)

**Category 1: Consent & PII**
- No leads without consent
- No prohibited form fields
- Withdrawals processed within 7 days
- No leads from minors

**Category 2: Age Assurance**
- No underage access to showrooms
- Revalidation enforced for level 1 users

**Category 3: Data Minimization**
- No conversation text in analytics
- Bot logs contain metadata only
- Form fields match approved list

**Category 4: No Dark Patterns**
- No urgency language ("limited time", "only today")
- No streak punishment
- Frequency caps enforced (max 3/month)

**Category 5: User Feedback**
- Withdrawal rate < 10%
- Satisfaction score > 4/5
- "Felt pressured" < 5%

**Report:** Monthly audit report with PASS/WARN/FAIL per category. Legal review required.

---

## 8. Data Architecture (New Tables)

### 8.1 Showroom Domain

```sql
-- Branded virtual spaces (managed by UC)
CREATE TABLE showrooms (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,           -- Host community
    brand_name VARCHAR(200) NOT NULL,
    brand_type ENUM('high_consideration', 'low_consideration') NOT NULL,
    status ENUM('draft', 'active', 'paused', 'archived') DEFAULT 'draft',
    contract_start DATE NULL,
    contract_end DATE NULL,              -- NULL = ongoing
    active_variant_id VARCHAR(50) NULL,
    config JSON NULL,                    -- Brand-specific settings
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    INDEX idx_status (status),
    INDEX idx_community (community_id)
);

-- Showroom layout variants (for A/B testing)
CREATE TABLE showroom_layouts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    showroom_id INT NOT NULL,
    variant_id VARCHAR(50) NOT NULL,
    hero_headline TEXT,
    hero_subheadline TEXT,
    cta_button_text VARCHAR(100),
    cta_button_position ENUM('above_fold', 'middle', 'bottom'),
    form_fields JSON,                    -- [{name: "email", required: true}, ...]
    form_submit_text VARCHAR(100),
    form_privacy_text TEXT,
    bot_opening_prompt TEXT,
    bot_mode ENUM('educational', 'consultative', 'sales_oriented'),
    bot_allowed_hard_sell BOOLEAN DEFAULT FALSE,
    bot_fallback_text TEXT,
    data_collection_intensity ENUM('minimal', 'moderate', 'full'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (showroom_id) REFERENCES showrooms(id) ON DELETE CASCADE,
    UNIQUE KEY uk_variant (showroom_id, variant_id)
);

-- Lead capture records
CREATE TABLE showroom_leads (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    showroom_id INT NOT NULL,
    identity_id INT NOT NULL,
    name VARCHAR(200) NOT NULL,
    email VARCHAR(255) NOT NULL,
    company VARCHAR(200) NULL,
    specific_need TEXT NULL,
    optional_field TEXT NULL,
    lead_quality_score DECIMAL(3,1) NULL,    -- 1-10
    intent_stage ENUM('exploratory', 'qualitative', 'transactional') NOT NULL,
    consent_timestamp TIMESTAMP NOT NULL,
    consent_withdrawn_at TIMESTAMP NULL,
    exported_at TIMESTAMP NULL,               -- When sent to brand CRM
    retention_expires_at DATE NOT NULL,        -- 90 days (not exported) or 1 year (exported)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (showroom_id) REFERENCES showrooms(id) ON DELETE CASCADE,
    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    INDEX idx_showroom (showroom_id, created_at),
    INDEX idx_retention (retention_expires_at),
    INDEX idx_consent (consent_withdrawn_at)
);

-- Bot session metadata (no conversation content)
CREATE TABLE showroom_bot_sessions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    showroom_id INT NOT NULL,
    identity_id INT NOT NULL,
    variant_id VARCHAR(50) NOT NULL,
    bot_mode ENUM('educational', 'consultative', 'sales_oriented') NOT NULL,
    user_question_count INT DEFAULT 0,
    bot_answer_count INT DEFAULT 0,
    intent_keywords JSON NULL,                -- ["pricing", "timeline"]
    intent_signal_strength DECIMAL(3,2) NULL, -- 0.00-1.00
    user_sentiment_proxy ENUM('positive', 'neutral', 'negative') NULL,
    resolution_type ENUM('form_started', 'question_answered', 'timeout', 'user_exit', 'escalated') NULL,
    dwell_seconds INT DEFAULT 0,
    escalated_to_human BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (showroom_id) REFERENCES showrooms(id) ON DELETE CASCADE,
    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    INDEX idx_showroom_date (showroom_id, created_at)
);

-- A/B test tracking
CREATE TABLE showroom_bot_tests (
    id INT PRIMARY KEY AUTO_INCREMENT,
    showroom_id INT NOT NULL,
    test_variant_id VARCHAR(50) NOT NULL,
    control_variant_id VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NULL,
    status ENUM('active', 'completed', 'paused') DEFAULT 'active',
    winner_variant_id VARCHAR(50) NULL,
    winning_metric DECIMAL(8,2) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (showroom_id) REFERENCES showrooms(id) ON DELETE CASCADE
);

-- Daily showroom performance aggregates
CREATE TABLE showroom_performance_daily (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    period_start DATE NOT NULL,
    showroom_id INT NOT NULL,
    variant_id VARCHAR(50) NULL,
    -- Funnel
    entrance_count INT DEFAULT 0,
    form_start_count INT DEFAULT 0,
    form_completion_count INT DEFAULT 0,
    lead_capture_count INT DEFAULT 0,
    -- Engagement
    avg_dwell_sec DECIMAL(8,2) DEFAULT 0,
    question_count_total INT DEFAULT 0,
    question_count_avg DECIMAL(6,2) DEFAULT 0,
    bot_interaction_count_total INT DEFAULT 0,
    -- Behavior
    exit_type_distribution JSON NULL,
    intent_signal_strength_avg DECIMAL(5,2) DEFAULT 0,
    -- Quality
    avg_lead_quality_score DECIMAL(5,2) DEFAULT 0,
    intent_improvement_pct DECIMAL(6,2) DEFAULT 0,
    -- Friction
    form_abandon_rate DECIMAL(5,2) DEFAULT 0,
    bot_fallback_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    UNIQUE KEY uk_showroom_date (showroom_id, period_start, variant_id),
    FOREIGN KEY (showroom_id) REFERENCES showrooms(id) ON DELETE CASCADE
);

-- Intent readiness tracking (cohort-level, not individual)
CREATE TABLE cohort_membership (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,
    cohort_type VARCHAR(50) NOT NULL,        -- e.g., 'high-intent-explorers'
    intent_stage ENUM('latent', 'exploratory', 'qualitative', 'transactional') NOT NULL,
    count_users INT DEFAULT 0,
    avg_engagement_score DECIMAL(5,2) DEFAULT 0,
    zone_diversity DECIMAL(4,2) DEFAULT 0,
    brand_zone_optins INT DEFAULT 0,
    d7_retention BOOLEAN DEFAULT FALSE,
    computed_date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_cohort (community_id, cohort_type, computed_date),
    INDEX idx_date (computed_date)
);
```

### 8.2 Retention Rules (Showroom Domain)

| Data Type | Retention | Action at Expiry | Legal Basis |
|-----------|-----------|------------------|-------------|
| Leads (not exported) | 90 days | Hard delete | Legitimate interest |
| Leads (exported to brand) | 1 year | Hard delete | Contract |
| Withdrawn leads | 7-day soft delete window | Hard delete | LGPD right to withdrawal |
| Bot session metadata | 90 days | Anonymize | Legitimate interest |
| Showroom performance aggregates | 5 years | Archive | Business need |
| A/B test records | 1 year | Archive | Business need |

---

## 9. New Events, Jobs, and Aggregates

### 9.1 New Events (add to 07_EVENT_AND_ANALYTICS_SPEC)

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `showroom.entered` | User enters branded space | `{ showroom_id, variant_id, intent_stage, community_id }` | Server |
| `showroom.exited` | User leaves branded space | `{ showroom_id, dwell_seconds, exit_reason, community_id }` | Server |
| `showroom.bot_started` | Bot conversation begins | `{ showroom_id, bot_mode, variant_id }` | Server |
| `showroom.bot_ended` | Bot conversation ends | `{ showroom_id, resolution_type, dwell_seconds, question_count }` | Server |
| `showroom.form_started` | User begins lead form | `{ showroom_id }` | Frontend |
| `showroom.form_completed` | User submits lead form | `{ showroom_id, lead_quality_score }` | API |
| `showroom.form_abandoned` | User abandons lead form | `{ showroom_id, fields_completed, abandon_field }` | Frontend |
| `showroom.opt_in_clicked` | User clicks soft conversion offer | `{ offer_type, showroom_id, community_id }` | Frontend |
| `showroom.offer_shown` | Soft conversion offer displayed | `{ offer_type, showroom_id, community_id }` | Frontend |
| `showroom.offer_dismissed` | User dismisses conversion offer | `{ offer_type, showroom_id, community_id }` | Frontend |
| `showroom.escalated` | Bot escalates to human | `{ showroom_id, reason, dwell_seconds }` | Server |
| `brand_zone.entered` | User enters brand zone | `{ community_id, brand_zone_id }` | Server |
| `brand_zone.dwell` | User dwell in brand zone | `{ community_id, brand_zone_id, dwell_seconds }` | Server |

### 9.2 New Aggregates (add to 07_EVENT_AND_ANALYTICS_SPEC)

| Aggregate | Window | Dimensions | Metrics | Purpose |
|-----------|--------|------------|---------|---------|
| `intent_readiness_daily` | 1 day | `{community_id, cohort_type}` | `{mission_completion_rate, zone_diversity_avg, brand_optins, social_reciprocity_pct}` | Daily readiness assessment |
| `community_readiness_trends` | 7 days | `{community_id}` | `{readiness_score, trend_direction, stability}` | Weekly readiness forecasting |
| `showroom_performance_daily` | 1 day | `{showroom_id, variant_id}` | `{entrances, form_completions, lead_quality_avg, dwell_avg}` | Showroom optimization |
| `community_brand_affinity` | 28 days | `{community_id, showroom_id}` | `{lead_count, avg_quality, conversion_rate, affinity_score}` | Community-to-brand matching |

### 9.3 New Jobs (add to 08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING)

| Job Name | Queue | Schedule | Priority | Purpose |
|----------|-------|----------|----------|---------|
| `intent-readiness-build` | analytics | 06:00 BRT (09:00 UTC) | Medium | Compute cohort intent stages and community readiness scores |
| `showroom-performance-build` | analytics | 08:00 BRT (11:00 UTC) | Medium | Aggregate showroom performance metrics per variant |
| `showroom-variant-deployment` | analytics | 08:30 BRT (11:30 UTC) | Medium | Select and deploy A/B test winners |
| `lead-retention-cleanup` | compliance | 02:30 BRT (05:30 UTC) | Medium | Delete expired leads and anonymize bot sessions |

**Schedule Dependencies:**

```
00:01 ── daily-missions-assignment
00:30 ── streak-check
01:00 ── daily-feature-aggregation
02:00 ── data-anonymization
02:30 ── lead-retention-cleanup          ← NEW
03:00 ── backup-database + session-cleanup
04:15 ── community-graph-build
06:00 ── intent-readiness-build          ← NEW (depends on graph build)
08:00 ── showroom-performance-build      ← NEW (depends on intent readiness)
08:30 ── showroom-variant-deployment     ← NEW (depends on performance build)
```

### 9.4 New API Endpoint

```
GET /api/v1/conversion/community-readiness
Query: communityIds[], minReadinessScore=0.75

Response:
{
  "communities": [{
    "communityId": 5,
    "readinessScore": 0.82,
    "readinessTier": "green",
    "eligibilityReason": "High engagement, strong social health, opt-in brand exposure",
    "cohorts": [{
      "cohortType": "high-intent-explorers",
      "cohortSize": 45,
      "behavioralProfile": {
        "avg_zone_diversity": 4.2,
        "mission_completion_pct": 65,
        "brand_zone_optins": 0.91,
        "social_reciprocity_pct": 0.78
      }
    }],
    "recommendedAction": "ELIGIBLE_FOR_BRIDGE",
    "lastUpdated": "2026-01-29T06:15:00Z"
  }]
}
```

No individual-level data in response. Reason codes explain readiness (transparency).

---

## 10. Metrics & Success Criteria

### 10.1 Pillar 1: Social Intelligence

| Metric | Target | Measurement |
|--------|--------|-------------|
| Community Readiness Score (avg) | ≥ 0.70 | `intent_readiness_daily` job |
| Communities at Tier 1 (Phase 1) | ≥ 5 | Daily dashboard |
| Mission Completion Rate | ≥ 50% | `mission_activity_daily` |
| D7 Retention | ≥ 40% | Cohort calculation |
| Zone Diversity (avg) | ≥ 3.5 zones | `zone_dwell_daily` |

### 10.2 Pillar 2: Conversion

| Metric | Target | Measurement |
|--------|--------|-------------|
| Showroom Dwell Time (avg) | ≥ 90 sec | `showroom_performance_daily` |
| Lead Quality Score (avg) | ≥ 7/10 | Bot qualification output |
| Form Completion Rate | 20-40% | Forms completed / started |
| Bot Escalation Rate | < 15% | `showroom_bot_sessions` |

### 10.3 The Bridge: Transition Quality

| Metric | Target | Measurement |
|--------|--------|-------------|
| Withdrawal Rate | < 10% | Monthly audit |
| User Felt Pressured | < 5% | Quarterly survey |
| Minors Access Violations | 0 | Monthly audit |
| Community-to-Brand Match Improvement | +15% per quarter | Affinity score trend |

### 10.4 Bridge Efficacy Control Test

Before scaling branded spaces beyond pilot:

```
Population: All Qualitative users across Tier 1 communities
Sample: 1,000+ users
Duration: 14 days
Split: 90% treatment (receive soft offers), 10% control (no offers)

Success Criteria (GO):
  - Showroom entrance rate: +10% relative lift
  - Lead quality: No degradation (|delta| < 0.5 points)
  - Trust: < 5% report feeling pressured

Failure Criteria (NO-GO):
  - Entrance rate: no lift or negative
  - Lead quality drops > 1 point
  - Trust: > 5% report pressure
```

---

## 11. Risk Mitigation

| Risk | Mitigation |
|------|------------|
| False positives (exploratory users offered showroom) | Multi-signal confirmation: zone diversity + brand zone ≥2 (repeated) + explicit action |
| Bot optimizes for slowness | Sentiment scoring: distinguish bounce (confused) from efficient resolution (satisfied) |
| Human escalation SLA breached | 2h (high) / 24h (medium) response, dashboard alerts |
| Control test shows nudge backfires | Iteration: redesign soft offer, retest (2 weeks) |
| Compliance drift | Monthly 15-point audit runbook |
| Partner data sharing violations | DPA required, monthly audit, row-level security on leads |
| Cold-start (no conversion data) | Research-based thresholds bootstrapping; progressive weight tuning |
| Brand expects immediate ROI | Low initial pricing (per-lead or cheap monthly); set expectations on data accumulation period |

---

## 12. Implementation Phases

### Phase 1: Foundation (Weeks 1-2)

**System Docs (blocking):**
- [ ] Update 07, 08, 06, 12, 09 with showroom concepts
- [ ] Register all new events, jobs, tables, compliance rules

**Pillar 1 Extensions:**
- [ ] Add `intent_readiness_daily` aggregate
- [ ] Implement `intent-readiness-build` job (06:00 BRT)
- [ ] Create cohort classification logic
- [ ] Build community readiness API (`GET /api/v1/conversion/community-readiness`)
- [ ] Add readiness dashboard (admin view)

**Pillar 2 Skeleton:**
- [ ] Create showroom tables (showrooms, layouts, leads, bot_sessions)
- [ ] Design showroom landing page (static)
- [ ] Implement lead form (5 fields, explicit consent)
- [ ] Add age assurance level 2 gate

**Delivery:** Community readiness API + admin dashboard working for 5 test communities

### Phase 2: AI Bot (Weeks 3-4)

- [ ] Design bot conversation framework (opening prompts, branching logic, fallback rules)
- [ ] Implement bot session logging (metadata only, no text)
- [ ] Add sentiment scoring (from user actions, not NLP)
- [ ] Implement fallback SLA (2h high-priority escalation)
- [ ] Create `showroom-performance-build` job (08:00 BRT)

**Delivery:** AI bot can educate, qualify, offer form

### Phase 3: Feedback Loop (Weeks 5-6)

- [ ] Implement automated layout adjustment rules
- [ ] Build A/B testing infrastructure (`showroom_bot_tests` table)
- [ ] Add winner selection logic
- [ ] Create `showroom-variant-deployment` job (08:30 BRT)
- [ ] Setup monitoring + alert thresholds

**Delivery:** Showroom learns and optimizes daily

### Phase 4: Bridge & UX (Weeks 7-8)

- [ ] Implement 3 soft offer types (passive, contextual, community-led)
- [ ] Add frequency capping + decay (max 3/month, expires 7d)
- [ ] Build transition UI components
- [ ] Add minors gate (hard enforcement)
- [ ] Implement withdrawal path (7-day window, one-click)

**Delivery:** Full bridge mechanism ready for pilot

### Phase 5: Validation & Iteration (Ongoing)

- [ ] Run bridge efficacy control test (14 days, 1,000 users)
- [ ] Monthly compliance audits (15-check runbook)
- [ ] Quarterly readiness criteria refresh
- [ ] Onboard first paying brand
- [ ] Build community-to-brand affinity engine

**Go/No-Go Gates:**
- Phase 1 → 2: ≥5 communities at readiness score 0.75
- Phase 2 → 3: AI bot qualified 50+ leads with 7+ quality score
- Phase 3 → 4: Control test passes (10%+ entrance lift, no quality degradation)
- Phase 4 → Pilot: All 3 gates clear, audit checklist 100%, legal sign-off

---

## 13. Open Decisions

| Decision | Options | Status |
|----------|---------|--------|
| Lead quality thresholds per brand type | Define per high-consideration vs low-consideration | Pending |
| CRM integration for lead handoff | Direct API, CSV export, or webhook | Pending |
| Brand onboarding process | Manual setup vs self-service config tool | Manual first (Phase 1) |
| Pricing model finalization | Per-lead, monthly flat, hybrid | Per-lead for pilot |
| Geographic/vertical variation | Same showroom for all verticals vs vertical-specific templates | Pending |
| False positive tolerance | Current: <20% offer-but-exit rate | Validate in control test |

---

## 14. System Doc Updates Required

Before implementation begins, update these 5 documents:

| Document | Updates Needed |
|----------|---------------|
| **07_EVENT_AND_ANALYTICS_SPEC** | Add `showroom.*` and `brand_zone.*` events (Section 9.1), add aggregates (Section 9.2), add bot transcript prohibition to Section 8.3 |
| **08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING** | Add 4 new jobs (Section 9.3), update cron schedule registry, update dependency diagram |
| **06_DATA_ARCHITECTURE_AND_LIFECYCLE** | Add Showroom/Conversion data domain, all tables from Section 8.1, retention rules from Section 8.2 |
| **12_COMPLIANCE_AND_DATA_PROTECTION** | Add lead capture privacy review, consent mechanism + withdrawal path, partner data sharing rules (DPA), monthly audit runbook |
| **09_SECURITY_AND_AUTHORIZATION_SPEC** | Add `showroom_admin` and `partner_admin` RBAC roles, age-gated feature access (level 2+ for showrooms) |

Additional docs to update after this doc is registered:
- **01_SYSTEM_CANONICAL_INDEX** — add this document
- **19_EXECUTION_PLAN** — integrate Phase 1-5 roadmap
- **22_LAUNCH_EXECUTION_PLAN** — add showroom launch checklist
- **20_ADULT_BY_DESIGN_SPEC** — add showroom as Gate C application
- **CLAUDE.md** + **README.md** — update file counts

---

## 15. References

- **Full Validation Analysis:** `Two-Pillar-Strategy-FINAL-v2.md` (archived)
- **Integration Guide:** `Integration-Guide.md` (archived)
- **Original Strategy (v1):** `Two-Pillar SaaS Strategy Refinement.txt` (archived)
- **System Architecture:** 01-24 UC System Documentation
- **Behavioral Science:** `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md`
- **Compliance:** `12_COMPLIANCE_AND_DATA_PROTECTION.md`
- **Adult-by-Design:** `20_ADULT_BY_DESIGN_SPEC.md`

---

*This document defines the strategic architecture for UC's two-pillar SaaS model. All implementation must conform to these specifications and pass the three mandatory gates (Compliance & Privacy, Technical & Operational, Specification Consistency) before proceeding.*
