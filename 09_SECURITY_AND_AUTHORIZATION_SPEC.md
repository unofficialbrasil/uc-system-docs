# Security and Authorization Specification

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## 1. Authentication Flows

### 1.1 Authentication Overview

The system uses **phone-based OTP authentication** as the sole authentication method. No passwords are stored.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│  │  Phone   │───▶│   OTP    │───▶│  Verify  │───▶│  Session │      │
│  │  Input   │    │   Send   │    │   OTP    │    │  Create  │      │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘      │
│                        │                               │            │
│                        ▼                               ▼            │
│                 ┌──────────┐                   ┌──────────┐        │
│                 │  Redis   │                   │  MySQL   │        │
│                 │  (OTP)   │                   │ (Session)│        │
│                 └──────────┘                   └──────────┘        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Signup Flow

```typescript
// Step 1: Initiate Signup
POST /api/auth/signup/start
{
  phone: "+5511999999999"
}

// System Actions:
// 1. Validate phone format (E.164)
// 2. Check if phone already registered → redirect to login
// 3. Generate 6-digit OTP
// 4. Store in Redis: otp:signup:{phone} → {otp, attempts: 0} (TTL: 5 min)
// 5. Send OTP via WhatsApp API
// 6. Return pending state

// Step 2: Verify OTP
POST /api/auth/signup/verify
{
  phone: "+5511999999999",
  otp: "123456"
}

// System Actions:
// 1. Retrieve OTP from Redis
// 2. Compare OTP (constant-time comparison)
// 3. If mismatch: increment attempts, max 3 → lock for 15 min
// 4. If match: delete OTP from Redis
// 5. Create Identity record
// 6. Create Session
// 7. Return session token + redirect to onboarding
```

### 1.3 Login Flow

```typescript
// Step 1: Initiate Login
POST /api/auth/login/start
{
  phone: "+5511999999999"
}

// System Actions:
// 1. Validate phone exists in system
// 2. Check account status (not suspended/deleted)
// 3. Generate 6-digit OTP
// 4. Store in Redis: otp:login:{phone} → {otp, attempts: 0} (TTL: 5 min)
// 5. Send OTP via WhatsApp API
// 6. Return pending state

// Step 2: Verify OTP
POST /api/auth/login/verify
{
  phone: "+5511999999999",
  otp: "123456"
}

// System Actions:
// 1. Verify OTP (same as signup)
// 2. Update Identity.last_active_at
// 3. Create new Session
// 4. Invalidate old sessions (optional: configurable)
// 5. Return session token
```

### 1.4 Session Management

```typescript
interface Session {
  session_id: string;      // UUID v4
  identity_id: number;     // FK to Identity
  token_hash: string;      // SHA-256 of session token
  created_at: Date;
  expires_at: Date;        // created_at + 7 days
  last_activity: Date;
  ip_address: string;
  user_agent: string;
}

// Session Token Structure
// Plain token: "uc_sess_" + base64(random 32 bytes)
// Stored: SHA-256(token) - never store plain tokens

// Session Validation (every request)
async function validateSession(token: string): Promise<Identity | null> {
  const hash = sha256(token);

  // Check Redis cache first
  const cached = await redis.get(`session:${hash}`);
  if (cached) {
    await redis.expire(`session:${hash}`, 3600); // Extend TTL
    return JSON.parse(cached);
  }

  // Fallback to database
  const session = await db.session.findUnique({
    where: { token_hash: hash },
    include: { identity: true }
  });

  if (!session || session.expires_at < new Date()) {
    return null;
  }

  // Cache in Redis
  await redis.setex(`session:${hash}`, 3600, JSON.stringify(session.identity));

  return session.identity;
}
```

---

## 2. Authorization Logic

