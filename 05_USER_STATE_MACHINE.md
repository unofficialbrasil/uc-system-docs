# User State Machine

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 1.1.0

---

## Purpose

This document defines all possible user states, allowed transitions, and system responses for the Unofficial Communities platform.

---

## 1. Identity States

### 1.1 State Diagram

```
                              ┌─────────────────┐
                              │                 │
                              │   [NOT_EXIST]   │
                              │                 │
                              └────────┬────────┘
                                       │
                                       │ signup_initiated
                                       ▼
                              ┌─────────────────┐
                              │                 │
                              │    PENDING      │
                              │  (OTP sent)     │
                              │                 │
                              └────────┬────────┘
                                       │
                       ┌───────────────┼───────────────┐
                       │               │               │
                       │ otp_failed    │ otp_verified  │ otp_expired
                       │ (3x)          │               │
                       ▼               ▼               ▼
              ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
              │   LOCKED    │ │   ACTIVE    │ │   PENDING   │
              │ (15 min)    │ │             │ │ (restart)   │
              └──────┬──────┘ └──────┬──────┘ └─────────────┘
                     │               │
                     │ lockout_      │
                     │ expired       │
                     ▼               │
              ┌─────────────┐        │
              │   PENDING   │◄───────┘
              │ (can retry) │        │
              └─────────────┘        │
                                     │
                     ┌───────────────┼───────────────┐
                     │               │               │
                     │ admin_suspend │ user_delete   │ admin_ban
                     │               │               │
                     ▼               ▼               ▼
              ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
              │  SUSPENDED  │ │   DELETED   │ │   BANNED    │
              │             │ │             │ │             │
              └──────┬──────┘ └─────────────┘ └─────────────┘
                     │           (terminal)     (terminal)
                     │
                     │ appeal_approved
                     ▼
              ┌─────────────┐
              │   ACTIVE    │
              │             │
              └─────────────┘
```

### 1.2 State Definitions

| State | Code | Description | Database Value |
|-------|------|-------------|----------------|
| NOT_EXIST | - | No identity record exists | N/A |
| PENDING | `pending` | OTP sent, awaiting verification | `status = 'pending'` |
| LOCKED | `locked` | Temporarily locked due to failed attempts | `status = 'locked'` |
| ACTIVE | `active` | Normal operating state | `status = 'active'` |
| SUSPENDED | `suspended` | Temporarily disabled by admin | `status = 'suspended'` |
| DELETED | `deleted` | User-requested deletion (anonymized) | `status = 'deleted'` |
| BANNED | `banned` | Permanently disabled by admin | `status = 'banned'` |

### 1.3 Transition Rules

| From | To | Event | Conditions | Side Effects |
|------|-----|-------|------------|--------------|
| NOT_EXIST | PENDING | `signup_initiated` | Valid phone format | Create identity record, send OTP |
| PENDING | ACTIVE | `otp_verified` | Correct OTP within TTL | Create session, log event |
| PENDING | PENDING | `otp_expired` | OTP TTL exceeded | Clear OTP, allow retry |
| PENDING | LOCKED | `otp_failed` | 3 consecutive failures | Set lockout_until timestamp |
| LOCKED | PENDING | `lockout_expired` | Current time > lockout_until | Clear lockout |
| ACTIVE | SUSPENDED | `admin_suspend` | Admin action | Invalidate sessions, notify user |
| SUSPENDED | ACTIVE | `appeal_approved` | Admin action | Log restoration |
| ACTIVE | DELETED | `user_delete` | User request + cooling period | Anonymize data |
| ACTIVE | BANNED | `admin_ban` | Admin action (severe violation) | Invalidate sessions, log |
| SUSPENDED | BANNED | `admin_ban` | Admin action | Log escalation |

### 1.4 Forbidden Transitions

| From | To | Reason |
|------|-----|--------|
| DELETED | Any | Terminal state, data anonymized |
| BANNED | Any | Terminal state, requires new identity |
| PENDING | DELETED | Cannot delete unverified account |
| NOT_EXIST | ACTIVE | Must go through PENDING |

---

## 2. Membership States

### 2.1 State Diagram

