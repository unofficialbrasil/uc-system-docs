# Data Architecture and Lifecycle

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 1.2.0

---

## 1. Data Domains

### 1.1 Domain Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA DOMAINS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │    IDENTITY     │  │   COMMUNITY     │  │  GAMIFICATION   │             │
│  │                 │  │                 │  │                 │             │
│  │ • identities    │  │ • communities   │  │ • points        │             │
│  │ • profiles      │  │ • members       │  │ • streaks       │             │
│  │ • consents      │  │ • invites       │  │ • missions      │             │
│  │ • sessions      │  │ • roles         │  │ • rankings      │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │    UC WORLD     │  │   INTEGRATION   │  │    ANALYTICS    │             │
│  │                 │  │                 │  │                 │             │
│  │ • presence      │  │ • webhooks      │  │ • events        │             │
│  │ • zones         │  │ • oauth_tokens  │  │ • aggregates    │             │
│  │ • rooms         │  │ • api_keys      │  │ • reports       │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────┐            │
│  │                      LIVING GRAPH                            │            │
│  │                                                              │            │
│  │ • community_day_features  • community_graph_edges            │            │
│  │ • community_portal_assignments                               │            │
│  └─────────────────────────────────────────────────────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Domain Ownership

| Domain | Owner | Access Pattern | Sensitivity |
|--------|-------|----------------|-------------|
| Identity | Auth Service | Read-heavy | High (PII) |
| Community | API Service | Balanced | Medium |
| Gamification | API Service | Write-heavy | Low |
| UC World | World Server | Real-time | Low |
| Integration | Webhook Service | Write-heavy | Medium |
| Analytics | Analytics Service | Read-heavy | Low (aggregated) |
| Living Graph | Graph Build Job | Daily batch | Low (aggregated) |

---

## 2. Conceptual Schema

### 2.1 Identity Domain

```sql
-- Core user identity (PII)
CREATE TABLE identities (
    id INT PRIMARY KEY AUTO_INCREMENT,
    phone VARCHAR(20) UNIQUE NOT NULL,      -- PII: Hashed on deletion
    phone_verified_at TIMESTAMP NULL,
    email VARCHAR(255) NULL,                 -- PII: Deleted on account deletion

    -- Age verification fields (Adult-by-Design)
    date_of_birth DATE NULL,                 -- PII: Stored for age verification
    birth_year SMALLINT NULL,                -- Derived from DOB for analytics
    age_band ENUM('under_13', '13_17', '18_24', '25_34', '35_plus') NULL,
    age_assurance_level TINYINT DEFAULT 0,   -- 0=none, 1=self-declared, 2=revalidated, 3=verified
    age_declared_at TIMESTAMP NULL,          -- When user first declared DOB
    age_last_verified_at TIMESTAMP NULL,     -- When last revalidation occurred

    status ENUM('pending', 'active', 'suspended', 'deleted', 'banned') DEFAULT 'pending',
    suspended_at TIMESTAMP NULL,
    suspended_reason TEXT NULL,
    deleted_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_status (status),
    INDEX idx_phone (phone),
    INDEX idx_age (age_band, age_assurance_level)
);

-- User profile (separate from identity for privacy)
CREATE TABLE profiles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT UNIQUE NOT NULL,
    display_name VARCHAR(100) NOT NULL,      -- Anonymized on deletion
    avatar_url VARCHAR(500) NULL,            -- Deleted on account deletion
    bio TEXT NULL,                           -- Deleted on account deletion
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE
);

-- User consent tracking (LGPD)
CREATE TABLE consents (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    data_processing BOOLEAN DEFAULT TRUE,    -- Required for service
    marketing BOOLEAN DEFAULT FALSE,         -- Opt-in
    telemetry BOOLEAN DEFAULT TRUE,          -- Default on, opt-out
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_identity (identity_id)
);
```

### 2.2 Community Domain

