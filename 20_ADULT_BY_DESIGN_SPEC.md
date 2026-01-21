# Adult-by-Design (18+) Specification

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 1.0.0

---

## 1. Purpose

This document specifies the Adult-by-Design policy for Unofficial Communities. The platform operates as a de facto 18+ service through quiet enforcement mechanisms rather than public marketing as an "adult platform."

---

## 2. Strategic Positioning

### 2.1 External Messaging

| Context | Messaging |
|---------|-----------|
| Marketing | "High-signal, moderated, brand-safe communities" |
| App Store Listing | "Community platform for professionals" |
| Landing Page | No 18+ branding; focus on community quality |
| Terms of Service | Clear 18+ age requirement (legal compliance) |

### 2.2 Internal Policy

| Aspect | Approach |
|--------|----------|
| Age Gate | Quiet enforcement at signup and risk points |
| Content Policy | Adult-friendly but not adult-content |
| Trust & Safety | Robust moderation with minor-protection priority |
| Reporting | Escalated handling for potential minor detection |

### 2.3 Rationale

- **Brand Safety:** Avoid association with "adult content" platforms
- **User Trust:** Mature environment without explicit marketing
- **Partner Relations:** Easier brand partnerships with professional positioning
- **Compliance:** Meet legal requirements without over-signaling

---

## 3. Age Data Model

### 3.1 Identity Extension

```sql
-- Extend identities table
ALTER TABLE identities ADD COLUMN date_of_birth DATE NULL;
ALTER TABLE identities ADD COLUMN birth_year SMALLINT NULL;
ALTER TABLE identities ADD COLUMN age_band ENUM(
    'under_13',      -- Hard block
    '13_17',         -- Soft block
    '18_24',         -- Adult
    '25_34',         -- Adult
    '35_plus'        -- Adult
) NULL;
ALTER TABLE identities ADD COLUMN age_assurance_level TINYINT DEFAULT 0;
ALTER TABLE identities ADD COLUMN age_declared_at TIMESTAMP NULL;
ALTER TABLE identities ADD COLUMN age_last_verified_at TIMESTAMP NULL;
```

### 3.2 Age Assurance Levels

| Level | Name | Source | Trust Score |
|-------|------|--------|-------------|
| 0 | None | No data collected | 0% |
| 1 | Self-Declared | User-entered DOB | 60% |
| 2 | Risk-Based Revalidation | Prompted recollection | 75% |
| 3 | Selective Verification | ID verification service | 95% |

### 3.3 Age Band Logic

```typescript
function calculateAgeBand(dateOfBirth: Date): AgeBand {
  const age = calculateAge(dateOfBirth);

  if (age < 13) return 'under_13';      // COPPA zone
  if (age < 18) return '13_17';          // Minor zone
  if (age < 25) return '18_24';
  if (age < 35) return '25_34';
  return '35_plus';
}

function isAdult(ageBand: AgeBand): boolean {
  return !['under_13', '13_17'].includes(ageBand);
}
```

---

## 4. Enforcement Gates

### 4.1 Gate A: Account Activation

**Trigger:** End of onboarding flow, before dashboard access

**Flow:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    ONBOARDING COMPLETE                           │
│                    (Profile created)                             │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Gate A: Age Declaration      │
               │  "Enter your date of birth"   │
               │  [MM/DD/YYYY picker]          │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Calculate Age Band           │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Age >= 18      │              │  Age < 18       │
    │  Proceed        │              │  Block          │
    │  Level = 1      │              │  Show Message   │
    └─────────────────┘              └─────────────────┘
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Redirect to    │              │  "This platform │
    │  /dashboard     │              │   is for adults │
    │                 │              │   18 and over"  │
    └─────────────────┘              └─────────────────┘