```
                              ┌─────────────────┐
                              │                 │
                              │   [NOT_MEMBER]  │
                              │                 │
                              └────────┬────────┘
                                       │
                       ┌───────────────┼───────────────┐
                       │               │               │
                       │ join_request  │ invited       │ direct_join
                       │               │               │ (public)
                       ▼               ▼               ▼
              ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
              │  REQUESTED  │ │  INVITED    │ │   ACTIVE    │
              │             │ │             │ │             │
              └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
                     │               │               │
           ┌─────────┼─────────┐     │               │
           │         │         │     │               │
           │approved │rejected │     │accept         │
           │         │         │     │               │
           ▼         ▼         ▼     ▼               │
    ┌───────────┐ ┌───────┐ ┌─────────────┐         │
    │  ACTIVE   │ │REJECTED│ │   ACTIVE    │◄────────┘
    │           │ │        │ │             │
    └─────┬─────┘ └────────┘ └──────┬──────┘
          │       (terminal)        │
          │                         │
          │        ┌────────────────┼────────────────┐
          │        │                │                │
          │        │ user_leave     │ admin_remove   │ inactivity
          │        │                │                │ (90 days)
          │        ▼                ▼                ▼
          │   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
          │   │    LEFT     │ │   REMOVED   │ │  INACTIVE   │
          │   │             │ │             │ │             │
          │   └─────────────┘ └─────────────┘ └──────┬──────┘
          │                                          │
          │                                          │ activity
          │                                          │ detected
          │                                          ▼
          └──────────────────────────────────► ┌─────────────┐
                      rejoin                   │   ACTIVE    │
                                               │             │
                                               └─────────────┘
```

### 2.2 State Definitions

| State | Code | Description | Permissions |
|-------|------|-------------|-------------|
| NOT_MEMBER | - | No membership record | View public info only |
| REQUESTED | `requested` | Pending approval | View public info only |
| INVITED | `invited` | Pending acceptance | View public info only |
| ACTIVE | `active` | Full member | Full community access |
| INACTIVE | `inactive` | Dormant (no activity 90d) | Read-only access |
| LEFT | `left` | User voluntarily left | None (can rejoin) |
| REMOVED | `removed` | Admin removed | None (may appeal) |
| REJECTED | `rejected` | Join request denied | None |

### 2.3 Transition Rules

| From | To | Event | Actor | Side Effects |
|------|-----|-------|-------|--------------|
| NOT_MEMBER | REQUESTED | `join_request` | User | Notify admins |
| NOT_MEMBER | INVITED | `invited` | Admin | Notify user |
| NOT_MEMBER | ACTIVE | `direct_join` | User | Public community only |
| REQUESTED | ACTIVE | `approved` | Admin | Initialize gamification |
| REQUESTED | REJECTED | `rejected` | Admin | Notify user |
| INVITED | ACTIVE | `accept` | User | Initialize gamification |
| ACTIVE | INACTIVE | `inactivity` | System | After 90 days |
| INACTIVE | ACTIVE | `activity_detected` | System | Auto-reactivate |
| ACTIVE | LEFT | `user_leave` | User | Preserve data |
| ACTIVE | REMOVED | `admin_remove` | Admin | Notify user |
| LEFT | ACTIVE | `rejoin` | User | Restore gamification |

### 2.4 Membership Roles

| Role | Code | Permissions |
|------|------|-------------|
| Member | `member` | Participate, view content |
| Moderator | `moderator` | + Remove content, warn users |
| Admin | `admin` | + Remove members, manage settings |
| Owner | `owner` | + Delete community, transfer ownership |

---

## 3. Session States

### 3.1 State Diagram

```
              ┌─────────────────┐
              │                 │
              │   [NO_SESSION]  │
              │                 │
              └────────┬────────┘
                       │
                       │ login_success
                       ▼
              ┌─────────────────┐
              │                 │
              │     ACTIVE      │◄──────────┐
              │                 │           │
              └────────┬────────┘           │
                       │                    │
       ┌───────────────┼───────────────┐    │
       │               │               │    │
       │ logout        │ expired       │    │ refresh
       │               │ (7 days)      │    │ (sliding)
       ▼               ▼               │    │
┌─────────────┐ ┌─────────────┐       │    │
│  TERMINATED │ │   EXPIRED   │       │    │
│             │ │             │───────┼────┘
└─────────────┘ └─────────────┘       │
                                      │
                       │ revoked      │
                       │ (security)   │
                       ▼              │
              ┌─────────────────┐     │
              │    REVOKED      │─────┘
              │                 │ (must re-login)
              └─────────────────┘
```