```sql
-- Community records
CREATE TABLE communities (
    id INT PRIMARY KEY AUTO_INCREMENT,
    slug VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT NULL,
    owner_identity_id INT NOT NULL,
    visibility ENUM('public', 'private', 'invite_only') DEFAULT 'private',
    max_members INT DEFAULT 500,
    status ENUM('active', 'archived', 'deleted') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (owner_identity_id) REFERENCES identities(id),
    INDEX idx_status (status),
    INDEX idx_visibility (visibility)
);

-- Membership links
CREATE TABLE members (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    community_id INT NOT NULL,
    role ENUM('member', 'moderator', 'admin', 'owner') DEFAULT 'member',
    status ENUM('active', 'inactive', 'left', 'removed', 'invited', 'requested') DEFAULT 'active',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_activity_at TIMESTAMP NULL,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_member (identity_id, community_id),
    INDEX idx_community_status (community_id, status)
);

-- Invites
CREATE TABLE invites (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,
    code VARCHAR(50) UNIQUE NOT NULL,
    created_by INT NOT NULL,
    max_uses INT NULL,
    uses INT DEFAULT 0,
    expires_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    FOREIGN KEY (created_by) REFERENCES identities(id)
);
```

### 2.3 Gamification Domain

```sql
-- XP and levels
CREATE TABLE gamification_points (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    community_id INT NOT NULL,
    xp INT DEFAULT 0,
    week_xp INT DEFAULT 0,                   -- Reset weekly
    month_xp INT DEFAULT 0,                  -- Reset monthly
    level INT DEFAULT 1,
    frozen BOOLEAN DEFAULT FALSE,            -- True when suspended
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_points (identity_id, community_id),
    INDEX idx_community_xp (community_id, xp DESC)
);

-- Streaks
CREATE TABLE gamification_streaks (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    community_id INT NOT NULL,
    current_streak INT DEFAULT 0,
    best_streak INT DEFAULT 0,
    last_activity_date DATE NULL,
    streak_started_at DATE NULL,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_streak (identity_id, community_id)
);

-- Mission templates
CREATE TABLE mission_templates (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NULL,                   -- NULL = global template
    type VARCHAR(50) NOT NULL,
    description TEXT NOT NULL,
    base_xp INT NOT NULL,
    criteria JSON NULL,                      -- Machine-readable criteria
    active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE
);

-- Daily mission instances
CREATE TABLE daily_missions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,
    template_id INT NOT NULL,
    date DATE NOT NULL,
    xp_reward INT NOT NULL,
    expires_at TIMESTAMP NOT NULL,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    FOREIGN KEY (template_id) REFERENCES mission_templates(id),
    INDEX idx_community_date (community_id, date)
);

-- Completed missions
CREATE TABLE completed_missions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    mission_id INT NOT NULL,
    completed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    xp_awarded INT NOT NULL,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (mission_id) REFERENCES daily_missions(id) ON DELETE CASCADE,
    UNIQUE KEY uk_completion (identity_id, mission_id)
);
```

### 2.3A Trust & Safety Domain (Age Verification)

```sql
-- Possible minor detection cases
CREATE TABLE possible_minor_cases (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    detection_signals JSON NOT NULL,         -- Array of signal types
    risk_score TINYINT NOT NULL,             -- 0-100 risk score
    status ENUM('pending', 'confirmed_adult', 'confirmed_minor', 'inconclusive') DEFAULT 'pending',
    assigned_to INT NULL,                    -- T&S staff identity_id
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP NULL,
    resolution_action ENUM('none', 'age_recheck', 'suspend', 'ban') NULL,
    resolution_notes TEXT NULL,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_to) REFERENCES identities(id),
    INDEX idx_status (status, created_at),
    INDEX idx_identity (identity_id)
);

-- Age verification audit log
CREATE TABLE age_verification_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    action ENUM('gate_a_passed', 'gate_a_blocked', 'gate_b_passed', 'gate_b_failed', 'gate_c_blocked', 'level_upgraded') NOT NULL,
    previous_level TINYINT NULL,
    new_level TINYINT NULL,
    trigger_reason VARCHAR(100) NULL,
    metadata JSON NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    INDEX idx_identity_time (identity_id, created_at)
);
```

### 2.4 Integration Domain

```sql
-- Webhook event log
CREATE TABLE webhook_events (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    source ENUM('whatsapp', 'instagram') NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    payload JSON NOT NULL,
    processed BOOLEAN DEFAULT FALSE,
    processed_at TIMESTAMP NULL,
    error TEXT NULL,
    received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_processed (processed, received_at),
    INDEX idx_source_type (source, event_type)
);

-- OAuth tokens (for API integrations)
CREATE TABLE oauth_tokens (
    id INT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NOT NULL,
    provider ENUM('instagram', 'whatsapp') NOT NULL,
    access_token TEXT NOT NULL,              -- Encrypted
    refresh_token TEXT NULL,                 -- Encrypted
    expires_at TIMESTAMP NULL,
    scope TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_provider (identity_id, provider)
);
```