```

**Implementation:**
- DOB field: Required date picker
- Validation: Server-side age calculation
- Storage: Store `date_of_birth`, `birth_year`, `age_band`, `age_assurance_level=1`
- Block UX: Friendly message, no shame language
- Retry: Allow correction within session if mistyped

### 4.2 Gate B: Risk-Based Recheck

**Triggers:**
- Account age > 30 days AND no activity in 60 days (dormant reactivation)
- Trust & Safety flag for "possible minor" behavior
- High-value feature access (creator monetization, direct messaging)
- Community admin role promotion

**Flow:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    RISK TRIGGER DETECTED                         │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Check current assurance      │
               │  level and age_band           │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Level >= 2     │              │  Level < 2      │
    │  Continue       │              │  Re-prompt      │
    └─────────────────┘              └─────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  "Please confirm your DOB"    │
                              │  [Pre-filled year]            │
                              │  "This helps us keep the      │
                              │   community safe"             │
                              └───────────────────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Compare to original DOB      │
                              └───────────────────────────────┘
                                              │
                       ┌──────────────────────┼──────────────────────┐
                       │                      │                      │
                       ▼                      ▼                      ▼
             ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
             │  Matches        │    │  Minor Mismatch │    │  Major Mismatch │
             │  Level = 2      │    │  (±1 year)      │    │  or Now Minor   │
             │  Continue       │    │  Accept, Log    │    │  Flag for T&S   │
             └─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Implementation:**
- Soft reconfirmation, not accusatory
- Log discrepancies for T&S review
- Major discrepancies: suspend pending review

### 4.3 Gate C: High-Risk Features

**Gated Features:**
| Feature | Min Level | Min Age Band | Additional Checks |
|---------|-----------|--------------|-------------------|
| Direct messaging | 1 | 18+ | Rate limited |
| Community creation | 2 | 18+ | Phone verified |
| Moderation role | 2 | 18+ | Account age > 30d |
| Monetization | 3 | 18+ | ID verification |
| UC World voice | 1 | 18+ | Consent per zone |

**Implementation:**
```typescript
async function canAccessFeature(
  identityId: number,
  feature: GatedFeature
): Promise<{ allowed: boolean; reason?: string }> {
  const user = await getIdentity(identityId);
  const config = FEATURE_GATES[feature];

  if (!user.age_band || !isAdult(user.age_band)) {
    return { allowed: false, reason: 'age_requirement_not_met' };
  }

  if (user.age_assurance_level < config.minLevel) {
    return { allowed: false, reason: 'verification_required' };
  }

  if (config.additionalChecks) {
    const passed = await runAdditionalChecks(user, config.additionalChecks);
    if (!passed) {
      return { allowed: false, reason: 'additional_verification_failed' };
    }
  }

  return { allowed: true };
}
```

---

## 5. Trust & Safety Integration

### 5.1 Possible Minor Detection

**Signals:**
| Signal | Weight | Action |
|--------|--------|--------|
| Self-reported age in chat | High | Immediate review queue |
| Username patterns (e.g., name2012) | Medium | Flag for review |
| Reported by other users | High | Immediate review queue |
| Behavioral patterns (school hours activity) | Low | Monitor only |
| Language analysis (juvenile vocabulary) | Low | Monitor only |

### 5.2 Minor Detection Queue

```typescript
interface PossibleMinorCase {
  case_id: string;
  identity_id: number;
  detection_signals: DetectionSignal[];
  risk_score: number;           // 0-100
  status: 'pending' | 'confirmed_adult' | 'confirmed_minor' | 'inconclusive';
  assigned_to?: number;         // T&S staff
  created_at: Date;
  resolved_at?: Date;
  resolution_action?: 'none' | 'age_recheck' | 'suspend' | 'ban';
}
```

### 5.3 Resolution Actions

| Outcome | Action |
|---------|--------|
| Confirmed Adult | Clear flag, update assurance level to 2 |
| Confirmed Minor | Suspend immediately, delete PII, notify |
| Inconclusive | Require Gate B recheck on next login |

---

## 6. Community Standards Upgrade

### 6.1 Content Policy Additions

Add to Terms of Service:
```markdown
## Age Requirements

1. This platform is intended for users aged 18 and older.
2. By creating an account, you confirm that you are at least 18 years old.
3. We may request age verification at any time.
4. If we determine you are under 18, your account will be suspended.

## Content Guidelines

1. Adult language and themes are permitted within community guidelines.
2. Explicit sexual content is prohibited.
3. Content that could harm or exploit minors is strictly prohibited.
```

### 6.2 Moderation Guidelines

| Category | Policy |
|----------|--------|
| Age-related reports | Priority review (< 4 hours) |
| Minor identification | Immediate suspension pending review |
| Age falsification | Permanent ban after confirmation |
| Content for minors | Immediate removal and creator warning |

---

## 7. Rollout Plan

### 7.1 Feature Flags

| Flag | Default | Description |
|------|---------|-------------|
| `FEATURE_AGE_GATE_SIGNUP` | true | Enable Gate A at signup |
| `FEATURE_AGE_GATE_RISK` | false | Enable Gate B risk checks |
| `FEATURE_AGE_GATE_FEATURES` | false | Enable Gate C feature gating |
| `FEATURE_MINOR_DETECTION` | false | Enable automated minor detection |

### 7.2 Phased Rollout

| Phase | Duration | Scope | Gates Enabled |
|-------|----------|-------|---------------|
| 1 - Foundation | Week 1-2 | New signups only | Gate A |
| 2 - Validation | Week 3-4 | + Risk triggers | Gate A, B |
| 3 - Feature Gating | Week 5-6 | + High-risk features | Gate A, B, C |
| 4 - Detection | Week 7+ | + Automated signals | All gates + detection |

### 7.3 Migration for Existing Users

```sql
-- Phase 1: Set existing users to level 0 (no data)
UPDATE identities
SET age_assurance_level = 0,
    age_band = NULL
