# Event and Analytics Specification

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 1.2.0

---

## 1. Event Taxonomy

### 1.1 Event Naming Convention

```
<domain>.<entity>.<action>

Examples:
- user.session.created
- gamification.mission.completed
- world.zone.entered
- community.member.joined
```

### 1.2 Event Categories

| Category | Purpose | Retention | PII |
|----------|---------|-----------|-----|
| System | Infrastructure health | 30 days | No |
| Security | Auth, access control | 2 years | Yes (logged) |
| Business | User actions, conversions | 90 days (raw), 5 years (aggregate) | Anonymized |
| Product | Feature usage, UX | 90 days | Anonymized |

---

## 2. Event Catalog

### 2.1 Authentication Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `user.signup.started` | User begins signup | `{ phone_prefix }` | Frontend |
| `user.signup.otp_sent` | OTP sent successfully | `{ delivery_method }` | API |
| `user.signup.otp_verified` | OTP verified | `{ attempts_used }` | API |
| `user.signup.completed` | Profile created | `{ has_avatar, community_id }` | API |
| `user.login.started` | User begins login | `{ phone_prefix }` | Frontend |
| `user.login.completed` | Login successful | `{ session_id_hash }` | API |
| `user.login.failed` | Login failed | `{ reason }` | API |
| `user.logout.completed` | User logged out | `{ session_duration_seconds }` | API |
| `user.session.expired` | Session auto-expired | `{ inactive_days }` | Job |

### 2.2 Gamification Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `gamification.xp.awarded` | XP granted | `{ amount, source, community_id }` | API |
| `gamification.level.up` | Level increased | `{ old_level, new_level, community_id }` | API |
| `gamification.mission.assigned` | Daily missions set | `{ mission_count, community_id }` | Job |
| `gamification.mission.completed` | Mission done | `{ mission_type, xp_awarded, community_id }` | API |
| `gamification.streak.started` | First activity | `{ community_id }` | API |
| `gamification.streak.continued` | Consecutive day | `{ streak_day, community_id }` | API |
| `gamification.streak.broken` | Missed day | `{ previous_streak, community_id }` | Job |
| `gamification.streak.milestone` | 7/30/100 days | `{ milestone, bonus_xp, community_id }` | API |
| `gamification.ranking.updated` | Weekly reset | `{ community_id, top_3_changed }` | Job |

### 2.3 Community Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `community.created` | New community | `{ visibility, owner_id_hash }` | API |
| `community.member.joined` | User joined | `{ method, community_id }` | API |
| `community.member.left` | User left | `{ reason, days_member, community_id }` | API |
| `community.member.removed` | Admin removed | `{ by_role, community_id }` | API |
| `community.invite.created` | Invite generated | `{ community_id }` | API |
| `community.invite.used` | Invite redeemed | `{ community_id }` | API |

### 2.4 UC World Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `world.session.started` | Connected to room | `{ community_id, room_population }` | Server |
| `world.session.ended` | Disconnected | `{ duration_seconds, community_id }` | Server |
| `world.zone.entered` | Entered zone | `{ zone_type, from_zone, community_id }` | Server |
| `world.zone.exited` | Left zone | `{ zone_type, duration_seconds, community_id }` | Server |
| `world.portal.activated` | User enters portal zone | `{ portal_direction, destination_community_id, destination_name, reason_codes }` | Server |
| `world.portal.traveled` | Portal travel completed | `{ source_community_id, destination_community_id, portal_direction, visitor_mode }` | Server |
| `world.portal.card.viewed` | Portal card UI opened | `{ portal_direction, destination_community_id, reason_codes, confidence }` | Client |
| `world.portal.why.clicked` | User expands "Why" explanation | `{ portal_direction, destination_community_id }` | Client |
| `world.portal.hide.clicked` | User hides a portal neighbor | `{ portal_direction, destination_community_id, hidden_neighbor_id }` | Client |
| `world.portal.report.submitted` | User reports portal issue | `{ source_community_id, destination_community_id, reason, report_id }` | Server |
| `world.visitor.timeout` | Visitor session expired | `{ visited_community_id, origin_community_id, duration_seconds }` | Server |
| `world.visitor.zone_blocked` | Visitor tried to enter restricted zone | `{ visited_community_id, attempted_zone, allowed_zones }` | Server |
| `world.presence.updated` | Position changed | `{ zone_type, community_id }` | Server (sampled) |