### 2.5 Analytics Domain

```sql
-- Event log (anonymized after 90 days)
CREATE TABLE analytics_events (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    identity_id INT NULL,                    -- NULL after anonymization
    anonymous_id VARCHAR(64) NULL,           -- Hash for de-duped aggregation
    event_name VARCHAR(100) NOT NULL,
    properties JSON NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_event_time (event_name, created_at),
    INDEX idx_identity (identity_id)
) PARTITION BY RANGE (UNIX_TIMESTAMP(created_at)) (
    -- Partitions by month for efficient deletion
);

-- Pre-computed aggregates
CREATE TABLE analytics_aggregates (
    id INT PRIMARY KEY AUTO_INCREMENT,
    aggregate_type VARCHAR(50) NOT NULL,
    dimensions JSON NOT NULL,                -- e.g., {"community_id": 1, "date": "2026-01-14"}
    metrics JSON NOT NULL,                   -- e.g., {"dau": 150, "missions_completed": 45}
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    computed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    UNIQUE KEY uk_aggregate (aggregate_type, period_start, dimensions(255))
);

-- Living Graph: Daily community feature aggregates
CREATE TABLE community_day_features (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,
    feature_date DATE NOT NULL,
    dau INT DEFAULT 0,                       -- Daily active users
    wau INT DEFAULT 0,                       -- Weekly active users (rolling 7d)
    missions_completed INT DEFAULT 0,
    portal_travels_in INT DEFAULT 0,         -- Visitors from other communities
    portal_travels_out INT DEFAULT 0,        -- Members traveling to other communities
    wa_messages INT DEFAULT 0,               -- WhatsApp messages (consent-gated)
    wa_reactions INT DEFAULT 0,              -- WhatsApp reactions (consent-gated)
    wa_replies INT DEFAULT 0,                -- WhatsApp replies (consent-gated)
    computed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_community_date (community_id, feature_date),
    INDEX idx_feature_date (feature_date)
);

-- Living Graph: Community similarity edges
CREATE TABLE community_graph_edges (
    id INT PRIMARY KEY AUTO_INCREMENT,
    source_community_id INT NOT NULL,
    target_community_id INT NOT NULL,
    edge_weight DECIMAL(10,6) NOT NULL,      -- Similarity score (0-1)
    edge_type ENUM('member_overlap', 'activity_pattern', 'content_similarity') NOT NULL,
    valid_from DATE NOT NULL,                -- Edge validity period
    valid_to DATE NOT NULL,
    computed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (source_community_id) REFERENCES communities(id) ON DELETE CASCADE,
    FOREIGN KEY (target_community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_edge (source_community_id, target_community_id, edge_type, valid_from),
    INDEX idx_source_weight (source_community_id, edge_weight DESC)
);

-- Living Graph: Portal assignments (which community appears at which portal)
CREATE TABLE community_portal_assignments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    community_id INT NOT NULL,               -- The community being assigned a portal
    portal_direction ENUM('N', 'NE', 'SE', 'S', 'SW', 'NW') NOT NULL,
    assigned_community_id INT NOT NULL,      -- The community that appears at this portal
    assignment_reason VARCHAR(100) NOT NULL, -- e.g., "member_overlap:0.45"
    valid_from DATE NOT NULL,
    valid_to DATE NOT NULL,
    computed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (community_id) REFERENCES communities(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_community_id) REFERENCES communities(id) ON DELETE CASCADE,
    UNIQUE KEY uk_portal (community_id, portal_direction, valid_from),
    INDEX idx_valid_dates (valid_from, valid_to)
);
```

---

## 3. Data Ownership and Stewardship

### 3.1 Ownership Matrix

| Table | Data Owner | Technical Owner | Business Owner |
|-------|------------|-----------------|----------------|
| identities | User | Engineering | Legal |
| profiles | User | Engineering | Product |
| consents | User | Engineering | Legal |
| communities | Admin | Engineering | Product |
| members | User/Admin | Engineering | Product |
| gamification_* | System | Engineering | Product |
| webhook_events | System | Engineering | Engineering |
| analytics_* | System | Engineering | Product |