### 3.2 State Definitions

| State | Redis Key Exists | Cookie Valid | User Access |
|-------|------------------|--------------|-------------|
| NO_SESSION | No | N/A | None |
| ACTIVE | Yes | Yes | Full |
| EXPIRED | No (TTL) | Invalid | None |
| TERMINATED | No (deleted) | Invalid | None |
| REVOKED | No (deleted) | Invalid | None |

### 3.3 Session Sliding Window

```
Every authenticated request:
    IF session exists in Redis:
        EXPIRE uc:sess:<id> 604800  # Reset 7-day TTL
        Session remains ACTIVE

IF no activity for 7 days:
    Redis key auto-expires
    Session becomes EXPIRED
```

---

## 4. Onboarding States

### 4.1 State Diagram

```
              ┌─────────────────┐
              │                 │
              │    NOT_STARTED  │
              │                 │
              └────────┬────────┘
                       │
                       │ begin_signup
                       ▼
              ┌─────────────────┐
              │                 │
              │   PHONE_SENT    │
              │                 │
              └────────┬────────┘
                       │
                       │ otp_verified
                       ▼
              ┌─────────────────┐
              │                 │
              │ PHONE_VERIFIED  │
              │                 │
              └────────┬────────┘
                       │
                       │ profile_saved
                       ▼
              ┌─────────────────┐
              │                 │
              │ PROFILE_CREATED │
              │                 │
              └────────┬────────┘
                       │
                       │ community_joined
                       ▼
              ┌─────────────────┐
              │                 │
              │   COMPLETED     │
              │                 │
              └─────────────────┘
```

### 4.2 Onboarding Progress Storage

```sql
CREATE TABLE onboarding_progress (
    identity_id INT PRIMARY KEY,
    state ENUM('not_started', 'phone_sent', 'phone_verified',
               'profile_created', 'completed'),
    started_at TIMESTAMP,
    phone_verified_at TIMESTAMP NULL,
    profile_created_at TIMESTAMP NULL,
    completed_at TIMESTAMP NULL,
    FOREIGN KEY (identity_id) REFERENCES identities(id)
);
```

### 4.3 State-Dependent UI

| State | Redirect | UI Elements |
|-------|----------|-------------|
| NOT_STARTED | `/signup` | Signup form |
| PHONE_SENT | `/signup/verify` | OTP input |
| PHONE_VERIFIED | `/signup/profile` | Profile form |
| PROFILE_CREATED | `/signup/community` | Community selection |
| COMPLETED | `/dashboard` | Full dashboard |

---

## 4A. Age Assurance States

### 4A.1 State Diagram

```
              ┌─────────────────┐
              │                 │
              │  UNVERIFIED     │  assurance_level = 0
              │  (no DOB data)  │
              │                 │
              └────────┬────────┘
                       │
                       │ gate_a_presented
                       ▼
              ┌─────────────────┐
              │                 │
              │  PENDING_AGE    │  (transient state)
              │  (Gate A shown) │
              │                 │
              └────────┬────────┘
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       │ age_confirmed │ age_blocked   │ gate_abandoned
       │ (18+)         │ (under 18)    │
       ▼               ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ SELF_       │ │ BLOCKED     │ │ UNVERIFIED  │
│ DECLARED    │ │ (terminal)  │ │ (restart)   │
│ level = 1   │ │             │ │             │
└──────┬──────┘ └─────────────┘ └─────────────┘
       │
       │ risk_trigger
       ▼
┌─────────────────┐
│  PENDING_RECHECK │  (Gate B shown)
│                 │
└────────┬────────┘
         │
┌────────┼────────┐
│        │        │
│ passed │ failed │
│        │        │
▼        ▼        ▼
┌─────────────┐ ┌─────────────┐
│ REVALIDATED │ │ FLAGGED     │
│ level = 2   │ │ T&S review  │
└──────┬──────┘ └─────────────┘
       │
       │ verification_requested
       ▼
┌─────────────────┐
│  PENDING_       │  (ID verification)
│  VERIFICATION   │
└────────┬────────┘
         │
┌────────┼────────┐
│        │        │
│ passed │ failed │
▼        ▼        ▼
┌─────────────┐ ┌─────────────┐
│  VERIFIED   │ │ REVALIDATED │
│  level = 3  │ │ (no change) │
└─────────────┘ └─────────────┘
```