### 2.1 Request Authorization Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTHORIZATION PIPELINE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Request                                                            │
│      │                                                               │
│      ▼                                                               │
│  ┌──────────────┐                                                   │
│  │ 1. Session   │──▶ Invalid? ──▶ 401 Unauthorized                 │
│  │   Validation │                                                    │
│  └──────┬───────┘                                                   │
│         │ Valid                                                      │
│         ▼                                                            │
│  ┌──────────────┐                                                   │
│  │ 2. Account   │──▶ Suspended? ──▶ 403 Forbidden                  │
│  │   Status     │                                                    │
│  └──────┬───────┘                                                   │
│         │ Active                                                     │
│         ▼                                                            │
│  ┌──────────────┐                                                   │
│  │ 3. Resource  │──▶ No Access? ──▶ 403 Forbidden                  │
│  │   Permission │                                                    │
│  └──────┬───────┘                                                   │
│         │ Permitted                                                  │
│         ▼                                                            │
│  ┌──────────────┐                                                   │
│  │ 4. Rate      │──▶ Exceeded? ──▶ 429 Too Many Requests           │
│  │   Limiting   │                                                    │
│  └──────┬───────┘                                                   │
│         │ OK                                                         │
│         ▼                                                            │
│     PROCEED                                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Endpoint Authorization Matrix

| Endpoint Pattern | Auth Required | Roles Allowed | Rate Limit |
|-----------------|---------------|---------------|------------|
| `GET /api/health` | No | Any | None |
| `POST /api/auth/*` | No | Any | 10/min |
| `GET /api/communities` | Yes | Any Member | 60/min |
| `POST /api/communities` | Yes | Any Authenticated | 5/hour |
| `PUT /api/communities/:id` | Yes | Owner, Admin | 30/min |
| `DELETE /api/communities/:id` | Yes | Owner Only | 1/hour |
| `GET /api/gamification/*` | Yes | Self Only | 120/min |
| `POST /api/admin/*` | Yes | System Admin | 30/min |
| `GET /api/internal/*` | Service Auth | Internal Services | None |

### 2.3 Resource-Level Authorization

```typescript
// Authorization check for community resources
async function authorizeResourceAccess(
  identity: Identity,
  communityId: number,
  requiredRole: MemberRole,
  action: string
): Promise<boolean> {
  // 1. Get membership
  const membership = await db.membership.findUnique({
    where: {
      identity_id_community_id: {
        identity_id: identity.id,
        community_id: communityId
      }
    }
  });

  if (!membership) {
    throw new ForbiddenError(`No membership in community ${communityId}`);
  }

  // 2. Check role hierarchy
  const roleHierarchy: Record<MemberRole, number> = {
    'member': 1,
    'moderator': 2,
    'admin': 3,
    'owner': 4
  };

  if (roleHierarchy[membership.role] < roleHierarchy[requiredRole]) {
    throw new ForbiddenError(`Role ${requiredRole} required for ${action}`);
  }

  // 3. Check suspension status
  if (membership.is_suspended) {
    throw new ForbiddenError('Membership is suspended');
  }

  return true;
}
```

---

## 3. RBAC Model

### 3.1 Role Definitions

```typescript
enum MemberRole {
  MEMBER = 'member',
  MODERATOR = 'moderator',
  ADMIN = 'admin',
  OWNER = 'owner'
}

interface RoleCapabilities {
  role: MemberRole;
  permissions: Permission[];
  inherits_from: MemberRole | null;
}

const ROLE_DEFINITIONS: RoleCapabilities[] = [
  {
    role: MemberRole.MEMBER,
    inherits_from: null,
    permissions: [
      'community:view',
      'community:leave',
      'gamification:view_own',
      'gamification:earn_xp',
      'world:connect',
      'world:move',
      'profile:view_own',
      'profile:edit_own'
    ]
  },
  {
    role: MemberRole.MODERATOR,
    inherits_from: MemberRole.MEMBER,
    permissions: [
      'member:view_list',
      'member:warn',
      'member:mute',
      'gamification:view_community',
      'world:kick_user'
    ]
  },
  {
    role: MemberRole.ADMIN,
    inherits_from: MemberRole.MODERATOR,
    permissions: [
      'member:remove',
      'member:suspend',
      'community:edit_settings',
      'community:manage_invites',
      'role:assign_moderator',
      'gamification:reset_missions'
    ]
  },
  {
    role: MemberRole.OWNER,
    inherits_from: MemberRole.ADMIN,
    permissions: [
      'community:delete',
      'community:transfer_ownership',
      'role:assign_admin',
      'role:revoke_admin',
      'webhooks:configure',
      'all_permissions'
    ]
  }
];
```