**Canonical Zone IDs** (defined in `uc-world/shared/src/zones.ts`):

| Zone ID | Display Name | Type |
|---------|--------------|------|
| `central_hub` | Central Hub | Interior |
| `workshop` | Workshop | Interior |
| `social_east` | Lounge East | Interior |
| `social_west` | Lounge West | Interior |
| `brand_zone` | Partner Space | Interior |
| `quiet_refuge` | Quiet Room | Interior |
| `onboarding` | Welcome | Interior |
| `portal_ne` | Northeast Portal | Portal |
| `portal_e` | East Portal | Portal |
| `portal_se` | Southeast Portal | Portal |
| `portal_sw` | Southwest Portal | Portal |
| `portal_w` | West Portal | Portal |
| `portal_nw` | Northwest Portal | Portal |

### 2.5 WhatsApp Activity Events (Living Graph)

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `whatsapp.message.sent` | User sends message to group | `{ community_id, identity_id, timestamp_bucket }` | Webhooks |
| `whatsapp.reaction.added` | User reacts to message | `{ community_id, identity_id, timestamp_bucket }` | Webhooks |
| `whatsapp.reply.sent` | User replies to message | `{ community_id, identity_id, timestamp_bucket }` | Webhooks |
| `whatsapp.member.joined` | User joins WhatsApp group | `{ community_id, identity_id, joined_via }` | Webhooks |
| `whatsapp.member.left` | User leaves WhatsApp group | `{ community_id, identity_id, left_reason }` | Webhooks |

> **Privacy Rules:**
> - Events are **counts-only** - no message content is ever stored
> - Events are stored in aggregate form (daily buckets) for graph computation
> - Collection requires explicit consent (`consents.whatsapp_activity = true`)
> - `identity_id` is anonymized after 90 days per retention policy

### 2.6 Age Verification Events (Adult-by-Design)

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `age.gate_presented` | Gate shown to user | `{ gate: 'A'|'B'|'C', trigger }` | Frontend |
| `age.declared` | User submits DOB (Gate A) | `{ age_band, assurance_level }` | API |
| `age.blocked` | User blocked (under 18) | `{ age_band, gate: 'A'|'B' }` | API |
| `age.revalidated` | Gate B completed | `{ matched, discrepancy_days, new_level }` | API |
| `age.feature_blocked` | Gate C denied access | `{ feature, reason, required_level }` | API |
| `age.minor_flagged` | T&S case created | `{ signal_type, risk_score }` | API |
| `age.case_resolved` | T&S case resolved | `{ case_id, outcome, action_taken }` | API |
| `age.appeal_opened` | User appeals block | `{ case_id }` | API |
| `age.appeal_resolved` | Appeal completed | `{ case_id, outcome }` | API |
| `age.level_upgraded` | Assurance level increased | `{ old_level, new_level, reason }` | API |

### 2.7 Webhook Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `webhook.received` | Incoming webhook | `{ source, event_type }` | Webhooks |
| `webhook.processed` | Successfully handled | `{ source, event_type, duration_ms }` | Webhooks |
| `webhook.failed` | Processing failed | `{ source, event_type, error_code }` | Webhooks |
| `webhook.retried` | Retry attempt | `{ source, attempt_number }` | Webhooks |
| `webhook.dead_lettered` | Max retries exceeded | `{ source, event_type }` | Webhooks |

### 2.8 System Events

| Event Name | Trigger | Properties | Source |
|------------|---------|------------|--------|
| `system.startup` | Service started | `{ service, version }` | All |
| `system.shutdown` | Service stopping | `{ service, uptime_seconds }` | All |
| `system.health.degraded` | Health check failed | `{ service, check_name }` | All |
| `system.health.recovered` | Health restored | `{ service, downtime_seconds }` | All |
| `system.job.started` | Background job began | `{ job_name }` | Jobs |
| `system.job.completed` | Background job done | `{ job_name, duration_ms }` | Jobs |
| `system.job.failed` | Background job error | `{ job_name, error }` | Jobs |