WHERE date_of_birth IS NULL;

-- Phase 2: Prompt existing users at next login
-- Implemented via Gate B trigger: "existing_user_migration"
```

---

## 8. Privacy & Compliance

### 8.1 LGPD/GDPR Considerations

| Data | Legal Basis | Retention |
|------|-------------|-----------|
| date_of_birth | Legitimate interest (safety) | Account lifetime + 1 year |
| birth_year | Legitimate interest (safety) | Account lifetime + 1 year |
| age_band | Legitimate interest (safety) | Account lifetime |
| assurance_level | Legitimate interest (safety) | Account lifetime |
| verification_docs | Contract (if provided) | 30 days after verification |

### 8.2 Data Subject Rights

| Right | Implementation |
|-------|----------------|
| Access | Include age data in export |
| Deletion | Delete with account (anonymize for analytics) |
| Rectification | Allow DOB correction with T&S review |
| Objection | Cannot opt out of age verification (legal requirement) |

### 8.3 Consent Text

At Gate A:
```
"To use this platform, please confirm your date of birth.
We collect this to ensure our community remains a safe,
adult-only space. Your date of birth is stored securely
and used only for age verification purposes."
```

---

## 9. Analytics Events

### 9.1 New Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `age.gate_presented` | Gate shown to user | `{ gate: 'A'|'B'|'C', trigger }` |
| `age.declared` | User submits DOB | `{ age_band, assurance_level }` |
| `age.blocked` | User blocked (minor) | `{ age_band }` |
| `age.revalidated` | Gate B completed | `{ matched, discrepancy_days }` |
| `age.feature_blocked` | Gate C denied access | `{ feature, reason }` |
| `age.minor_flagged` | T&S flag created | `{ signal_type, risk_score }` |
| `age.appeal_opened` | User appeals block | `{ case_id }` |
| `age.appeal_resolved` | Appeal completed | `{ case_id, outcome }` |

### 9.2 Dashboard Metrics

| Metric | Calculation |
|--------|-------------|
| Age Declaration Rate | Users with level >= 1 / Total users |
| Minor Block Rate | Blocked at Gate A / Gate A presentations |
| Revalidation Pass Rate | Gate B passed / Gate B triggered |
| False Positive Rate | Appealed & approved / Total blocks |

---

## 10. Success Metrics

### 10.1 Primary KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| Age Gate Completion Rate | > 98% | Signups completing Gate A |
| Under-18 Block Rate | < 5% | Users blocked as minors |
| False Positive Rate | < 0.1% | Appeals approved / Total blocks |
| T&S Queue Time | < 4 hours | Avg time to resolve minor flags |

### 10.2 Secondary KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| Signup Abandonment | < 10% increase | Pre vs post Gate A |
| User Complaints | < 0.1% | Age-related support tickets |
| Moderation Efficiency | > 90% | Correct minor detection |
| Revalidation Compliance | > 95% | Users completing Gate B |

---

## 11. Communication Guidelines

### 11.1 Internal Documentation

Use: "18+ enforcement", "age assurance", "adult-by-design"

### 11.2 External Communications

Avoid: "adult platform", "18+ only", "age restricted"
Use: "professional community", "mature discussion", "high-signal space"

### 11.3 Support Scripts

**User asks about age requirement:**
> "Our platform is designed for adults aged 18 and older. This helps us
> maintain a professional, high-quality environment for meaningful
> community discussions."

**User blocked as minor:**
> "Based on the information provided, we're unable to verify that you
> meet our age requirements. We're committed to maintaining a safe
> environment for all our users. If you believe this is an error,
> please contact support with verification."

---

## 12. Background Jobs

### 12.1 Age-Related Jobs

| Job | Queue | Schedule | Description |
|-----|-------|----------|-------------|
| `age-revalidation-prompt` | age-queue | Daily 08:00 | Prompt dormant users for Gate B |
| `minor-detection-scan` | age-queue | Daily 04:00 | Analyze behavioral signals |
| `age-data-cleanup` | maintenance | Weekly | Purge expired verification docs |

### 12.2 Job Definitions

```typescript
// age-revalidation-prompt
{
  name: 'age-revalidation-prompt',
  queue: 'age-queue',
  schedule: '0 8 * * *',  // 08:00 daily
  handler: async () => {
    // Find users needing revalidation
    const users = await findUsersForRevalidation({
      accountAgeDays: 30,
      inactiveDays: 60,
      assuranceLevel: 1
    });

    for (const user of users) {
      await scheduleGateBPrompt(user.id);
    }
  }
}
```

---

## 13. Implementation Checklist

### Phase 1: Foundation
- [ ] Add age columns to identities table (migration)
- [ ] Create Gate A UI component
- [ ] Implement age calculation service
- [ ] Add `FEATURE_AGE_GATE_SIGNUP` flag
- [ ] Update onboarding flow to include Gate A
- [ ] Add analytics events for age gates

### Phase 2: Validation
- [ ] Implement Gate B risk detection triggers
- [ ] Create revalidation UI component
- [ ] Add T&S minor detection queue
- [ ] Implement minor flag handling
- [ ] Add `FEATURE_AGE_GATE_RISK` flag

### Phase 3: Feature Gating
- [ ] Implement feature gating service
- [ ] Add Gate C to gated features
- [ ] Update feature access checks
- [ ] Add `FEATURE_AGE_GATE_FEATURES` flag

### Phase 4: Detection
- [ ] Implement behavioral signal detection
- [ ] Create minor detection ML model
- [ ] Add automated flagging pipeline
- [ ] Add `FEATURE_MINOR_DETECTION` flag

### Documentation Updates
- [ ] Update Terms of Service
- [ ] Update Privacy Policy
- [ ] Create internal moderation guidelines
- [ ] Update API documentation

---

## Appendix A: Database Migration

```sql
-- Migration: add_age_columns_to_identities

