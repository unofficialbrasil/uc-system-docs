# STRATEGY REFINEMENT: SECTIONS TO ADD/REPLACE

**Based on:** Validation #1 & #2 Analysis  
**Date:** January 29, 2026  
**Action Required:** Integrate these sections into main strategy doc

---

## SECTION 1: ADD to Part 2.1 (After "Community-Level Readiness")

### Title: "Compliant Implementation: User Routing via Cohort + Explicit Actions"

**Replace pseudocode section with:**

```typescript
// ✅ COMPLIANT: This is what implementation must look like

async function routeUserToShowroom(
  userId: number,
  eventType: 'showroom.opt_in_clicked' | 'brand_zone_visited' | 'page_load'
): Promise<ShowroomRoutingDecision> {
  
  // Step 1: Verify age assurance (hard gate, hard enforcement)
  const identity = await identityRepo.get(userId)
  if (identity.ageAssuranceLevel < 2) {
    return 'age_revalidation_required'
  }
  
  // Step 2: Get AGGREGATE cohort data (no individual profiling)
  const cohort = await cohortMembershipRepo.findByUserId(userId)
  if (!cohort || !cohort.communityReadinessScore >= 0.75) {
    return 'engagement_only'
  }
  
  // Step 3: Route based on EXPLICIT USER ACTION (primary signal only)
  // NEVER infer intent from zone visits or mission completion alone
  
  if (eventType === 'showroom.opt_in_clicked') {
    // User explicitly clicked "Learn More" → primary signal
    return 'tier_2_showroom'  // Consultative mode
  }
  
  if (eventType === 'brand_zone_visited') {
    // User entered brand zone (check frequency cap + decay)
    const recentOffers = await showroomOfferRepo.countRecent(userId, '7 days')
    if (recentOffers >= 1) return 'no_offer_frequency_capped'
    
    const recentDismissals = await showroomOfferRepo.countDismissals(userId, '30 days')
    if (recentDismissals >= 2) return 'no_offer_user_opted_out'
    
    return 'suggestion_eligible'  // Can suggest via soft offer
  }
  
  return 'engagement_only'
}

// ❌ ANTI-PATTERNS (Don't Do This):
// - Don't score user based on zone visits alone
// - Don't store behavioral profile in analytics without consent
// - Don't use conversation text for routing
// - Don't expose individual scores to third parties
```

**Key Principle:** Cohort aggregates (community-level) + Explicit actions (individual-level) = Safe routing

---

## SECTION 2: REPLACE Section 3.1 "Intent Classification Framework"

### New Table: "Intent Stage Framework (Multi-Signal Confirmation)"

| Stage | Signals | Bot Mode | Showroom Tier | Offer Strategy | False Positive Risk |
|-------|---------|----------|---------------|-----------------|-------------------|
| **Latent** | Engaged in Pillar 1 only | N/A | None | No offers | Low (not interested) |
| **Exploratory** | Zone diversity ≥4 + brand zone ≥1 + D7 retention | Educational | Tier 1 (info) | Passive sugg. (1/mo) | **Medium** (just exploring) |
| **Qualitative** | Exploratory + brand zone ≥2 (REPEATED) + active 3d+ | Consultative | Tier 2 (discuss fit) | Contextual card | Low (confirmed interest) |
| **Transactional** | Qualitative + explicit action (click/question) | Sales-Oriented | Tier 3 (lead) | Optional form | **Very Low** (proof) |

**New SQL Rule:**
```sql
-- Stage progression must include BOTH aggregate signals AND explicit action
UPDATE cohortmembership
SET intent_stage = CASE
  -- Exploratory: High engagement + brand interest (aggregate signals)
  WHEN zone_diversity >= 4 
    AND brand_zone_optins >= 1 
    AND d7_retention = true
  THEN 'exploratory'
  
  -- Qualitative: Exploratory + REPEATED brand interest (multiple visits)
  WHEN intent_stage = 'exploratory'
    AND brand_zone_optins >= 2  -- NOT one-time visit
    AND days_since_last_activity <= 3
  THEN 'qualitative'
  
  -- Transactional: Qualitative + EXPLICIT USER ACTION (click, question, form)
  WHEN intent_stage = 'qualitative'
    AND (explicit_action_count >= 1 OR bot_question_count >= 1)
  THEN 'transactional'
  
  ELSE intent_stage
END
```

**Key Principle:** Don't auto-escalate based on passive signals. Require explicit action (click) to reach Transactional stage.

---

## SECTION 3: ADD after Bot Decision Tree (Section 3.2)

### Title: "Bot Session Quality Metrics (Metadata Only, No Conversation Content)"

