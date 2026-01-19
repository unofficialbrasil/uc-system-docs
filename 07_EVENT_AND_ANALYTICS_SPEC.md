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
| `world.portal.activated` | User enters portal zone | `{ portal_direction, destination_community_id, destination_name, reason_code }` | Server |
| `world.portal.traveled` | Portal travel completed | `{ source_community_id, destination_community_id, portal_direction, travel_reason }` | Server |
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
  event_id: string;           // UUID
  event_name: string;         // From catalog
  timestamp: string;          // ISO 8601
  source: 'frontend' | 'api' | 'server' | 'job' | 'webhooks';
  environment: 'production' | 'staging' | 'development';

  // Identity (anonymizable)
  identity_id?: number;       // NULL after anonymization
  anonymous_id?: string;      // SHA-256 hash for aggregation

  // Context
  session_id?: string;        // Hashed
  community_id?: number;
  request_id?: string;        // For tracing

  // Event-specific data
  properties: Record<string, unknown>;
}
```

### 3.2 Example Events

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
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
       ┌───────────┐ ┌───────────┐ ┌───────────┐
       │   MySQL   │ │  Redis    │ │  (Future) │
       │  events   │ │ counters  │ │ Analytics │
       │  table    │ │           │ │  Service  │
       └───────────┘ └───────────┘ └───────────┘
```

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
    timestamp TIMESTAMP(3) NOT NULL,
    source VARCHAR(20) NOT NULL,
    identity_id INT NULL,
    anonymous_id VARCHAR(64) NULL,
    community_id INT NULL,
    session_id VARCHAR(64) NULL,
    properties JSON NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_event_time (event_name, timestamp),
    INDEX idx_identity (identity_id, timestamp),
    INDEX idx_community (community_id, timestamp)
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

### 8.2 Consent Integration

```typescript
async function trackEvent(event: BaseEvent, identityId: number) {
  // Check telemetry consent
  const consent = await getConsent(identityId);

  if (!consent.telemetry) {
    // Track anonymously without identity link
    event.identity_id = null;
    event.anonymous_id = hashForAnonymity(identityId);
  }

  await persistEvent(event);
}
```

### 8.3 Data Subject Rights

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

---

*This document defines how events flow through the system. All new events must be added to the catalog before implementation.*

<!-- Last Updated: 2026-01-19 - Step 1: Event taxonomy upgrade - fixed naming convention, added portal.activated, added WhatsApp member events -->