### 3.2 Data Classification

| Classification | Tables | Access | Encryption |
|----------------|--------|--------|------------|
| PII (High) | identities, profiles | Auth required, logged | At rest |
| Sensitive (Medium) | oauth_tokens, consents | Auth required | At rest + in transit |
| Internal (Low) | gamification_*, members | Auth required | In transit |
| Public (None) | communities (public) | No auth for public | In transit |

---

## 4. Data Lineage

### 4.1 User Data Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Signup    │────►│  identities │────►│  profiles   │
│   Form      │     │             │     │             │
└─────────────┘     └──────┬──────┘     └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  consents   │
                   │             │
                   └─────────────┘
```

### 4.2 Gamification Data Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Webhook   │────►│  XP Award   │────►│   points    │
│   Event     │     │   Logic     │     │             │
└─────────────┘     └──────┬──────┘     └─────────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
       ┌───────────┐ ┌─────────┐ ┌───────────┐
       │  streaks  │ │ levels  │ │  rankings │
       │           │ │         │ │  (cache)  │
       └───────────┘ └─────────┘ └───────────┘
```

### 4.3 Analytics Data Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Raw Event  │────►│   events    │────►│ aggregates  │
│             │     │  (90 days)  │     │ (permanent) │
└─────────────┘     └──────┬──────┘     └─────────────┘
                          │
                          │ anonymize
                          ▼
                   ┌─────────────┐
                   │  Deleted/   │
                   │  Anonymized │
                   └─────────────┘
```

---

## 5. Retention Rules

### 5.1 Retention Schedule

| Data Type | Retention Period | Action at Expiry | Legal Basis |
|-----------|------------------|------------------|-------------|
| Active user PII | Account lifetime | N/A | Contract |
| Deleted user PII | Immediate | Anonymize | LGPD right to deletion |
| Session data | 7 days inactivity | Delete | Legitimate interest |
| Webhook events | 90 days | Delete | Legitimate interest |
| Analytics events | 90 days | Anonymize | Legitimate interest |
| Analytics aggregates | 5 years | Archive | Business need |
| Audit logs | 2 years | Archive | Legal requirement |
| Backups | 30 days | Delete | Disaster recovery |
| Age data (DOB) | Account lifetime + 1 year | Anonymize | Legitimate interest (safety) |
| Age verification logs | 2 years | Archive | Legal requirement |
| Possible minor cases | 5 years (resolved) | Archive | Legal requirement |
| ID verification docs | 30 days after verification | Delete | Minimization |

### 5.2 Retention Implementation

```sql
-- Daily retention job
CREATE EVENT daily_retention
ON SCHEDULE EVERY 1 DAY
STARTS '2026-01-15 03:00:00'
DO
BEGIN
    -- Delete expired sessions (handled by Redis TTL)

    -- Delete old webhook events
    DELETE FROM webhook_events
    WHERE received_at < DATE_SUB(NOW(), INTERVAL 90 DAY);

    -- Anonymize old analytics events
    UPDATE analytics_events
    SET identity_id = NULL,
        anonymous_id = SHA2(CONCAT(identity_id, created_at), 256)
    WHERE created_at < DATE_SUB(NOW(), INTERVAL 90 DAY)
      AND identity_id IS NOT NULL;

    -- Delete orphaned data (cascade should handle, but verify)
    DELETE FROM profiles
    WHERE identity_id NOT IN (SELECT id FROM identities);
END;
```

---

## 6. Deletion and Anonymization Logic

### 6.1 Account Deletion Procedure

```sql
DELIMITER //