```typescript
// ✅ DO LOG (metadata only):
logBotInteraction({
  sessionId: uuid(),
  botMode: 'educational' | 'consultative' | 'sales_oriented',
  
  // Interaction counts (no content)
  userQuestionCount: 5,
  botAnswerCount: 5,
  
  // Sentiment proxy (from user actions, not text analysis)
  userSentimentProxy: 'positive' | 'neutral' | 'negative',
    // Derived from: asked follow-up (+), said "thanks" (+), 
    //              said "not helpful" (-), exited early (-)
  
  // Intent signals (keywords only)
  intentKeywordsDetected: ['pricing', 'timeline', 'support'],
  intentScore: 0.72,  // 0-1 composite
  
  // Resolution quality
  resolutionType: 'form_started' | 'question_answered' | 'timeout' | 'user_exit',
  dwell_seconds: 240
})

// ❌ DO NOT LOG:
// - userMessage: "I need help with pricing for 100+ users"
// - botResponse: "Absolutely! Our enterprise plan..."
// - Full conversation transcript
// - Any identifiable info beyond keywords
```

**Use in Feedback Loop:**
```sql
-- Refined rule: Distinguish bounce from efficient resolution
IF bounce_rate > 0.4 THEN
  action = 'simplify_layout'  -- Users confused
ELSIF sentiment_score_avg >= 0.6 AND dwell_time_30_120s_pct > 0.5 THEN
  action = 'no_change'  -- Users satisfied + efficient (don't add friction)
  priority = 'maintain_variant'
END IF
```

---

## SECTION 4: ADD after Showroom Performance Metrics (Section 3.3)

### Title: "Lead Capture Form: Privacy by Design"

**Form HTML:**
```html
<form onsubmit="submitLead()">
  <!-- Field 1: Name (Required) -->
  <input type="text" name="name" required placeholder="Your full name" />
  
  <!-- Field 2: Email (Required) -->
  <input type="email" name="email" required placeholder="your@email.com" />
  
  <!-- Field 3: Company (Optional) -->
  <input type="text" name="company" placeholder="Your company (optional)" />
  
  <!-- Field 4: Specific Need (Optional) -->
  <textarea name="specific_need" placeholder="What's your main challenge?"></textarea>
  
  <!-- Field 5: EXPLICIT CONSENT (Mandatory) -->
  <label>
    <input type="checkbox" name="consent_contact" required />
    I agree that <strong>[Partner Name]</strong> may contact me about this offering,
    as described in their <a href="[partner_privacy_url]" target="_blank">privacy policy</a>.
  </label>
  <p class="legal">You can withdraw this consent anytime (link will be in confirmation email).</p>
  
  <!-- UC Privacy Notice -->
  <p class="legal">Your data is processed by UC per our 
    <a href="/privacy">privacy policy</a> and shared with [Partner].</p>
  
  <button type="submit">Get in Touch</button>
</form>
```

**Consent Checkbox Rules:**
- ❌ NOT pre-checked
- ❌ NOT hidden in fine print
- ✅ Mandatory to submit
- ✅ Links to partner privacy policy
- ✅ Clearly states who gets contact

**Confirmation Email (Immediate):**
```
Subject: Confirm your interest in [Partner]

Hi [Name],

Thanks! We've received your form.
[Partner] will contact you shortly.

---
MANAGE YOUR PREFERENCES:
- Withdraw consent: [one-click link, valid 7 days]
- Report issue: privacy@unofficial.com
---
```

**Automatic Cleanup (Retention):**
- Submitted but not exported: 90 days → hard delete
- Exported to partner CRM: 1 year → hard delete
- User withdrawal: 7-day window → soft delete
- Hard deletion logged to audit trail

---

## SECTION 5: ADD after User Experience Flow (Section 4.4)

### Title: "Bridge Efficacy Validation: Control Group Experiment"

```
Population: All Qualitative users across Tier 1 communities
Sample: 1,000+ users
Duration: 14 days
Randomization: user_id % 10 >= 9 → control, else → treatment

Treatment Group (90%): Receive soft conversion offers
- Passive: "You seem interested in [Partner]. View showroom?"
- Contextual: Floating card on 3rd brand zone visit
- Community-led: Leader recommendation

Control Group (10%): NO offers (baseline, hidden)
- Users can find showroom via direct link
- Measures organic discovery rate

Success Criteria:
✅ GO if:
  - Entrance rate: +10% relative lift (Y > X * 1.1)
  - Lead quality: Similar (|A - B| < 0.5 points)
  - Trust: < 5% felt pressured

❌ NO-GO if:
  - Y ≤ X (nudge backfires)
  - D > C + 10% (increases friction)
  - A > B by > 1 point (lower quality leads)
```

---

## SECTION 6: ADD after Readiness Score Weights (Section 7)

### Title: "Progressive Readiness Score Tuning (Phase 1 → 3)"

**Phase 1 (Safety-First, Weeks 1–2):**
```
ReadinessScore_Phase1 = 0.2(Engagement) 
                      + 0.2(Social) 
                      + 0.1(Intent) 
                      + 0.5(Safety)  ← Prioritize stable communities

Eligibility: Score >= 0.75 AND moderation_response < 48h
Communities: < 10 established communities
```

**Phase 2 (Balanced, Weeks 3–4):**
```
ReadinessScore_Phase2 = 0.25(Engagement) 
                      + 0.25(Social) 
                      + 0.2(Intent) 
                      + 0.3(Safety)

Eligibility: Score >= 0.70
Communities: Phase 1 winners + 20–30 new
```