---

## 3. Event Schema

### 3.1 Base Event Structure

```typescript
interface BaseEvent {
  event_id: string;           // UUID v4
  event_name: string;         // From catalog (e.g., "whatsapp.message.sent")
  event_version: string;      // Schema version (e.g., "1.0.0")
  timestamp: string;          // ISO 8601 UTC
  source: 'frontend' | 'api' | 'server' | 'job' | 'webhooks';
  environment: 'production' | 'staging' | 'development';

  // Identity (anonymizable)
  identity_id?: number;       // NULL after anonymization or if consent denied
  anonymous_id?: string;      // SHA-256 hash for aggregation

  // Context
  session_id?: string;        // Hashed
  community_id?: number;
  request_id?: string;        // For tracing

  // Event-specific data
  properties: Record<string, unknown>;

  // Consent tracking (added for Living Graph)
  consent_flags?: {
    telemetry: boolean;
    whatsapp_activity?: boolean;
  };
}
```

### 3.2 Required Fields by Event Type

| Event Domain | Required Fields | Consent Required |
|--------------|-----------------|------------------|
| `user.*` | `identity_id`, `event_version` | `telemetry` |
| `gamification.*` | `identity_id`, `community_id`, `event_version` | `telemetry` |
| `community.*` | `community_id`, `event_version` | `telemetry` |
| `world.*` | `community_id`, `event_version` | `telemetry` |
| `whatsapp.*` | `community_id`, `identity_id`, `event_version` | `whatsapp_activity` |
| `age.*` | `identity_id`, `event_version` | None (required for safety) |
| `webhook.*` | `event_version` | None (system event) |
| `system.*` | `event_version` | None (system event) |

### 3.3 Example Events

```json
// Signup completed
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "event_name": "user.signup.completed",
  "timestamp": "2026-01-14T12:30:00.000Z",
  "source": "api",
  "environment": "production",
  "identity_id": 12345,
  "properties": {
    "has_avatar": true,
    "community_id": 1,
    "signup_duration_seconds": 180
  }
}

// Mission completed
{
  "event_id": "550e8400-e29b-41d4-a716-446655440001",
  "event_name": "gamification.mission.completed",
  "timestamp": "2026-01-14T14:00:00.000Z",
  "source": "api",
  "environment": "production",
  "identity_id": 12345,
  "community_id": 1,
  "properties": {
    "mission_type": "send_messages",
    "xp_awarded": 50,
    "missions_remaining": 2
  }
}

// Zone entered (anonymized after 90 days)
{
  "event_id": "550e8400-e29b-41d4-a716-446655440002",
  "event_name": "world.zone.entered",
  "timestamp": "2026-01-14T15:00:00.000Z",
  "source": "server",
  "environment": "production",
  "identity_id": null,
  "anonymous_id": "a1b2c3d4e5f6...",
  "community_id": 1,
  "properties": {
    "zone_type": "social_east",
    "from_zone": "central_hub"
  }
}
```

---

## 4. Event Collection

### 4.1 Collection Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Frontend   │     │    API      │     │   Server    │
│  Events     │     │   Events    │     │   Events    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │    Event Collector    │
               │    (BullMQ Queue)     │
               └───────────┬───────────┘
                           │
                           ▼
               ┌───────────────────────┐
               │   CONSENT GATE        │  ← Step 2: Consent-gated ingestion
               │   ├─ Check consent    │
               │   ├─ Validate schema  │
               │   └─ Block prohibited │
               └───────────┬───────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
       ┌───────────┐ ┌───────────┐ ┌───────────┐
       │   MySQL   │ │  Redis    │ │  (Future) │
       │  events   │ │ counters  │ │ Analytics │
       │  table    │ │           │ │  Service  │
       └───────────┘ └───────────┘ └───────────┘
