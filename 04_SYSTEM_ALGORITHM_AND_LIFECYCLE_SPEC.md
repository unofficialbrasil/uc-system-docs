# System Algorithm and Lifecycle Specification

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## Purpose

This document describes the **complete end-to-end behavior** of the Unofficial Communities SaaS. No system behavior is allowed to exist without being documented here.

---

## Table of Contents

1. [Landing Page and Anonymous Access](#1-landing-page-and-anonymous-access)
2. [Session Initialization](#2-session-initialization)
3. [Authentication and Account Creation](#3-authentication-and-account-creation)
4. [User State Initialization](#4-user-state-initialization)
5. [Onboarding Logic](#5-onboarding-logic)
6. [Feature Execution Flows](#6-feature-execution-flows)
7. [Gamification Engine](#7-gamification-engine)
8. [UC World Lifecycle](#8-uc-world-lifecycle)
9. [Background Jobs and Async Processing](#9-background-jobs-and-async-processing)
10. [Data Mutation Rules](#10-data-mutation-rules)
11. [Error Handling and Retries](#11-error-handling-and-retries)
12. [Edge Cases](#12-edge-cases)
13. [Account Suspension](#13-account-suspension)
14. [Account Deletion](#14-account-deletion)
15. [Data Anonymization and Final Termination](#15-data-anonymization-and-final-termination)

---

## 1. Landing Page and Anonymous Access

### 1.1 Public Routes (No Authentication Required)

| Route | Purpose | Behavior |
|-------|---------|----------|
| `/` | Landing page | Marketing content, signup CTA |
| `/about` | About page | Company information |
| `/pricing` | Pricing page | Plan comparison |
| `/login` | Login page | Authentication form |
| `/signup` | Signup page | Registration form |
| `/join/:inviteCode` | Invite link | Community-specific signup |

### 1.2 Anonymous User Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     ANONYMOUS USER ARRIVES                       │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │   Check for existing session   │
               │   Cookie: uc_session           │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  No Cookie      │              │  Cookie Exists  │
    │  Stay Anonymous │              │  Validate in    │
    │                 │              │  Redis          │
    └─────────────────┘              └─────────────────┘
              │                                 │
              │                    ┌────────────┴────────────┐
              │                    │                         │
              │                    ▼                         ▼
              │          ┌─────────────────┐      ┌─────────────────┐
              │          │  Valid Session  │      │  Invalid/Expired│
              │          │  Redirect to    │      │  Clear Cookie   │
              │          │  /dashboard     │      │  Stay Anonymous │
              │          └─────────────────┘      └─────────────────┘
              │
              ▼
    ┌─────────────────────────────────────────────────────────────┐
    │                   ANONYMOUS ALLOWED ACTIONS                  │
    ├─────────────────────────────────────────────────────────────┤
    │  • View landing page                                         │
    │  • View pricing                                              │
    │  • Start signup flow                                         │
    │  • Start login flow                                          │
    │  • Use invite link                                           │
    └─────────────────────────────────────────────────────────────┘
```

### 1.3 Session Cookie Specification

| Property | Value |
|----------|-------|
| Name | `uc_session` |
| HttpOnly | `true` |
| Secure | `true` (production) |
| SameSite | `lax` |
| Path | `/` |
| Max-Age | `604800` (7 days) |

---

## 2. Session Initialization

### 2.1 Session Creation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   SESSION CREATION TRIGGER                       │
│            (Successful authentication event)                     │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Generate Session ID          │
               │  crypto.randomUUID()          │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Create Redis Entry           │
               │  Key: uc:sess:<sessionId>     │
               │  Value: {                     │
               │    identityId: <number>,      │
               │    createdAt: <timestamp>,    │
               │    userAgent: <string>,       │
               │    ip: <string>               │
               │  }                            │
               │  TTL: 604800 (7 days)         │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Set Cookie in Response       │
               │  uc_session=<sessionId>       │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Log Security Event           │
               │  type: session_created        │
               └───────────────────────────────┘
```

### 2.2 Session Validation Flow (Every Protected Request)

```
┌─────────────────────────────────────────────────────────────────┐
│                   PROTECTED ROUTE REQUEST                        │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Extract uc_session cookie    │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Cookie Missing │              │  Cookie Present │
    │  Return 401     │              │  Query Redis    │
    └─────────────────┘              └─────────────────┘
                                              │
                              ┌───────────────┴───────────────┐
                              │                               │
                              ▼                               ▼
                    ┌─────────────────┐          ┌─────────────────┐
                    │  Key Not Found  │          │  Key Found      │
                    │  Return 401     │          │  Parse Session  │
                    │  Clear Cookie   │          │  Continue       │
                    └─────────────────┘          └─────────────────┘
                                                          │
                                                          ▼
                                        ┌───────────────────────────────┐
                                        │  Touch TTL (slide expiration) │
                                        │  EXPIRE uc:sess:<id> 604800   │
                                        └───────────────────────────────┘
                                                          │
                                                          ▼
                                        ┌───────────────────────────────┐
                                        │  Attach identityId to request │
                                        │  req.session = { identityId } │
                                        └───────────────────────────────┘
```

### 2.3 Session Termination

| Trigger | Action | Behavior |
|---------|--------|----------|
| Explicit Logout | User clicks logout | DELETE Redis key, clear cookie |
| Session Expiry | 7 days inactivity | Redis TTL expires automatically |
| Security Event | Suspicious activity detected | DELETE Redis key, clear cookie, log event |
| Account Deletion | User deletes account | DELETE all user sessions |

---

## 3. Authentication and Account Creation

### 3.1 Signup Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER STARTS SIGNUP                        │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Step 1: Enter Phone Number   │
               │  Brazilian format: +55...     │
               │  Validate: ^\\+55\\d{10,11}$  │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Step 2: Send WhatsApp OTP    │
               │  via WhatsApp Cloud API       │
               │  Template: verification_code   │
               │  Code: 6-digit random          │
               │  TTL: 10 minutes              │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  API Success    │              │  API Failure    │
    │  Show OTP Input │              │  Show Error     │
    │                 │              │  Allow Retry    │
    └─────────────────┘              └─────────────────┘
                │
                ▼
    ┌───────────────────────────────┐
    │  Step 3: User Enters OTP      │
    │  Max attempts: 3              │
    │  Lockout: 15 min after 3 fail │
    └───────────────────────────────┘
                │
    ┌───────────┴───────────┐
    │                       │
    ▼                       ▼
┌───────────┐        ┌───────────┐
│ OTP Valid │        │OTP Invalid│
│           │        │Decrement  │
│           │        │Attempts   │
└───────────┘        └───────────┘
      │
      ▼
┌───────────────────────────────┐
│  Step 4: Check Existing User  │
│  SELECT FROM identities       │
│  WHERE phone = ?              │
└───────────────────────────────┘
      │
┌─────┴─────┐
│           │
▼           ▼
┌─────┐  ┌─────────────────────────────────┐
│Found│  │  Not Found: Create New Identity │
│Login│  │  INSERT INTO identities         │
│Flow │  │  (phone, created_at, status)    │
└─────┘  └─────────────────────────────────┘
               │
               ▼
┌───────────────────────────────┐
│  Step 5: Profile Creation     │
│  - Display name (required)    │
│  - Avatar (optional)          │
│  - Bio (optional)             │
└───────────────────────────────┘
               │
               ▼
┌───────────────────────────────┐
│  Step 6: Community Selection  │
│  - From invite link, OR       │
│  - Browse public communities  │
└───────────────────────────────┘
               │
               ▼
┌───────────────────────────────┐
│  Step 7: Create Session       │
│  Create member record         │
│  Initialize gamification      │
│  Redirect to /dashboard       │
└───────────────────────────────┘
```

### 3.2 Login Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER STARTS LOGIN                        │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Enter Phone Number           │
               │  Send WhatsApp OTP            │
               │  (Same as signup steps 1-3)   │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Verify OTP                   │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  User Exists    │              │  User Not Found │
    │  Check Status   │              │  Redirect to    │
    │                 │              │  /signup        │
    └─────────────────┘              └─────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
┌────────┐        ┌──────────┐
│ Active │        │Suspended │
│Create  │        │Show Error│
│Session │        │          │
└────────┘        └──────────┘
      │
      ▼
┌───────────────────────────────┐
│  Log Login Event              │
│  Update last_login_at         │
│  Redirect to /dashboard       │
└───────────────────────────────┘
```

### 3.3 OTP Rate Limiting

| Condition | Limit | Lockout |
|-----------|-------|---------|
| OTP requests per phone | 3 per 10 minutes | 10 minute wait |
| OTP verification attempts | 3 per code | New code required |
| Failed verifications per phone | 5 per hour | 1 hour lockout |

---

## 4. User State Initialization

### 4.1 New User Setup Sequence

When a new identity is created, the following records are initialized:

```sql
-- 1. Identity record (created during signup)
INSERT INTO identities (phone, status, created_at)
VALUES (?, 'active', NOW());

-- 2. Profile record (after profile step)
INSERT INTO profiles (identity_id, display_name, avatar_url, bio, created_at)
VALUES (?, ?, ?, ?, NOW());

-- 3. Membership record (when joining community)
INSERT INTO members (identity_id, community_id, role, joined_at)
VALUES (?, ?, 'member', NOW());

-- 4. Gamification initialization
INSERT INTO gamification_points (identity_id, community_id, xp, level, created_at)
VALUES (?, ?, 0, 1, NOW());

INSERT INTO gamification_streaks (identity_id, community_id, current_streak, best_streak, last_activity_date)
VALUES (?, ?, 0, 0, NULL);

-- 5. Consent record
INSERT INTO consents (identity_id, data_processing, marketing, telemetry, created_at)
VALUES (?, true, false, true, NOW());
```

### 4.2 Default Values

| Record | Field | Default | Rationale |
|--------|-------|---------|-----------|
| identity | status | `active` | Users start active |
| gamification_points | xp | `0` | No free XP |
| gamification_points | level | `1` | Start at level 1 |
| gamification_streaks | current_streak | `0` | Must earn streak |
| consents | data_processing | `true` | Required for service |
| consents | marketing | `false` | Opt-in only |
| consents | telemetry | `true` | Default on, opt-out available |

---

## 5. Onboarding Logic

### 5.1 Onboarding State Machine

```
┌─────────────────────────────────────────────────────────────────┐
│                     ONBOARDING STATES                            │
└─────────────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │   STARTED    │ ─── User clicked signup
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │  PHONE_SENT  │ ─── OTP sent to WhatsApp
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │PHONE_VERIFIED│ ─── OTP confirmed
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │PROFILE_CREATED│ ─── Name/avatar set
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │COMMUNITY_JOINED│ ─── Joined first community
    └──────────────┘
           │
           ▼
    ┌──────────────┐
    │  COMPLETED   │ ─── Full access granted
    └──────────────┘
```

### 5.2 Onboarding Checklist (Stored Per User)

| Step | Required | XP Reward | Check |
|------|----------|-----------|-------|
| Phone verified | Yes | 0 | `identity.phone IS NOT NULL` |
| Profile name set | Yes | 10 | `profile.display_name IS NOT NULL` |
| Avatar uploaded | No | 20 | `profile.avatar_url IS NOT NULL` |
| Bio written | No | 10 | `profile.bio IS NOT NULL` |
| First community joined | Yes | 50 | `COUNT(members) > 0` |
| First mission completed | No | 0 | `COUNT(completed_missions) > 0` |

### 5.3 Onboarding Nudges

| Condition | Trigger | Action |
|-----------|---------|--------|
| No avatar after 24h | Daily job | Show banner on dashboard |
| No bio after 48h | Daily job | Show banner on dashboard |
| No mission completed after 72h | Daily job | Send WhatsApp reminder (if opted in) |

---

## 6. Feature Execution Flows

### 6.1 Dashboard Load

```
GET /dashboard
    │
    ├── requireUserAuth()
    │       │
    │       ▼
    │   Validate session → Get identityId
    │
    ├── Parallel API calls:
    │   ├── GET /api/me → User profile
    │   ├── GET /api/gamification/points/{id}/summary → XP, level
    │   ├── GET /api/gamification/streaks/{id} → Streak data
    │   ├── GET /api/gamification/missions/{id}/today → Daily missions
    │   └── GET /api/world/overview → World stats (optional)
    │
    └── Render dashboard with aggregated data
```

### 6.2 Mission Completion

```
POST /api/gamification/missions/{id}/complete
    │
    ├── requireUserAuth()
    │
    ├── Validate:
    │   ├── Mission belongs to user's community
    │   ├── Mission not already completed today
    │   ├── Mission deadline not passed
    │   └── Mission criteria met (if verifiable)
    │
    ├── IF validation fails:
    │   └── Return 400 with error code
    │
    ├── IF validation passes:
    │   ├── Mark mission completed
    │   │   └── INSERT INTO completed_missions (...)
    │   │
    │   ├── Award XP (sync)
    │   │   └── UPDATE gamification_points SET xp = xp + ? WHERE ...
    │   │
    │   ├── Check level up (sync)
    │   │   └── IF xp >= threshold THEN UPDATE level
    │   │
    │   ├── Update streak (sync)
    │   │   └── CALL update_streak_procedure(...)
    │   │
    │   ├── Emit event (async)
    │   │   └── PUBLISH mission.completed { userId, missionId, xp }
    │   │
    │   └── Return 200 with updated stats
```

### 6.3 Ranking Fetch

```
GET /api/gamification/rankings/weekly
    │
    ├── requireUserAuth()
    │
    ├── Get user's community ID from session
    │
    ├── Query (with caching):
    │   │
    │   │  Cache key: rankings:weekly:{communityId}
    │   │  Cache TTL: 5 minutes
    │   │
    │   └── IF cache miss:
    │       └── SELECT
    │             p.identity_id,
    │             pr.display_name,
    │             pr.avatar_url,
    │             p.xp,
    │             p.level,
    │             RANK() OVER (ORDER BY p.xp DESC) as rank
    │           FROM gamification_points p
    │           JOIN profiles pr ON p.identity_id = pr.identity_id
    │           WHERE p.community_id = ?
    │             AND p.week_xp > 0
    │           ORDER BY p.xp DESC
    │           LIMIT 100
    │
    └── Return ranked list with current user's position highlighted
```

---

## 7. Gamification Engine

### 7.1 XP Award Rules

| Action | XP | Frequency Limit | Validation |
|--------|-----|-----------------|------------|
| Complete daily mission | 10-200 | Per mission | Mission exists, not completed |
| Send message in group | 5 | 10/day | Webhook verification |
| React to message | 2 | 20/day | Webhook verification |
| Join event | 50 | Per event | Event exists, not joined |
| Visit UC World | 10 | 1/day | Presence logged |
| Complete profile | 40 | Once | Profile complete check |
| 7-day streak | 50 | Weekly | Streak = 7 |
| 30-day streak | 200 | Monthly | Streak = 30 |

### 7.2 Level Thresholds

| Level | Title | XP Required | XP to Next |
|-------|-------|-------------|------------|
| 1 | Novato | 0 | 100 |
| 2 | Iniciante | 100 | 200 |
| 3 | Membro | 300 | 300 |
| 4 | Ativo | 600 | 400 |
| 5 | Engajado | 1000 | 500 |
| 6 | Veterano | 1500 | 700 |
| 7 | Destaque | 2200 | 800 |
| 8 | Influenciador | 3000 | 1000 |
| 9 | Lenda | 4000 | 1500 |
| 10 | Lenda Suprema | 5500 | ∞ |

### 7.3 Level Up Flow

```
FUNCTION check_level_up(identity_id, community_id):
    current_xp = SELECT xp FROM gamification_points WHERE ...
    current_level = SELECT level FROM gamification_points WHERE ...

    thresholds = [0, 100, 300, 600, 1000, 1500, 2200, 3000, 4000, 5500]

    new_level = current_level
    FOR i = current_level TO 10:
        IF current_xp >= thresholds[i]:
            new_level = i + 1
        ELSE:
            BREAK

    IF new_level > current_level:
        UPDATE gamification_points SET level = new_level WHERE ...
        EMIT event.level_up { identity_id, old_level, new_level }
        RETURN { leveled_up: true, new_level }

    RETURN { leveled_up: false }
```

### 7.4 Streak Calculation

```
FUNCTION update_streak(identity_id, community_id, activity_date):
    last_activity = SELECT last_activity_date FROM gamification_streaks WHERE ...
    current_streak = SELECT current_streak FROM gamification_streaks WHERE ...
    best_streak = SELECT best_streak FROM gamification_streaks WHERE ...

    IF last_activity IS NULL:
        -- First activity ever
        new_streak = 1
    ELSE IF activity_date = last_activity:
        -- Same day, no change
        RETURN current_streak
    ELSE IF activity_date = last_activity + 1 day:
        -- Consecutive day
        new_streak = current_streak + 1
    ELSE:
        -- Streak broken
        new_streak = 1

    new_best = MAX(new_streak, best_streak)

    UPDATE gamification_streaks SET
        current_streak = new_streak,
        best_streak = new_best,
        last_activity_date = activity_date
    WHERE ...

    -- Check for streak milestones
    IF new_streak IN (7, 30, 100) AND new_streak > current_streak:
        EMIT event.streak_milestone { identity_id, streak: new_streak }
        award_streak_bonus_xp(identity_id, new_streak)

    RETURN new_streak
```

### 7.5 Mission Generation

Daily missions are generated at 00:00 UTC-3 (São Paulo timezone):

```
FUNCTION generate_daily_missions(community_id):
    -- Select 3-5 missions from template pool
    templates = SELECT * FROM mission_templates WHERE community_id = ?

    -- Weight by user's completion history (favor not-recently-completed)
    weighted_templates = apply_recency_weighting(templates)

    -- Select missions ensuring variety
    selected = SELECT_RANDOM(weighted_templates, count=RANDOM(3,5))

    -- Create mission instances for today
    FOR template IN selected:
        INSERT INTO daily_missions (
            community_id,
            template_id,
            date,
            xp_reward,
            expires_at
        ) VALUES (
            community_id,
            template.id,
            TODAY(),
            template.base_xp,
            TODAY() + 1 day
        )
```

---

## 8. UC World Lifecycle

### 8.1 World Connection Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    USER OPENS UC WORLD                           │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Load Three.js Scene          │
               │  Initialize renderer          │
               │  Load assets (Kenney pack)    │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Get Session Token            │
               │  From uc_session cookie       │
               └───────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Connect to Colyseus          │
               │  ws://localhost:3006          │
               │  Room: "community_<id>"       │
               │  Auth: { token: sessionId }   │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Auth Failed    │              │  Auth Success   │
    │  Show Error     │              │  Join Room      │
    │  Redirect Login │              │                 │
    └─────────────────┘              └─────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Receive Initial State        │
                              │  - All players in room        │
                              │  - Current player position    │
                              │  - Zone boundaries            │
                              └───────────────────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Spawn Avatar                 │
                              │  Position: Onboarding zone    │
                              │  OR last known position       │
                              └───────────────────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Start Game Loop              │
                              │  - Render at 60fps            │
                              │  - Send input at 20hz         │
                              │  - Receive state at 20hz      │
                              └───────────────────────────────┘
```

### 8.2 Movement and State Sync

```
CLIENT (20Hz):
    input = { x, y, direction }
    colyseus.room.send("move", input)

SERVER:
    onMessage("move", (client, input)):
        player = state.players.get(client.sessionId)

        -- Validate input
        IF NOT valid_position(input.x, input.y):
            RETURN  -- Ignore invalid input

        -- Check collision
        IF collides_with_obstacle(input.x, input.y):
            RETURN  -- Ignore colliding move

        -- Check zone boundaries
        new_zone = get_zone_at(input.x, input.y)
        IF new_zone != player.currentZone:
            emit_zone_change(player, new_zone)

        -- Update state (auto-synced to clients)
        player.x = input.x
        player.y = input.y
        player.direction = input.direction
        player.currentZone = new_zone

CLIENT (on state change):
    FOR player IN state.players:
        update_avatar_position(player)
```

### 8.3 Zone Events

| Zone | On Enter | On Exit |
|------|----------|---------|
| Central Hub | Log presence | - |
| Workshop | Show tools UI | Hide tools UI |
| Lounge | Enable chat | - |
| Brand Zone | Log visit for analytics | - |
| Quiet Room | Mute all audio | Unmute |
| Onboarding | Show tutorial | - |
| Portal (any) | Show portal UI | Hide portal UI |

### 8.4 Disconnect Handling

```
SERVER (Colyseus):
    onLeave(client, consented):
        player = state.players.get(client.sessionId)

        IF consented:
            -- Clean disconnect
            log_session_end(player)
            state.players.delete(client.sessionId)
        ELSE:
            -- Unexpected disconnect
            player.disconnectedAt = NOW()

            -- Allow 30s reconnection window
            setTimeout(() => {
                IF player.disconnectedAt IS NOT NULL:
                    log_session_end(player)
                    state.players.delete(client.sessionId)
            }, 30000)

    onReconnect(client):
        player = state.players.get(client.sessionId)
        IF player AND player.disconnectedAt:
            player.disconnectedAt = NULL
            -- Restore player state
```

---

## 9. Background Jobs and Async Processing

### 9.1 Job Registry

| Job | Queue | Schedule | Idempotent | Dead Letter |
|-----|-------|----------|------------|-------------|
| generate_daily_missions | missions | 00:00 BRT daily | Yes | dlq:missions |
| reset_weekly_rankings | rankings | Monday 00:00 BRT | Yes | dlq:rankings |
| process_webhook | webhooks | Real-time | Yes | dlq:webhooks |
| send_streak_reminder | notifications | 18:00 BRT daily | Yes | dlq:notifications |
| cleanup_expired_sessions | maintenance | 03:00 BRT daily | Yes | - |
| backup_database | maintenance | 03:00 BRT daily | Yes | - |

### 9.2 Webhook Processing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│              WEBHOOK RECEIVED FROM META                          │
│              POST /webhooks/whatsapp                             │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
               ┌───────────────────────────────┐
               │  Verify Signature             │
               │  X-Hub-Signature-256 header   │
               └───────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │  Invalid Sig    │              │  Valid Sig      │
    │  Return 401     │              │  Continue       │
    │  Log Security   │              │                 │
    └─────────────────┘              └─────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Return 200 Immediately       │
                              │  (Meta requires fast response)│
                              └───────────────────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  Enqueue to BullMQ            │
                              │  Queue: webhooks              │
                              │  Job: { payload, receivedAt } │
                              └───────────────────────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │  WORKER: Process Job          │
                              │  1. Parse webhook type        │
                              │  2. Route to handler          │
                              │  3. Execute business logic    │
                              │  4. Mark complete             │
                              └───────────────────────────────┘
```

### 9.3 Webhook Types and Handlers

| Webhook Type | Handler | Business Logic |
|--------------|---------|----------------|
| `messages` | handleMessage | Award XP for group message |
| `message_reactions` | handleReaction | Award XP for reaction |
| `group_membership` | handleMembership | Update member status |
| `verification_status` | handleVerification | Complete phone verification |

---

## 10. Data Mutation Rules

### 10.1 Create Operations

| Entity | Creator | Validation | Side Effects |
|--------|---------|------------|--------------|
| Identity | System (signup) | Unique phone | Create profile stub |
| Profile | User | Valid identity_id | - |
| Member | User (join) | Valid community, not already member | Initialize gamification |
| Community | Admin | Valid creator identity | Create default zones |
| Mission | System (job) | Valid template | - |

### 10.2 Update Operations

| Entity | Who Can Update | Fields | Validation |
|--------|----------------|--------|------------|
| Profile | Owner only | display_name, avatar_url, bio | Length limits |
| Gamification | System only | xp, level, streak | Never decrease XP |
| Member | Owner, Admin | role, status | Valid transitions |
| Community | Admin only | name, settings | Owner validation |

### 10.3 Delete Operations

| Entity | Who Can Delete | Soft/Hard | Side Effects |
|--------|----------------|-----------|--------------|
| Profile | Owner | Soft (anonymize) | See Account Deletion |
| Member | Owner, Admin | Soft | Preserve for analytics |
| Community | Owner | Soft | Notify all members |
| Session | System | Hard | Immediate |

---

## 11. Error Handling and Retries

### 11.1 Error Categories

| Category | HTTP Code | Retry | User Message |
|----------|-----------|-------|--------------|
| Validation | 400 | No | Specific field errors |
| Authentication | 401 | No | "Please log in again" |
| Authorization | 403 | No | "You don't have permission" |
| Not Found | 404 | No | "Resource not found" |
| Rate Limited | 429 | After delay | "Too many requests, try again later" |
| Server Error | 500 | Yes (3x) | "Something went wrong, please retry" |
| Service Unavailable | 503 | Yes (3x) | "Service temporarily unavailable" |

### 11.2 Retry Configuration

```javascript
const retryConfig = {
  maxRetries: 3,
  backoff: 'exponential',
  delays: [1000, 5000, 15000], // ms
  retryableErrors: [500, 502, 503, 504, 'ETIMEDOUT', 'ECONNRESET']
};
```

### 11.3 Dead Letter Queue Handling

```
JOB fails after max retries:
    │
    ├── Move to dead letter queue
    │
    ├── Log error with full context
    │   - Job ID
    │   - Payload
    │   - Error stack
    │   - Attempt count
    │   - Timestamps
    │
    ├── Alert on-call if > 10 DLQ items
    │
    └── Manual review required
```

---

## 12. Edge Cases

### 12.1 Concurrent Session Handling

| Scenario | Behavior |
|----------|----------|
| Same user logs in from new device | Both sessions valid |
| Same user opens multiple tabs | Shared session, no conflict |
| Session expires during action | Return 401, prompt re-login |

### 12.2 Race Conditions

| Scenario | Prevention |
|----------|------------|
| Double XP award | Idempotency key on mission completion |
| Concurrent profile update | Last-write-wins (acceptable) |
| Simultaneous join requests | Database unique constraint |

### 12.3 Network Failures

| Scenario | Client Behavior | Server Behavior |
|----------|-----------------|-----------------|
| API timeout | Retry with backoff | Request may have succeeded |
| WebSocket disconnect | Attempt reconnect | 30s grace period |
| Partial response | Treat as failure | Transaction rollback |

### 12.4 Data Inconsistency

| Scenario | Detection | Resolution |
|----------|-----------|------------|
| XP mismatch | Periodic audit job | Recalculate from events |
| Orphaned member | FK constraint | Cascade delete |
| Missing profile | Startup check | Create stub |

---

## 13. Account Suspension

### 13.1 Suspension Triggers

| Trigger | Automatic | Appeal |
|---------|-----------|--------|
| TOS violation report (verified) | No | Yes |
| Spam detection | Yes | Yes |
| Payment fraud | Yes | Yes |
| Admin action | No | Yes |

### 13.2 Suspension Flow

```
SUSPEND_ACCOUNT(identity_id, reason, duration):
    │
    ├── Update identity status
    │   UPDATE identities SET status = 'suspended',
    │     suspended_at = NOW(), suspended_reason = ?
    │   WHERE id = ?
    │
    ├── Invalidate all sessions
    │   DELETE FROM redis WHERE key LIKE 'uc:sess:*'
    │     AND identity_id = ?
    │
    ├── Disable gamification
    │   UPDATE gamification_points SET frozen = true WHERE identity_id = ?
    │
    ├── Remove from active rooms (UC World)
    │   EMIT force_disconnect { identity_id }
    │
    ├── Send notification
    │   WhatsApp message with reason and appeal info
    │
    └── Log security event
```

### 13.3 Suspended User Behavior

| Action | Allowed | Response |
|--------|---------|----------|
| Login attempt | No | "Account suspended" message |
| API access | No | 403 with suspension info |
| UC World access | No | Connection rejected |
| Appeal submission | Yes | Via support email |

---

## 14. Account Deletion

### 14.1 Deletion Request Flow

```
USER REQUESTS DELETION:
    │
    ├── Verify identity (re-authenticate)
    │
    ├── Show data summary
    │   - Profile data
    │   - Communities
    │   - Gamification stats
    │
    ├── Confirm understanding
    │   - "This cannot be undone"
    │   - "Your data will be anonymized"
    │
    ├── Cooling-off period: 7 days
    │   - Can cancel during this period
    │   - Account marked 'pending_deletion'
    │
    └── After 7 days: Execute deletion
```

### 14.2 Deletion Execution

```
EXECUTE_DELETION(identity_id):
    │
    ├── Anonymize profile
    │   UPDATE profiles SET
    │     display_name = 'Deleted User',
    │     avatar_url = NULL,
    │     bio = NULL
    │   WHERE identity_id = ?
    │
    ├── Anonymize identity
    │   UPDATE identities SET
    │     phone = HASH(phone),
    │     email = NULL,
    │     status = 'deleted',
    │     deleted_at = NOW()
    │   WHERE id = ?
    │
    ├── Preserve aggregated data
    │   - XP totals (anonymized)
    │   - Event counts (anonymized)
    │   - Community health metrics
    │
    ├── Delete personal data
    │   - Session records
    │   - Consent records
    │   - Notification preferences
    │
    ├── Remove from communities
    │   UPDATE members SET
    │     status = 'deleted',
    │     identity_id = NULL
    │   WHERE identity_id = ?
    │
    └── Send confirmation
        - Final WhatsApp message
        - Email confirmation if available
```

---

## 15. Data Anonymization and Final Termination

### 15.1 Anonymization Rules

| Data Type | Anonymization Method | Retention |
|-----------|---------------------|-----------|
| Phone number | SHA-256 hash | Indefinite (for uniqueness) |
| Display name | Replace with "Deleted User" | Indefinite |
| Avatar | Delete | - |
| Bio | Delete | - |
| Messages | Preserve (no PII) | Per retention policy |
| XP/Level | Preserve (anonymized) | Indefinite |
| Events | Preserve (anonymized) | Per retention policy |

### 15.2 Final Termination Checklist

| Step | Verification | Reversible |
|------|--------------|------------|
| Sessions invalidated | Redis check | No |
| Profile anonymized | DB check | No |
| PII removed | DB check | No |
| Aggregate data preserved | Analytics check | N/A |
| Confirmation sent | Log check | N/A |
| Audit log created | Log check | N/A |

### 15.3 LGPD Compliance

| Right | Implementation |
|-------|----------------|
| Right to access | Data export endpoint |
| Right to deletion | Account deletion flow |
| Right to rectification | Profile edit |
| Right to portability | JSON export |
| Right to object | Consent management |

---

## Appendix: Quick Reference

### State Transitions Summary

```
Identity: created → active → suspended → active
                           → deleted

Member: invited → active → inactive → removed
                        → deleted

Session: created → active → expired
                         → revoked
```

### API Response Codes

| Code | Meaning | Client Action |
|------|---------|---------------|
| 200 | Success | Process response |
| 201 | Created | Process response |
| 400 | Bad request | Fix input, retry |
| 401 | Unauthorized | Re-authenticate |
| 403 | Forbidden | Show error |
| 404 | Not found | Show error |
| 429 | Rate limited | Wait and retry |
| 500 | Server error | Retry with backoff |

---

*This document is the authoritative specification for all system behavior. If code differs from this document, the code is incorrect.*

<!-- Last Reviewed: 2026-01-17 - No updates needed -->