**Phase 3+ (Standard):**
```
ReadinessScore_Final = 0.3(Engagement) 
                     + 0.3(Social) 
                     + 0.2(Intent) 
                     + 0.2(Safety)

Eligibility: Score >= 0.65
Communities: All Tier 1 (validated at scale)
```

---

## SECTION 7: EXPAND Section 5.3 "Audit & Compliance"

### Add: "Monthly Audit Runbook (90 Minutes, 15-Point Checklist)"

**Audit 1: Consent & PII**
```sql
-- Check: No leads without consent
SELECT COUNT(*) FROM showroomleads WHERE consent_timestamp IS NULL;

-- Check: No prohibited fields (phone, title, salary, etc.)
-- Check: Withdrawals processed (within 7 days)
-- Check: No leads to minors (age < 18)
```

**Audit 2: Age Assurance**
```sql
-- Check: No underage access to Tier 2+ showrooms
-- Check: Revalidation enforced for level 1 users
```

**Audit 3: Data Minimization**
```sql
-- Check: No conversation text in analytics
-- Check: Bot logs contain metadata only
-- Check: Form fields = approved list (name, email, company, need)
```

**Audit 4: No Dark Patterns**
```sql
-- Check: No urgency language ("limited time", "only today")
-- Check: No streak punishment
-- Check: Frequency caps enforced (max 3/month)
```

**Audit 5: User Feedback**
```sql
-- Check: Withdrawal rate < 10%
-- Check: Satisfaction score > 4/5
-- Check: "Felt pressured" < 5%
```

**Report Template:**
```
# Monthly Compliance Audit Report
Date: [Month] [Year]

Findings by Category:
1. Consent & PII: [PASS / WARN / FAIL]
2. Age Assurance: [PASS / WARN / FAIL]
3. Data Minimization: [PASS / WARN / FAIL]
4. Dark Patterns: [PASS / WARN / FAIL]
5. User Feedback: [PASS / WARN / FAIL]

Actions Required: [List]
Auditor: __________ Legal Review: __________
```

---

## SECTION 8: NEW SUBSECTION in Implementation Roadmap

### Add: "System Doc Updates (Before Phase 1)"

**Required updates to 5 docs:**

1. **07_EVENT_AND_ANALYTICS_SPEC**
   - Add events: showroom.*, brand_zone.*
   - Add aggregate: intentreadinessdaily
   - Rule: No conversation text logged

2. **08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING**
   - Add: intentReadinessBuild (0600 BRT)
   - Add: showroomPerformanceBuild (0800 BRT)
   - Add: showroomVariantDeployment (0830 BRT)

3. **06_DATA_ARCHITECTURE_AND_LIFECYCLE**
   - Add: showrooms, layouts, leads tables
   - Lifecycle: 90-day auto-delete (if not exported)

4. **12_COMPLIANCE_AND_DATA_PROTECTION**
   - Add: Lead capture privacy review
   - Add: Consent mechanism + withdrawal path
   - Add: Data sharing rules (partner handoff)

5. **09_SECURITY_AND_AUTHORIZATION_SPEC**
   - Add: Age-gated feature access (level 2+ for Tier 2 showrooms)
   - Add: RBAC (showroom_admin, partner_admin roles)

**Pre-Phase 1 Blocking:** All 5 docs must be updated before implementation begins.

---

## CRITICAL CHANGES SUMMARY

| Original Section | Refinement | Why |
|------------------|-----------|-----|
| User routing logic | Add explicit action requirement | Prevent individual profiling |
| Intent stages | Require multi-signal confirmation | Reduce false positives |
| Bot logging | Metadata only, no text | Compliance + privacy |
| Form design | Add consent checkbox + withdrawal | LGPD requirement |
| Feedback loop | Sentiment + distinguish bounce | Don't optimize for slowness |
| Phase 1 scoring | Raise safety weight to 50% | Safety-first pilot |
| Validation | Add control test | Prove bridge works |
| Audit | Add 15-point monthly runbook | Prevent compliance drift |

---

## QUICK ACTION ITEMS

**Today:**
- [ ] Approve 3 gates (Compliance, Technical, Spec Consistency)
- [ ] Share validation analysis with legal & compliance

**Days 1–3:**
- [ ] Update 07, 08, 06, 12, 09
- [ ] Review with engineering for feasibility

**Week 1:**
- [ ] Begin Phase 1 foundation (intentReadinessBuild job)
- [ ] Deploy readiness API + dashboard

**Weeks 3–8:**
- [ ] Phases 2–4 (AI bot, feedback loop, bridge UX)
- [ ] Control test (Phase 3, weeks 5–6)

**Go/No-Go Gates:**
- [ ] Phase 1 → 2: ≥5 communities at readiness score 0.75
- [ ] Phase 2 → 3: AI bot qualified 50+ leads with 7+ quality score
- [ ] Phase 3 → 4: Control test passes (10%+ entrance lift, no quality degradation)
- [ ] Phase 4 → Pilot: All 3 gates clear, audit checklist 100%, legal sign-off

---

**Document Location:** strategy_validation_analysis.md (full detail)  
**Strategy Update Status:** Ready for integration  
**Timeline Confidence:** High (8 weeks, 5 engineers)