ALTER TABLE identities
ADD COLUMN date_of_birth DATE NULL AFTER email,
ADD COLUMN birth_year SMALLINT NULL AFTER date_of_birth,
ADD COLUMN age_band ENUM('under_13', '13_17', '18_24', '25_34', '35_plus') NULL AFTER birth_year,
ADD COLUMN age_assurance_level TINYINT DEFAULT 0 AFTER age_band,
ADD COLUMN age_declared_at TIMESTAMP NULL AFTER age_assurance_level,
ADD COLUMN age_last_verified_at TIMESTAMP NULL AFTER age_declared_at;

-- Index for age-related queries
CREATE INDEX idx_identities_age ON identities (age_band, age_assurance_level);

-- Create T&S queue table
CREATE TABLE possible_minor_cases (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    detection_signals JSON NOT NULL,
    risk_score TINYINT NOT NULL,
    status ENUM('pending', 'confirmed_adult', 'confirmed_minor', 'inconclusive') DEFAULT 'pending',
    assigned_to INT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP NULL,
    resolution_action ENUM('none', 'age_recheck', 'suspend', 'ban') NULL,
    resolution_notes TEXT NULL,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_to) REFERENCES identities(id),
    INDEX idx_status (status, created_at),
    INDEX idx_identity (identity_id)
);
```

---

## Appendix B: API Endpoints

```yaml
# Age Gate Endpoints

POST /api/v1/age/declare:
  description: Submit date of birth (Gate A)
  body:
    date_of_birth: string (YYYY-MM-DD)
  response:
    success: boolean
    age_band: string
    assurance_level: number

POST /api/v1/age/revalidate:
  description: Confirm date of birth (Gate B)
  body:
    date_of_birth: string (YYYY-MM-DD)
  response:
    success: boolean
    matched: boolean
    new_assurance_level: number

GET /api/v1/age/status:
  description: Get user's age verification status
  response:
    age_band: string | null
    assurance_level: number
    requires_action: boolean
    action_type: 'gate_a' | 'gate_b' | 'gate_c' | null

POST /api/v1/age/appeal:
  description: Appeal age block decision
  body:
    reason: string
  response:
    case_id: string
    estimated_review_time: string
```

---

*This document is the authoritative specification for the Adult-by-Design policy. Implementation must conform to these specifications.*

<!-- Last Reviewed: 2026-01-20 - No updates needed -->