### 3.2 Permission Catalog

| Permission | Description | Roles |
|------------|-------------|-------|
| `community:view` | View community details | All |
| `community:leave` | Leave the community | All |
| `community:edit_settings` | Modify community settings | Admin, Owner |
| `community:delete` | Delete the community | Owner |
| `community:transfer_ownership` | Transfer to another member | Owner |
| `community:manage_invites` | Create/revoke invites | Admin, Owner |
| `member:view_list` | View all members | Moderator+ |
| `member:warn` | Issue warning to member | Moderator+ |
| `member:mute` | Temporarily mute member | Moderator+ |
| `member:remove` | Remove from community | Admin, Owner |
| `member:suspend` | Suspend membership | Admin, Owner |
| `role:assign_moderator` | Promote to moderator | Admin, Owner |
| `role:assign_admin` | Promote to admin | Owner |
| `role:revoke_admin` | Demote admin | Owner |
| `gamification:view_own` | View own gamification data | All |
| `gamification:view_community` | View leaderboards | Moderator+ |
| `gamification:earn_xp` | Earn XP from activities | All |
| `gamification:reset_missions` | Reset member missions | Admin, Owner |
| `world:connect` | Connect to UC World | All |
| `world:move` | Move avatar in world | All |
| `world:kick_user` | Kick user from world | Moderator+ |
| `profile:view_own` | View own profile | All |
| `profile:edit_own` | Edit own profile | All |
| `webhooks:configure` | Setup webhook integrations | Owner |

---

## 4. Permission Inheritance

### 4.1 Inheritance Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PERMISSION INHERITANCE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         ┌─────────┐                                 │
│                         │  OWNER  │                                 │
│                         │ (all)   │                                 │
│                         └────┬────┘                                 │
│                              │ inherits                             │
│                              ▼                                       │
│                         ┌─────────┐                                 │
│                         │  ADMIN  │                                 │
│                         │ (+admin)│                                 │
│                         └────┬────┘                                 │
│                              │ inherits                             │
│                              ▼                                       │
│                       ┌───────────┐                                 │
│                       │ MODERATOR │                                 │
│                       │  (+mod)   │                                 │
│                       └─────┬─────┘                                 │
│                             │ inherits                              │
│                             ▼                                        │
│                        ┌────────┐                                   │
│                        │ MEMBER │                                   │
│                        │ (base) │                                   │
│                        └────────┘                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 Permission Resolution

```typescript
function resolvePermissions(role: MemberRole): Set<Permission> {
  const permissions = new Set<Permission>();
  let currentRole: MemberRole | null = role;

  while (currentRole) {
    const definition = ROLE_DEFINITIONS.find(r => r.role === currentRole);
    if (!definition) break;

    // Add all permissions for this role
    definition.permissions.forEach(p => permissions.add(p));

    // Move to parent role
    currentRole = definition.inherits_from;
  }

  return permissions;
}

// Check permission with inheritance
function hasPermission(
  membership: Membership,
  permission: Permission
): boolean {
  const effectivePermissions = resolvePermissions(membership.role);

  // Owner has all permissions
  if (effectivePermissions.has('all_permissions')) {
    return true;
  }

  return effectivePermissions.has(permission);
}
```

### 4.3 Cross-Community Isolation

```typescript
// Permissions are ALWAYS scoped to a single community
// A user being Owner in Community A has NO special access to Community B

interface AuthorizationContext {
  identity: Identity;
  community_id: number;
  membership: Membership;  // Specific to this community
  permissions: Set<Permission>;  // Resolved for this community
}

// IMPORTANT: Always verify community context
async function authorizeAction(
  identityId: number,
  communityId: number,
  action: Permission
): Promise<void> {
  const membership = await getMembership(identityId, communityId);

  if (!membership) {
    throw new ForbiddenError('Not a member of this community');
  }

  if (!hasPermission(membership, action)) {
    throw new ForbiddenError(`Permission denied: ${action}`);
  }
}
```

---

## 5. Token Lifecycle

### 5.1 Token Types