### 4A.2 State Definitions

| State | Assurance Level | Description | Allowed Actions |
|-------|-----------------|-------------|-----------------|
| UNVERIFIED | 0 | No DOB data collected | View landing page only |
| PENDING_AGE | - | Gate A in progress | Submit DOB |
| SELF_DECLARED | 1 | User-entered DOB | Full platform access |
| PENDING_RECHECK | - | Gate B in progress | Submit DOB |
| REVALIDATED | 2 | Confirmed via Gate B | + High-value features |
| PENDING_VERIFICATION | - | ID verification in progress | Wait for result |
| VERIFIED | 3 | ID-verified | + Monetization |
| BLOCKED | - | Under-18 detected | None (terminal) |
| FLAGGED | - | T&S review pending | Limited access |

### 4A.3 Transition Rules

| From | To | Event | Conditions | Side Effects |
|------|-----|-------|------------|--------------|
| UNVERIFIED | PENDING_AGE | `gate_a_presented` | Onboarding complete | Show Gate A UI |
| PENDING_AGE | SELF_DECLARED | `age_confirmed` | DOB shows 18+ | Store DOB, emit event |
| PENDING_AGE | BLOCKED | `age_blocked` | DOB shows under-18 | Emit event, show message |
| PENDING_AGE | UNVERIFIED | `gate_abandoned` | User closes/cancels | No storage |
| SELF_DECLARED | PENDING_RECHECK | `risk_trigger` | Risk signal detected | Show Gate B UI |
| PENDING_RECHECK | REVALIDATED | `passed` | DOB matches or ±1 year | Update level to 2 |
| PENDING_RECHECK | FLAGGED | `failed` | Major mismatch or now minor | Suspend, create T&S case |
| REVALIDATED | PENDING_VERIFICATION | `verification_requested` | Monetization requested | Initiate ID check |
| PENDING_VERIFICATION | VERIFIED | `passed` | ID valid, matches DOB | Update level to 3 |
| PENDING_VERIFICATION | REVALIDATED | `failed` | ID check failed | Keep level 2 |

### 4A.4 Age Band Classification

| Age Range | Age Band | Adult Status |
|-----------|----------|--------------|
| 0-12 | `under_13` | Minor (hard block) |
| 13-17 | `13_17` | Minor (soft block) |
| 18-24 | `18_24` | Adult |
| 25-34 | `25_34` | Adult |
| 35+ | `35_plus` | Adult |

---

## 5. Gamification States

### 5.1 Streak State Machine

```
              ┌─────────────────┐
              │                 │
              │   NO_STREAK     │ streak = 0
              │                 │
              └────────┬────────┘
                       │
                       │ first_activity
                       ▼
              ┌─────────────────┐
              │                 │
              │   BUILDING      │ streak >= 1
              │                 │
              └────────┬────────┘
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       │ miss_day      │ 7_days        │ 30_days
       │               │               │
       ▼               ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  NO_STREAK  │ │   BRONZE    │ │   SILVER    │
│  (reset)    │ │   streak=7  │ │  streak=30  │
└─────────────┘ └─────────────┘ └─────────────┘
                       │               │
                       │               │ 100_days
                       │               ▼
                       │        ┌─────────────┐
                       │        │    GOLD     │
                       │        │ streak=100  │
                       │        └─────────────┘
                       │
                       │ miss_day (from any)
                       ▼
              ┌─────────────────┐
              │                 │
              │   NO_STREAK     │
              │   (reset to 0)  │
              └─────────────────┘
```

### 5.2 Level Progression