```

**Consent Gate Rules:**
- `whatsapp.*` events → Require `whatsapp_activity` consent → DROP if denied
- `user.*`, `world.*`, etc. → Require `telemetry` consent → Anonymize if denied
- `age.*`, `system.*` → No consent required (safety/operational)

### 4.2 Collection Methods

| Source | Method | Batching | Reliability |
|--------|--------|----------|-------------|
| Frontend | POST /api/events | Yes (10 events/5s) | Best effort |
| API | Direct insert | No | Guaranteed |
| Server (UC World) | Redis buffer | Yes (100/s) | Best effort |
| Jobs | Direct insert | No | Guaranteed |

### 4.3 Frontend SDK

```typescript
// Frontend event tracking
class EventTracker {
  private queue: BaseEvent[] = [];
  private flushInterval = 5000; // 5 seconds
  private maxBatchSize = 10;

  track(eventName: string, properties: Record<string, unknown>) {
    this.queue.push({
      event_id: crypto.randomUUID(),
      event_name: eventName,
      timestamp: new Date().toISOString(),
      source: 'frontend',
      properties
    });

    if (this.queue.length >= this.maxBatchSize) {
      this.flush();
    }
  }

  private async flush() {
    if (this.queue.length === 0) return;

    const batch = this.queue.splice(0, this.maxBatchSize);
    try {
      await fetch('/api/events', {
        method: 'POST',
        body: JSON.stringify({ events: batch })
      });
    } catch {
      // Events are not critical; log and continue
      console.warn('Failed to send events');
    }
  }
}
```

---

## 5. Event Consumers

### 5.1 Consumer Registry

| Consumer | Events Subscribed | Purpose | Latency Requirement |
|----------|-------------------|---------|---------------------|
| Analytics DB | All business events | Storage | < 1 minute |
| Real-time counters | gamification.* | Dashboard updates | < 1 second |
| Alerting | system.*, security.* | Incident detection | < 30 seconds |
| Aggregation job | All | Daily rollups | Daily batch |

### 5.2 Consumer Configuration

```yaml
consumers:
  analytics_db:
    events: ["*"]
    exclude: ["system.*"]
    handler: insertToMySQL
    retry: 3
    dlq: true

  realtime_counters:
    events:
      - "gamification.xp.awarded"
      - "gamification.mission.completed"
    handler: updateRedisCounters
    retry: 1
    dlq: false

  alerting:
    events:
      - "system.health.degraded"
      - "security.*"
      - "webhook.dead_lettered"
    handler: sendAlert
    retry: 3
    dlq: true