| Token Type | Purpose | Storage | TTL | Revocable |
|------------|---------|---------|-----|-----------|
| OTP Code | One-time password | Redis | 5 min | Auto-expire |
| Session Token | API authentication | Redis + MySQL | 7 days | Yes |
| Invite Code | Community invites | MySQL | 24h or 1 use | Yes |
| Service Token | Internal API calls | Environment | Never | Config change |

### 5.2 Session Token Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SESSION TOKEN LIFECYCLE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐                                                        │
│  │ CREATE  │ Login/Signup success                                   │
│  └────┬────┘                                                        │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────┐    Activity    ┌─────────┐                            │
│  │ ACTIVE  │◀──────────────▶│ REFRESH │                            │
│  └────┬────┘    (extend)    └─────────┘                            │
│       │                                                              │
│       ├────────────────────┬────────────────────┐                   │
│       │                    │                    │                   │
│       ▼                    ▼                    ▼                   │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐              │
│  │ EXPIRED │         │ REVOKED │         │ REPLACED│              │
│  │ (7 days)│         │ (logout)│         │(new login)             │
│  └─────────┘         └─────────┘         └─────────┘              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.3 Token Security Measures

```typescript
// Token Generation
function generateSessionToken(): { token: string; hash: string } {
  const randomBytes = crypto.randomBytes(32);
  const token = `uc_sess_${randomBytes.toString('base64url')}`;
  const hash = crypto.createHash('sha256').update(token).digest('hex');

  return { token, hash };
}

// Token Storage Rules:
// 1. NEVER store plain tokens in database
// 2. ALWAYS use SHA-256 hash for storage/lookup
// 3. Return plain token to client ONCE at creation
// 4. Client stores token in httpOnly cookie (frontend) or secure storage

// Cookie Configuration
const SESSION_COOKIE_CONFIG = {
  httpOnly: true,
  secure: true,          // HTTPS only
  sameSite: 'lax',       // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
  path: '/',
  domain: '.unofficialbrasil.com.br'
};

// Token Rotation (on sensitive operations)
async function rotateSessionToken(oldTokenHash: string): Promise<string> {
  const { token: newToken, hash: newHash } = generateSessionToken();

  await db.$transaction([
    // Invalidate old token
    db.session.update({
      where: { token_hash: oldTokenHash },
      data: { revoked_at: new Date() }
    }),
    // Create new session
    db.session.create({
      data: {
        /* ... copy session data with new hash ... */
        token_hash: newHash
      }
    })
  ]);

  // Clear Redis cache for old token
  await redis.del(`session:${oldTokenHash}`);

  return newToken;
}
```

### 5.4 OTP Security

```typescript
const OTP_CONFIG = {
  length: 6,
  digits_only: true,
  ttl_seconds: 300,        // 5 minutes
  max_attempts: 3,
  lockout_seconds: 900,    // 15 minutes after max attempts
  rate_limit: {
    per_phone: 3,          // Max 3 OTPs per phone per hour
    window_seconds: 3600
  }
};

async function generateOTP(phone: string, purpose: 'signup' | 'login'): Promise<void> {
  // Check rate limit
  const rateLimitKey = `otp_rate:${phone}`;
  const attempts = await redis.incr(rateLimitKey);

  if (attempts === 1) {
    await redis.expire(rateLimitKey, OTP_CONFIG.rate_limit.window_seconds);
  }

  if (attempts > OTP_CONFIG.rate_limit.per_phone) {
    throw new RateLimitError('Too many OTP requests. Try again later.');
  }

  // Generate OTP
  const otp = crypto.randomInt(100000, 999999).toString();

  // Store with TTL
  await redis.setex(
    `otp:${purpose}:${phone}`,
    OTP_CONFIG.ttl_seconds,
    JSON.stringify({ otp, attempts: 0 })
  );

  // Send via WhatsApp
  await whatsappService.sendOTP(phone, otp);
}

async function verifyOTP(
  phone: string,
  purpose: 'signup' | 'login',
  inputOtp: string
): Promise<boolean> {
  const key = `otp:${purpose}:${phone}`;
  const data = await redis.get(key);

  if (!data) {
    throw new InvalidOTPError('OTP expired or not found');
  }

  const { otp, attempts } = JSON.parse(data);

  // Check lockout
  if (attempts >= OTP_CONFIG.max_attempts) {
    throw new LockedOutError('Too many failed attempts. Request new OTP.');
  }

  // Constant-time comparison to prevent timing attacks
  const isValid = crypto.timingSafeEqual(
    Buffer.from(inputOtp),
    Buffer.from(otp)
  );

  if (!isValid) {
    // Increment attempts
    await redis.setex(
      key,
      OTP_CONFIG.ttl_seconds,
      JSON.stringify({ otp, attempts: attempts + 1 })
    );
    throw new InvalidOTPError('Incorrect OTP');
  }

  // Delete OTP on success
  await redis.del(key);
  return true;
}
```