```
Level 1 (Novato)
    │ xp >= 100
    ▼
Level 2 (Iniciante)
    │ xp >= 300
    ▼
Level 3 (Membro)
    │ xp >= 600
    ▼
Level 4 (Ativo)
    │ xp >= 1000
    ▼
Level 5 (Engajado)
    │ xp >= 1500
    ▼
Level 6 (Veterano)
    │ xp >= 2200
    ▼
Level 7 (Destaque)
    │ xp >= 3000
    ▼
Level 8 (Influenciador)
    │ xp >= 4000
    ▼
Level 9 (Lenda)
    │ xp >= 5500
    ▼
Level 10 (Lenda Suprema)
    │ (max level)
```

---

## 6. UC World Presence States

### 6.1 State Diagram

```
              ┌─────────────────┐
              │                 │
              │    OFFLINE      │
              │                 │
              └────────┬────────┘
                       │
                       │ websocket_connect
                       ▼
              ┌─────────────────┐
              │                 │
              │   CONNECTING    │
              │                 │
              └────────┬────────┘
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       │ auth_failed   │ auth_success  │
       │               │               │
       ▼               ▼               │
┌─────────────┐ ┌─────────────┐       │
│   OFFLINE   │ │    ONLINE   │◄──────┘
│  (error)    │ │  (in room)  │
└─────────────┘ └──────┬──────┘
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       │ disconnect    │ move_zone     │ idle_5min
       │               │               │
       ▼               ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   OFFLINE   │ │    ONLINE   │ │    IDLE     │
│             │ │(zone change)│ │             │
└─────────────┘ └─────────────┘ └──────┬──────┘
                                       │
                       ┌───────────────┼───────────────┐
                       │               │               │
                       │ activity      │ idle_15min    │
                       │               │               │
                       ▼               ▼               │
                ┌─────────────┐ ┌─────────────┐       │
                │    ONLINE   │ │    AWAY     │───────┘
                │             │ │             │ (kicked after 30min)
                └─────────────┘ └─────────────┘
```

### 6.2 Presence State Definitions

| State | Visible to Others | Can Interact | Server Resources |
|-------|-------------------|--------------|------------------|
| OFFLINE | No | No | None |
| CONNECTING | No | No | Minimal |
| ONLINE | Yes (avatar) | Yes | Full |
| IDLE | Yes (dimmed) | Yes | Full |
| AWAY | Yes (ghost) | Limited | Reduced |

---

## 7. System Response Matrix

### 7.1 By Identity State

| Action | PENDING | ACTIVE | SUSPENDED | DELETED | BANNED |
|--------|---------|--------|-----------|---------|--------|
| Login | OTP flow | Success | Blocked | Blocked | Blocked |
| View profile | - | Yes | - | - | - |
| Edit profile | - | Yes | - | - | - |
| Join community | - | Yes | - | - | - |
| Access dashboard | - | Yes | - | - | - |
| Delete account | - | Yes | - | - | - |

### 7.2 By Membership State

| Action | ACTIVE | INACTIVE | LEFT | REMOVED |
|--------|--------|----------|------|---------|
| View community | Yes | Read-only | No | No |
| Post/interact | Yes | No | No | No |
| Earn XP | Yes | No | No | No |
| View rankings | Yes | Yes | No | No |
| Rejoin | - | - | Yes | Appeal |

---

## 8. Event Triggers

### 8.1 Time-Based Triggers

| Event | Trigger | Check Frequency |
|-------|---------|-----------------|
| Session expiry | 7 days inactivity | On access |
| Lockout expiry | 15 minutes | On access |
| Inactivity flag | 90 days no activity | Daily job |
| Streak reset | Midnight without activity | Daily job |
| Deletion execution | 7 days after request | Daily job |

### 8.2 Action-Based Triggers

| Event | Trigger | Immediate Effect |
|-------|---------|------------------|
| OTP verified | Correct code entered | Create session |
| Mission completed | User action | Award XP |
| Level up | XP threshold crossed | Update level, emit event |
| Streak milestone | 7/30/100 days | Award bonus, emit event |
| Zone entered | Avatar position | Update presence |

---

*This document defines all valid state transitions. Any transition not explicitly listed is forbidden.*

<!-- Last Updated: 2026-01-17 - Added Section 4A: Age Assurance States -->