```

---

## 6. Storage

### 6.1 Raw Event Storage

```sql
CREATE TABLE analytics_events (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    event_id VARCHAR(36) UNIQUE NOT NULL,
    event_name VARCHAR(100) NOT NULL,
    event_version VARCHAR(10) NOT NULL DEFAULT '1.0.0',  -- Schema version
    timestamp TIMESTAMP(3) NOT NULL,
    source VARCHAR(20) NOT NULL,
    identity_id INT NULL,                                 -- NULL if consent denied or anonymized
    anonymous_id VARCHAR(64) NULL,                        -- SHA-256 hash for aggregation
    community_id INT NULL,
    session_id VARCHAR(64) NULL,
    properties JSON NOT NULL,
    consent_telemetry BOOLEAN NULL,                       -- Consent state at ingestion time
    consent_whatsapp BOOLEAN NULL,                        -- WhatsApp consent state
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_event_time (event_name, timestamp),
    INDEX idx_identity (identity_id, timestamp),
    INDEX idx_community (community_id, timestamp),
    INDEX idx_version (event_version)
) ENGINE=InnoDB
PARTITION BY RANGE (UNIX_TIMESTAMP(timestamp)) (
    PARTITION p_2026_01 VALUES LESS THAN (UNIX_TIMESTAMP('2026-02-01')),
    PARTITION p_2026_02 VALUES LESS THAN (UNIX_TIMESTAMP('2026-03-01')),
    PARTITION p_2026_03 VALUES LESS THAN (UNIX_TIMESTAMP('2026-04-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 6.2 Retention Implementation

```sql
-- Monthly partition maintenance
CREATE EVENT partition_maintenance
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
    -- Drop partitions older than 90 days
    -- (Events are anonymized before this)
    ALTER TABLE analytics_events DROP PARTITION p_oldest;

    -- Add new partition for upcoming month
    ALTER TABLE analytics_events ADD PARTITION (
        PARTITION p_new VALUES LESS THAN (UNIX_TIMESTAMP('NEW_DATE'))
    );
END;
```

---

## 7. Aggregations

### 7.1 Standard Aggregates

| Aggregate | Dimensions | Metrics | Schedule |
|-----------|------------|---------|----------|
| Daily Active Users | community_id, date | count(distinct identity) | Daily 01:00 |
| Mission Completion | community_id, mission_type, date | count, avg_xp | Daily 01:00 |
| Zone Dwell Time | community_id, zone_type, date | avg_duration, visits | Daily 01:00 |
| Retention Cohorts | signup_week, cohort_week | d1, d7, d30 retention | Weekly |

### 7.1.1 Living Graph Aggregates (Step 3)

These aggregates are required for the Living Graph community-to-community similarity computation.

| Aggregate Type | Window | Dimensions | Metrics | Purpose |
|----------------|--------|------------|---------|---------|
| `community_activity_daily` | 1 day | `{community_id}` | `{active_identities, sessions, dwell_total_sec}` | Baseline activity normalization |
| `zone_dwell_daily` | 1 day | `{community_id, zone_id}` | `{enter_count, exit_count, dwell_sec_bucketed}` | Proxemics evaluation |
| `whatsapp_activity_daily` | 1 day | `{community_id}` | `{messages, reactions, joins, leaves}` | Primary social rhythm |
| `mission_activity_daily` | 1 day | `{community_id, mission_type}` | `{completed, unique_completers}` | Interest/competence signature |
| `portal_activity_daily` | 1 day | `{from_community_id, to_community_id}` | `{travels, bounces, unique_travelers}` | Cross-community flow (E2 signal) |

**Implementation Rules:**
- Dwell is derived from `world.zone.entered` / `world.zone.exited` timestamp pairs
- WhatsApp aggregates only include consented users (`whatsapp_activity = true`)
- All aggregates run at **04:00 America/Sao_Paulo** daily
- Backfill capability required for 28 days (pilot requirement)

### 7.2 Aggregate Schema

```sql
CREATE TABLE analytics_aggregates (
    id INT PRIMARY KEY AUTO_INCREMENT,
    aggregate_type VARCHAR(50) NOT NULL,
    period_type ENUM('daily', 'weekly', 'monthly') NOT NULL,
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    dimensions JSON NOT NULL,
    metrics JSON NOT NULL,
    computed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    UNIQUE KEY uk_aggregate (aggregate_type, period_start, dimensions(255)),
    INDEX idx_type_period (aggregate_type, period_start)
);
```

### 7.3 Example Aggregates

```json
// Daily Active Users
{
  "aggregate_type": "dau",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "community_id": 1
  },
  "metrics": {
    "active_users": 150,
    "new_users": 12,
    "returning_users": 138
  }
}

// Mission Completion
{
  "aggregate_type": "mission_completion",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "community_id": 1,
    "mission_type": "send_messages"
  },
  "metrics": {
    "completions": 89,
    "unique_users": 72,
    "total_xp_awarded": 4450,
    "avg_completion_time_minutes": 45
  }
}

// Living Graph: Community Activity Daily
{
  "aggregate_type": "community_activity_daily",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "community_id": 1
  },
  "metrics": {
    "active_identities": 45,
    "sessions": 78,
    "dwell_total_sec": 145800
  }
}

// Living Graph: Zone Dwell Daily
{
  "aggregate_type": "zone_dwell_daily",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "community_id": 1,
    "zone_id": "central_hub"
  },
  "metrics": {
    "enter_count": 156,
    "exit_count": 152,
    "dwell_sec_bucketed": {
      "0-60": 45,
      "60-300": 78,
      "300-900": 25,
      "900+": 8
    }
  }
}

// Living Graph: WhatsApp Activity Daily
{
  "aggregate_type": "whatsapp_activity_daily",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "community_id": 1
  },
  "metrics": {
    "messages": 234,
    "reactions": 89,
    "joins": 3,
    "leaves": 1
  }
}

// Living Graph: Portal Activity Daily
{
  "aggregate_type": "portal_activity_daily",
  "period_type": "daily",
  "period_start": "2026-01-14",
  "period_end": "2026-01-14",
  "dimensions": {
    "from_community_id": 1,
    "to_community_id": 5
  },
  "metrics": {
    "travels": 12,
    "bounces": 3,
    "unique_travelers": 8
  }
}
```

---

## 8. Privacy and Compliance

### 8.1 PII Handling

| Field | Contains PII | Anonymization | Retention |
|-------|--------------|---------------|-----------|
| identity_id | Yes (linkable) | Set to NULL | 90 days |
| anonymous_id | No (hashed) | N/A | Aggregate lifetime |
| session_id | Yes (linkable) | Hash or delete | 90 days |
| properties.* | Depends | Review each property | 90 days |

### 8.2 Consent-Gated Ingestion (Living Graph Requirement)

All event ingestion MUST check consent before storing identity-linked data.

```typescript
/**
 * Consent-gated event ingestion
 *
 * CRITICAL: This is the enforcement point for user consent.
 * Events without proper consent MUST NOT store identity_id.
 */
async function ingestEvent(event: BaseEvent, identityId: number): Promise<void> {
  const consent = await getConsent(identityId);
  const eventDomain = event.event_name.split('.')[0]; // e.g., "whatsapp"

  // Determine required consent based on event domain
  const consentRequired = getRequiredConsent(eventDomain);

  // Check consent and gate accordingly
  if (consentRequired === 'whatsapp_activity' && !consent.whatsapp_activity) {
    // WhatsApp events require explicit opt-in - DROP if no consent
    console.log(`[Consent] Dropping ${event.event_name} - no whatsapp_activity consent`);
    return; // Do not store event
  }

  if (consentRequired === 'telemetry' && !consent.telemetry) {
    // Telemetry events - anonymize if no consent
    event.identity_id = null;
    event.anonymous_id = hashForAnonymity(identityId);
  }

  // Record consent state with event (for audit)
  event.consent_flags = {
    telemetry: consent.telemetry,
    whatsapp_activity: consent.whatsapp_activity,
  };

  // Validate schema before persistence
  validateEventSchema(event);

  await persistEvent(event);
}

function getRequiredConsent(domain: string): 'telemetry' | 'whatsapp_activity' | null {
  switch (domain) {
    case 'whatsapp': return 'whatsapp_activity';
    case 'user':
    case 'gamification':
    case 'community':
    case 'world': return 'telemetry';
    case 'age':
    case 'webhook':
    case 'system': return null; // No consent required
    default: return 'telemetry';
  }
}
```

### 8.3 Content Prohibition (Hard Guarantee)

**CRITICAL:** The following content MUST NEVER be stored in `analytics_events`:

| Prohibited Content | Reason | Enforcement |
|--------------------|--------|-------------|
| Message text/body | Privacy, LGPD | Schema validation rejects `message_content`, `body`, `text` fields |
| Media URLs | Privacy | Schema validation rejects `media_url`, `attachment` fields |
| Phone numbers | PII | Schema validation, phone fields stripped |
| Email addresses | PII | Schema validation, email fields stripped |
| GPS coordinates | Location tracking | Schema validation rejects `lat`, `lng`, `coordinates` |
| IP addresses | PII | Never logged in events |

```typescript
// Schema validation - HARD BLOCK on prohibited fields
const PROHIBITED_PROPERTY_PATTERNS = [
  /^(message_)?content$/i,
  /^(message_)?body$/i,
  /^(message_)?text$/i,
  /^media_url$/i,
  /^attachment/i,
  /^phone/i,
  /^email$/i,
  /^(lat|lng|latitude|longitude|coordinates)$/i,
  /^ip_address$/i,
];

function validateEventSchema(event: BaseEvent): void {
  for (const key of Object.keys(event.properties)) {
    for (const pattern of PROHIBITED_PROPERTY_PATTERNS) {
      if (pattern.test(key)) {
        throw new Error(`PROHIBITED: Property '${key}' cannot be stored in analytics events`);
      }
    }
  }
}
```

> **Audit Requirement:** Any attempt to store prohibited content MUST be logged to security audit trail with event details (excluding the prohibited content itself).

### 8.4 Data Subject Rights

| Right | Implementation |
|-------|----------------|
| Access | Export events with identity_id |
| Deletion | Anonymize events (set identity_id = NULL) |
| Objection | Respect telemetry consent opt-out |

---

## 9. Dashboards and Reporting

### 9.1 Standard Dashboards

| Dashboard | Audience | Refresh | Key Metrics |
|-----------|----------|---------|-------------|
| Executive Summary | Leadership | Daily | DAU, retention, revenue |
| Community Health | Admins | Hourly | Activity, engagement, churn |
| Gamification | Product | Real-time | XP distribution, mission completion |
| UC World | Product | Real-time | Population, zone usage |
| System Health | Engineering | Real-time | Errors, latency, throughput |

### 9.2 Metric Definitions

| Metric | Definition | Calculation |
|--------|------------|-------------|
| DAU | Daily Active Users | COUNT(DISTINCT identity_id) WHERE event_name IN (...) |
| D1 Retention | Users active day after signup | Users active on day N / Users signed up on day N-1 |
| D7 Retention | Users active 7 days after signup | Users active on day N / Users signed up on day N-7 |
| Mission Completion Rate | % of assigned missions completed | Completed missions / Assigned missions |
| Avg Session Duration | Time spent in platform | SUM(session_duration) / COUNT(sessions) |

### 9.3 Living Graph Dashboards (Step 6)

Three operational dashboards provide visibility into graph health and portal performance.

#### 9.3.1 Community Health Dashboard

**Audience:** Community Admins, Ops Team
**Refresh:** Hourly (real-time for alerts)
**Data Sources:** `community_activity_daily`, `zone_dwell_daily`, `members`, `gamification_points`

| Metric | Definition | Calculation | Alert Threshold |
|--------|------------|-------------|-----------------|
| Active Members (7d) | Members with activity in last 7 days | COUNT(DISTINCT identity_id) FROM community_activity_daily WHERE date >= TODAY-7 | < 30 (k-anon threshold) |
| Onboarding Rate | New members completing first mission within 48h | Completed / Joined (rolling 7d) | < 40% |
| Activity Trend | Week-over-week activity change | (This week DAU / Last week DAU) - 1 | < -20% |
| Zone Dwell Distribution | Time spent across zones | AVG(total_dwell_seconds) GROUP BY zone_type | N/A (informational) |
| Engagement Score | Composite health metric | (activity_score * 0.4) + (retention_score * 0.3) + (social_score * 0.3) | < 50 |

**Proxemic Proxies (spatial indicators of community health):**

| Proxy | Measures | Good Signal | Warning Signal |
|-------|----------|-------------|----------------|
| Hub Dwell Ratio | Time in central_hub vs. total | 20-40% | > 60% (users stuck) |
| Zone Diversity | Unique zones visited per session | > 3 zones | < 2 zones |
| Social Zone Time | Time in social_east + social_west | > 30% of session | < 10% |
| Portal Zone Visits | Users entering portal zones | > 10% of active users | < 2% |

#### 9.3.2 Portal Performance Dashboard

**Audience:** Ops Team, Product
**Refresh:** Hourly
**Data Sources:** `portal_activity_daily`, `portal_reports`, `world.portal.*` events

| Metric | Definition | Calculation | Alert Threshold |
|--------|------------|-------------|-----------------|
| Portal Travel Rate | Travels per active user per day | COUNT(world.portal.traveled) / DAU | Informational |
| Visitor Conversion | Visitors who join after visit | Joins from visitors / Total visitors (7d) | < 5% (low value) |
| Bounce Rate | Visitors who leave within 60s | COUNT(duration < 60) / COUNT(visits) | > 50% |
| Portal Report Rate | Reports per 1000 travels | (reports / travels) * 1000 | > 5 per 1000 |
| Card View Rate | Portal cards opened per portal zone entry | card.viewed / portal.activated | Informational |
| Why Click Rate | Users expanding explanation | why.clicked / card.viewed | Informational |
| Hide Rate | Users hiding portals | hide.clicked / card.viewed | > 20% (bad matches) |

**Per-Portal Breakdown (by source_community → destination_community):**

| Metric | Definition | Action Trigger |
|--------|------------|----------------|
| Travel Count (7d) | Total travels through this portal | N/A |
| Bounce Rate | % leaving within 60s | > 60% → review edge quality |
| Report Count | Total reports on this portal | > 3 → ops review |
| Conversion Rate | Visitors who joined | < 2% → consider removing edge |

#### 9.3.3 Neighbor Stability Dashboard

**Audience:** Ops Team, Graph Engineers
**Refresh:** Daily (after graph build)
**Data Sources:** `community_portals`, `community_edges`, `community_graph_runs`

| Metric | Definition | Calculation | Alert Threshold |
|--------|------------|-------------|-----------------|
| Churn Rate | % of portals changed from previous period | Changed portals / Total portals | > 30% |
| Churn-Limited Communities | Communities hitting churn limit | COUNT WHERE churn_limited = true | > 10% of communities |
| Edge Stability | % of edges unchanged from previous period | Stable edges / Total edges | < 70% |
| Average Neighbor Tenure | Days a neighbor stays assigned | AVG(days_assigned) | < 7 days |
| Orphan Communities | Communities with 0 active portals | COUNT WHERE active_portals = 0 | > 5% |
| Override Count | Active safety overrides | COUNT FROM community_graph_overrides WHERE expires_at IS NULL OR expires_at > NOW() | > 10 |

**Graph Build Health:**

| Metric | Definition | Alert Threshold |
|--------|------------|-----------------|
| Build Success Rate | Successful runs / Total runs (7d) | < 100% |
| Build Duration | Time to complete graph build | > 12 minutes |
| Edge Count Delta | Day-over-day edge count change | > ±20% |
| Portal Assignment Coverage | Communities with ≥1 portal | < 90% of eligible |

### 9.4 Dashboard Alert Thresholds

| Alert | Condition | Severity | Action |
|-------|-----------|----------|--------|
| Community Below K-Anon | active_members_7d < 30 | High | Hide from graph, notify ops |
| High Portal Bounce | bounce_rate > 60% for 3 days | Medium | Review edge, consider removal |
| Excessive Churn | churn_rate > 40% | High | Review graph params, pause if needed |
| Report Spike | report_rate > 10 per 1000 | Critical | Page on-call, review reported portals |
| Build Failure | graph_build_status = 'failed' | Critical | Page on-call, fallback to previous |
| Aggregation Gap | consecutive_success_days < 7 | High | Block graph build, investigate |
| Override Accumulation | active_overrides > 20 | Medium | Review override reasons |
| Orphan Spike | orphan_communities > 10% | Medium | Review eligibility rules |

### 9.5 Export Functionality

All dashboard data supports CSV/JSON export for offline analysis.

#### 9.5.1 Exportable Reports

| Report | Contents | Frequency | Access |
|--------|----------|-----------|--------|
| Community Health Summary | All health metrics per community | Daily | Admin+ |
| Portal Performance Report | Travel, bounce, conversion by portal | Weekly | Admin+ |
| Graph Stability Report | Churn, edge changes, overrides | Weekly | Ops only |
| Aggregated Activity Export | Community-level aggregates (no PII) | On-demand | Admin+ |

#### 9.5.2 Export API

```typescript
// Export endpoint
GET /api/admin/dashboard/export

// Query parameters
interface ExportParams {
  report_type: 'community_health' | 'portal_performance' | 'graph_stability' | 'activity';
  community_id?: number;       // Optional filter
  date_from: string;           // ISO date
  date_to: string;             // ISO date
  format: 'csv' | 'json';
}

// Response headers
Content-Type: text/csv | application/json
Content-Disposition: attachment; filename="report_<type>_<date>.csv"

// Rate limit
10 exports per hour per admin
```

#### 9.5.3 Data Minimization in Exports

Exports follow data minimization principles:

| Field Type | In Export | Notes |
|------------|-----------|-------|
| Community aggregates | Yes | Activity, health metrics |
| Identity IDs | No | Never exported |
| Member counts | Yes | Aggregated only |
| Individual events | No | Only aggregates |
| Portal assignments | Yes | Community IDs only |
| Reason codes | Yes | Graph explainability |

---

*This document defines how events flow through the system. All new events must be added to the catalog before implementation.*

<!-- Last Updated: 2026-01-19 - Step 6: Added Living Graph Dashboards (Section 9.3-9.5) with community health, portal performance, neighbor stability metrics, alerts, and export functionality -->