---

## 6. Abuse and Threat Scenarios

### 6.1 Threat Model

| Threat | Description | Severity | Mitigation |
|--------|-------------|----------|------------|
| T1: Credential Stuffing | Automated login attempts | High | Rate limiting, OTP-only auth |
| T2: Session Hijacking | Stealing session tokens | Critical | HttpOnly cookies, token rotation |
| T3: IDOR | Accessing other users' data | High | Resource-level authorization |
| T4: Privilege Escalation | Gaining unauthorized roles | Critical | Server-side role validation |
| T5: OTP Brute Force | Guessing OTP codes | Medium | 3-attempt limit, lockout |
| T6: Account Takeover | Compromising phone number | Critical | Out of scope (telco security) |
| T7: CSRF | Cross-site request forgery | Medium | SameSite cookies, CSRF tokens |
| T8: XSS | Script injection | High | CSP headers, input sanitization |
| T9: SQL Injection | Database manipulation | Critical | Parameterized queries (Prisma) |
| T10: API Abuse | Excessive API calls | Medium | Rate limiting per identity |

### 6.2 Security Controls

```typescript
// Rate Limiting Configuration
const RATE_LIMITS = {
  // Global limits
  global: {
    requests_per_second: 100,
    burst: 200
  },

  // Per-endpoint limits
  endpoints: {
    'auth/login': { rpm: 10, burst: 3 },
    'auth/signup': { rpm: 5, burst: 2 },
    'api/communities': { rpm: 60, burst: 20 },
    'api/gamification': { rpm: 120, burst: 30 },
    'world/connect': { rpm: 10, burst: 5 }
  },

  // Per-identity limits
  identity: {
    requests_per_minute: 300,
    concurrent_sessions: 5
  }
};

// Rate Limit Middleware
async function rateLimitMiddleware(req: Request, res: Response, next: Function) {
  const identityId = req.session?.identity_id || req.ip;
  const endpoint = req.path;

  const key = `ratelimit:${endpoint}:${identityId}`;
  const limit = RATE_LIMITS.endpoints[endpoint] || RATE_LIMITS.global;

  const current = await redis.incr(key);
  if (current === 1) {
    await redis.expire(key, 60);
  }

  if (current > limit.rpm) {
    res.setHeader('Retry-After', 60);
    throw new RateLimitError('Rate limit exceeded');
  }

  next();
}
```

### 6.3 Security Headers

```typescript
// Security Headers Configuration
const SECURITY_HEADERS = {
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Content-Security-Policy': [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' 'unsafe-eval'",  // Required for Next.js
    "style-src 'self' 'unsafe-inline'",
    "img-src 'self' data: blob: https:",
    "font-src 'self' data:",
    "connect-src 'self' wss: https:",
    "frame-ancestors 'none'",
    "base-uri 'self'",
    "form-action 'self'"
  ].join('; '),
  'Permissions-Policy': 'geolocation=(), microphone=(), camera=()'
};
```

### 6.4 Abuse Detection