CREATE PROCEDURE delete_user_account(IN p_identity_id INT)
BEGIN
    DECLARE v_phone VARCHAR(20);

    -- Start transaction
    START TRANSACTION;

    -- Get phone for hashing
    SELECT phone INTO v_phone FROM identities WHERE id = p_identity_id;

    -- 1. Anonymize identity
    UPDATE identities SET
        phone = SHA2(v_phone, 256),           -- Hash phone for uniqueness
        email = NULL,
        status = 'deleted',
        deleted_at = NOW()
    WHERE id = p_identity_id;

    -- 2. Anonymize profile
    UPDATE profiles SET
        display_name = 'Deleted User',
        avatar_url = NULL,
        bio = NULL
    WHERE identity_id = p_identity_id;

    -- 3. Delete consents
    DELETE FROM consents WHERE identity_id = p_identity_id;

    -- 4. Update memberships (preserve for community stats)
    UPDATE members SET
        status = 'deleted',
        identity_id = NULL  -- Unlink but keep record
    WHERE identity_id = p_identity_id;

    -- 5. Preserve gamification aggregates (anonymized)
    UPDATE gamification_points SET
        identity_id = NULL  -- Unlink
    WHERE identity_id = p_identity_id;

    -- 6. Delete OAuth tokens
    DELETE FROM oauth_tokens WHERE identity_id = p_identity_id;

    -- 7. Anonymize analytics
    UPDATE analytics_events SET
        identity_id = NULL,
        anonymous_id = SHA2(CONCAT(p_identity_id, created_at), 256)
    WHERE identity_id = p_identity_id;

    -- 8. Log deletion
    INSERT INTO audit_log (action, entity_type, entity_id, performed_at)
    VALUES ('account_deleted', 'identity', p_identity_id, NOW());

    COMMIT;
END //

DELIMITER ;
```

### 6.2 Data Anonymization Rules

| Field | Anonymization Method | Reversible |
|-------|---------------------|------------|
| phone | SHA-256 hash | No |
| email | Delete | No |
| display_name | Replace with "Deleted User" | No |
| avatar_url | Delete | No |
| bio | Delete | No |
| IP addresses | Delete or hash | No |
| identity_id (in related tables) | Set to NULL | No |

### 6.3 Cascade Behavior

| Parent Table | Child Table | On Delete |
|--------------|-------------|-----------|
| identities | profiles | CASCADE |
| identities | consents | CASCADE |
| identities | members | SET NULL (preserve stats) |
| identities | gamification_points | SET NULL (preserve stats) |
| identities | oauth_tokens | CASCADE |
| communities | members | CASCADE |
| communities | gamification_points | CASCADE |

---

## 7. Data Export (LGPD Right to Portability)

### 7.1 Export Format

```json
{
  "export_date": "2026-01-14T12:00:00Z",
  "user_id": 12345,
  "identity": {
    "phone": "+5511999999999",
    "email": "user@example.com",
    "created_at": "2025-06-01T10:00:00Z"
  },
  "profile": {
    "display_name": "João Silva",
    "bio": "Community enthusiast",
    "avatar_url": "https://..."
  },
  "consents": {
    "data_processing": true,
    "marketing": false,
    "telemetry": true,
    "updated_at": "2025-06-01T10:00:00Z"
  },
  "memberships": [
    {
      "community_name": "Lendas VIP",
      "role": "member",
      "joined_at": "2025-06-01T10:05:00Z"
    }
  ],
  "gamification": {
    "total_xp": 1250,
    "level": 5,
    "current_streak": 12,
    "best_streak": 30
  },
  "activity_summary": {
    "missions_completed": 45,
    "messages_sent": 230,
    "world_visits": 15
  }
}
```

### 7.2 Export Procedure

```
User requests export:
    │
    ├── Verify identity (re-authenticate)
    │
    ├── Queue export job
    │
    ├── Generate JSON file
    │   ├── Query all user tables
    │   ├── Aggregate statistics
    │   └── Format as JSON
    │
    ├── Encrypt file
    │
    ├── Store temporarily (24 hours)
    │
    ├── Notify user with download link
    │
    └── Delete after download or 24 hours
```

---

## 8. Data Integrity Constraints

### 8.1 Business Rules

| Rule | Implementation | Enforcement |
|------|----------------|-------------|
| One profile per identity | UNIQUE KEY | Database |
| One membership per identity/community | UNIQUE KEY | Database |
| Level matches XP threshold | Application logic | API |
| Streak calculated from activity | Application logic | Job |
| XP never decreases | Application logic | API |

### 8.2 Referential Integrity

All foreign keys use appropriate ON DELETE behavior:
- CASCADE: When parent deletion should delete children
- SET NULL: When children should be preserved anonymously
- RESTRICT: When parent deletion should be blocked

---

*This document defines how data flows through the system and its lifecycle. Implementation must conform to these specifications.*

<!-- Last Updated: 2026-01-18 - Added age fields to identities, Section 2.3A T&S Domain, age retention rules -->