```typescript
// Suspicious Activity Detection
interface SuspiciousActivity {
  type: 'rapid_requests' | 'auth_failures' | 'role_enumeration' | 'data_scraping';
  identity_id?: number;
  ip_address: string;
  details: Record<string, unknown>;
}

async function detectAbuse(activity: SuspiciousActivity): Promise<void> {
  const key = `abuse:${activity.type}:${activity.ip_address}`;
  const count = await redis.incr(key);

  if (count === 1) {
    await redis.expire(key, 3600); // 1 hour window
  }

  // Thresholds for different abuse types
  const thresholds: Record<string, number> = {
    'rapid_requests': 100,
    'auth_failures': 10,
    'role_enumeration': 5,
    'data_scraping': 50
  };

  if (count >= thresholds[activity.type]) {
    // Log security event
    await logSecurityEvent('abuse_detected', activity);

    // Temporary IP block
    await redis.setex(`blocked:ip:${activity.ip_address}`, 3600, '1');

    // Alert if threshold significantly exceeded
    if (count >= thresholds[activity.type] * 2) {
      await alertSecurityTeam(activity);
    }
  }
}
```

### 6.5 Account Suspension

```typescript
// Account Suspension Levels
enum SuspensionLevel {
  WARNING = 'warning',         // No access restriction, visible warning
  TEMPORARY = 'temporary',     // Restricted access for N days
  PERMANENT = 'permanent'      // Full access revocation
}

interface Suspension {
  identity_id: number;
  community_id?: number;       // NULL = global suspension
  level: SuspensionLevel;
  reason: string;
  suspended_by: number;        // Admin identity_id
  suspended_at: Date;
  expires_at?: Date;           // NULL for permanent
  appeal_status?: 'pending' | 'approved' | 'denied';
}

async function suspendUser(
  adminId: number,
  targetId: number,
  communityId: number | null,
  level: SuspensionLevel,
  reason: string,
  durationDays?: number
): Promise<void> {
  // Verify admin permissions
  if (communityId) {
    await authorizeAction(adminId, communityId, 'member:suspend');
  } else {
    // Global suspension requires system admin
    await verifySystemAdmin(adminId);
  }

  // Create suspension record
  await db.suspension.create({
    data: {
      identity_id: targetId,
      community_id: communityId,
      level,
      reason,
      suspended_by: adminId,
      expires_at: durationDays
        ? new Date(Date.now() + durationDays * 86400000)
        : null
    }
  });

  // Revoke all active sessions
  await revokeAllSessions(targetId);

  // Log security event
  await logSecurityEvent('user_suspended', {
    admin_id: adminId,
    target_id: targetId,
    community_id: communityId,
    level,
    reason
  });
}
```

---

## 7. Security Audit Log

### 7.1 Logged Events

| Event | When Logged | Data Captured |
|-------|-------------|---------------|
| `auth.login.success` | Successful login | identity_id, ip, user_agent |
| `auth.login.failure` | Failed login | phone_hash, ip, reason |
| `auth.logout` | User logout | identity_id, session_id |
| `auth.session.created` | New session | identity_id, ip, user_agent |
| `auth.session.revoked` | Session invalidated | identity_id, by_whom, reason |
| `permission.denied` | Authorization failure | identity_id, resource, action |
| `role.changed` | Role assignment | identity_id, old_role, new_role, by_whom |
| `suspension.created` | User suspended | identity_id, level, reason, by_whom |
| `data.exported` | Data export request | identity_id, data_types |
| `data.deleted` | Data deletion | identity_id, scope |

### 7.2 Audit Log Schema

```sql
CREATE TABLE security_audit_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    event_type VARCHAR(50) NOT NULL,
    severity ENUM('info', 'warning', 'critical') NOT NULL,
    timestamp TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    identity_id INT NULL,
    ip_address VARCHAR(45) NOT NULL,
    user_agent TEXT NULL,
    resource_type VARCHAR(50) NULL,
    resource_id VARCHAR(100) NULL,
    action VARCHAR(50) NOT NULL,
    outcome ENUM('success', 'failure', 'blocked') NOT NULL,
    details JSON NULL,

    INDEX idx_event_time (event_type, timestamp),
    INDEX idx_identity (identity_id, timestamp),
    INDEX idx_ip (ip_address, timestamp),
    INDEX idx_severity (severity, timestamp)
) ENGINE=InnoDB;
```

---

*This document defines the security boundaries of the system. All authentication and authorization logic must adhere to these specifications.*
