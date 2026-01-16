# Environment and Configuration Registry

**System:** Unofficial Communities
**Last Updated:** 2026-01-16
**Version:** 1.1.0

---

## 1. Configuration Overview

### 1.1 Configuration Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONFIGURATION HIERARCHY                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │                    ENVIRONMENT VARIABLES                    │    │
│  │              (Highest Priority - Runtime Override)          │    │
│  └────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │                      .env FILES                             │    │
│  │              (Per-Environment Defaults)                     │    │
│  │     .env.production | .env.staging | .env.development      │    │
│  └────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │                    DEFAULT VALUES                           │    │
│  │              (Hardcoded in Application)                     │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Configuration Categories

| Category | Description | Example |
|----------|-------------|---------|
| Infrastructure | Database, Redis, ports | `DATABASE_URL`, `REDIS_URL` |
| Security | Keys, tokens, secrets | `JWT_SECRET`, `OTP_SECRET` |
| Integration | External API credentials | `META_API_TOKEN`, `WHATSAPP_PHONE_ID` |
| Feature | Feature flags | `FEATURE_UC_WORLD`, `FEATURE_GAMIFICATION` |
| Business | Business logic parameters | `XP_PER_MISSION`, `STREAK_BONUS_MULTIPLIER` |
| Observability | Logging, monitoring | `LOG_LEVEL`, `ENABLE_METRICS` |

---

## 2. Configuration Variables

### 2.1 Infrastructure Configuration

| Variable | Description | Type | Required | Default |
|----------|-------------|------|----------|---------|
| `NODE_ENV` | Runtime environment | `production` \| `staging` \| `development` | Yes | `development` |
| `PORT` | API server port | number | No | `3001` |
| `HOST` | API server host | string | No | `0.0.0.0` |
| `DATABASE_URL` | MySQL connection string | string | Yes | - |
| `REDIS_URL` | Redis connection string | string | Yes | - |
| `REDIS_HOST` | Redis host (alternative) | string | No | `localhost` |
| `REDIS_PORT` | Redis port | number | No | `6379` |
| `REDIS_PASSWORD` | Redis password | string | No | - |

### 2.2 Security Configuration

| Variable | Description | Type | Required | Default |
|----------|-------------|------|----------|---------|
| `SESSION_SECRET` | Session encryption key | string (32+ chars) | Yes | - |
| `OTP_SECRET` | OTP generation seed | string | Yes | - |
| `CORS_ORIGINS` | Allowed CORS origins | comma-separated | Yes | - |
| `RATE_LIMIT_WINDOW_MS` | Rate limit window | number | No | `60000` |
| `RATE_LIMIT_MAX` | Max requests per window | number | No | `100` |
| `SESSION_TTL_DAYS` | Session expiration | number | No | `7` |
| `OTP_TTL_SECONDS` | OTP expiration | number | No | `300` |
| `OTP_MAX_ATTEMPTS` | Max OTP verification attempts | number | No | `3` |

### 2.3 Integration Configuration

| Variable | Description | Type | Required | Default |
|----------|-------------|------|----------|---------|
| `META_API_VERSION` | Meta Graph API version | string | No | `v21.0` |
| `META_API_TOKEN` | Meta API access token | string | Yes (prod) | - |
| `META_APP_ID` | Meta App ID | string | Yes (prod) | - |
| `META_APP_SECRET` | Meta App secret | string | Yes (prod) | - |
| `WHATSAPP_PHONE_ID` | WhatsApp phone number ID | string | Yes (prod) | - |
| `WHATSAPP_BUSINESS_ID` | WhatsApp Business account ID | string | Yes (prod) | - |
| `WEBHOOK_VERIFY_TOKEN` | Webhook verification token | string | Yes | - |

### 2.4 Application Configuration

| Variable | Description | Type | Required | Default |
|----------|-------------|------|----------|---------|
| `NEXT_PUBLIC_API_URL` | Public API URL | URL | Yes | - |
| `NEXT_PUBLIC_WS_URL` | WebSocket URL | URL | Yes | - |
| `NEXT_PUBLIC_APP_NAME` | Application name | string | No | `Unofficial` |
| `NEXT_PUBLIC_SUPPORT_EMAIL` | Support email | email | No | - |
| `INTERNAL_API_KEY` | Internal service auth | string | Yes | - |

### 2.5 Observability Configuration

| Variable | Description | Type | Required | Default |
|----------|-------------|------|----------|---------|
| `LOG_LEVEL` | Logging verbosity | `debug` \| `info` \| `warn` \| `error` | No | `info` |
| `LOG_FORMAT` | Log output format | `json` \| `pretty` | No | `json` |
| `ENABLE_METRICS` | Enable Prometheus metrics | boolean | No | `true` |
| `METRICS_PORT` | Metrics endpoint port | number | No | `9090` |
| `ENABLE_TRACING` | Enable distributed tracing | boolean | No | `false` |
| `TRACING_ENDPOINT` | Tracing collector URL | URL | No | - |

---

## 3. Secrets Management

### 3.1 Secret Categories

| Category | Storage | Rotation | Access |
|----------|---------|----------|--------|
| Database Credentials | Docker secrets | 90 days | Services only |
| API Tokens | Environment vars | On expiry | Services only |
| Session Keys | Docker secrets | Never (use rotation) | API only |
| Webhook Secrets | Environment vars | On compromise | Webhooks only |

### 3.2 Secret Inventory

| Secret | Service | Rotation Policy | Last Rotated |
|--------|---------|-----------------|--------------|
| `DATABASE_URL` | All | 90 days | - |
| `REDIS_PASSWORD` | All | 90 days | - |
| `SESSION_SECRET` | uc-api | On compromise | - |
| `META_API_TOKEN` | uc-api, uc-webhooks | 60 days | - |
| `META_APP_SECRET` | uc-webhooks | On compromise | - |
| `WEBHOOK_VERIFY_TOKEN` | uc-webhooks | On compromise | - |
| `INTERNAL_API_KEY` | All | 90 days | - |

### 3.3 Secret Requirements

```typescript
// Secret validation
interface SecretRequirements {
  name: string;
  min_length: number;
  pattern?: RegExp;
  entropy_bits?: number;
}

const SECRET_REQUIREMENTS: SecretRequirements[] = [
  {
    name: 'SESSION_SECRET',
    min_length: 32,
    entropy_bits: 256
  },
  {
    name: 'OTP_SECRET',
    min_length: 16,
    pattern: /^[A-Za-z0-9+/=]+$/  // Base64
  },
  {
    name: 'INTERNAL_API_KEY',
    min_length: 32,
    pattern: /^[A-Za-z0-9_-]+$/
  },
  {
    name: 'WEBHOOK_VERIFY_TOKEN',
    min_length: 16,
    pattern: /^[A-Za-z0-9_-]+$/
  }
];

// Validation function
function validateSecrets(): void {
  for (const req of SECRET_REQUIREMENTS) {
    const value = process.env[req.name];

    if (!value) {
      throw new Error(`Missing required secret: ${req.name}`);
    }

    if (value.length < req.min_length) {
      throw new Error(`${req.name} must be at least ${req.min_length} characters`);
    }

    if (req.pattern && !req.pattern.test(value)) {
      throw new Error(`${req.name} does not match required pattern`);
    }
  }
}
```

### 3.4 Secret Generation

```bash
# Generate secure secrets
# SESSION_SECRET (32 bytes, base64)
openssl rand -base64 32

# OTP_SECRET (16 bytes, base64)
openssl rand -base64 16

# INTERNAL_API_KEY (32 bytes, URL-safe)
openssl rand -base64 32 | tr '+/' '-_' | tr -d '='

# WEBHOOK_VERIFY_TOKEN (16 bytes, alphanumeric)
openssl rand -hex 16
```

---

## 4. Feature Flags

### 4.1 Feature Flag Registry

| Flag | Description | Default | Environments |
|------|-------------|---------|--------------|
| `FEATURE_UC_WORLD` | Enable UC World virtual space | `true` | All |
| `FEATURE_GAMIFICATION` | Enable gamification system | `true` | All |
| `FEATURE_STREAKS` | Enable streak tracking | `true` | All |
| `FEATURE_DAILY_MISSIONS` | Enable daily missions | `true` | All |
| `FEATURE_WHATSAPP_OTP` | Use WhatsApp for OTP | `true` | prod, staging |
| `FEATURE_ANALYTICS` | Enable event analytics | `true` | All |
| `FEATURE_ADMIN_DASHBOARD` | Enable admin dashboard | `true` | All |
| `FEATURE_INVITE_SYSTEM` | Enable community invites | `true` | All |
| `FEATURE_MAINTENANCE_MODE` | Enable maintenance mode | `false` | All |
| `FEATURE_LIVING_GRAPH` | Enable Living Graph portal discovery | `false` | prod (pilot), staging |
| `FEATURE_GRAPH_ADVANCED` | Enable Louvain/Node2Vec algorithms | `false` | None (gated) |

### 4.2 Feature Flag Configuration

```typescript
// Feature flag types
interface FeatureFlag {
  name: string;
  description: string;
  default_value: boolean;
  environments: ('production' | 'staging' | 'development')[];
  rollout_percentage?: number;  // 0-100 for gradual rollout
  user_targeting?: {
    include_identities?: number[];
    exclude_identities?: number[];
    include_communities?: number[];
  };
}

// Feature flag definitions
const FEATURE_FLAGS: FeatureFlag[] = [
  {
    name: 'FEATURE_UC_WORLD',
    description: 'Enable UC World virtual space feature',
    default_value: true,
    environments: ['production', 'staging', 'development']
  },
  {
    name: 'FEATURE_GAMIFICATION',
    description: 'Enable XP, levels, and leaderboards',
    default_value: true,
    environments: ['production', 'staging', 'development']
  },
  {
    name: 'FEATURE_STREAKS',
    description: 'Enable streak tracking and bonuses',
    default_value: true,
    environments: ['production', 'staging', 'development']
  },
  {
    name: 'FEATURE_DAILY_MISSIONS',
    description: 'Enable daily mission assignments',
    default_value: true,
    environments: ['production', 'staging', 'development']
  },
  {
    name: 'FEATURE_WHATSAPP_OTP',
    description: 'Send OTP via WhatsApp instead of SMS',
    default_value: true,
    environments: ['production', 'staging']
  },
  {
    name: 'FEATURE_MAINTENANCE_MODE',
    description: 'Show maintenance page to all users',
    default_value: false,
    environments: ['production', 'staging', 'development']
  }
];

// Feature flag evaluation
function isFeatureEnabled(
  flagName: string,
  context?: { identity_id?: number; community_id?: number }
): boolean {
  const envValue = process.env[flagName];

  // Environment variable override
  if (envValue !== undefined) {
    return envValue === 'true' || envValue === '1';
  }

  // Find flag definition
  const flag = FEATURE_FLAGS.find(f => f.name === flagName);
  if (!flag) return false;

  // Check environment
  const currentEnv = process.env.NODE_ENV as 'production' | 'staging' | 'development';
  if (!flag.environments.includes(currentEnv)) {
    return false;
  }

  // Check user targeting
  if (context && flag.user_targeting) {
    const { include_identities, exclude_identities, include_communities } = flag.user_targeting;

    if (exclude_identities?.includes(context.identity_id!)) {
      return false;
    }

    if (include_identities?.length && !include_identities.includes(context.identity_id!)) {
      return false;
    }

    if (include_communities?.length && !include_communities.includes(context.community_id!)) {
      return false;
    }
  }

  // Check rollout percentage
  if (flag.rollout_percentage !== undefined && context?.identity_id) {
    const bucket = context.identity_id % 100;
    if (bucket >= flag.rollout_percentage) {
      return false;
    }
  }

  return flag.default_value;
}
```

### 4.3 Feature Flag Usage

```typescript
// Usage in application code
async function handleWorldConnection(identityId: number, communityId: number) {
  // Check feature flag
  if (!isFeatureEnabled('FEATURE_UC_WORLD', { identity_id: identityId, community_id: communityId })) {
    throw new FeatureDisabledError('UC World is not available');
  }

  // Proceed with connection
  await connectToWorld(identityId, communityId);
}

// Middleware for feature-gated routes
function requireFeature(flagName: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const context = {
      identity_id: req.session?.identity_id,
      community_id: req.params.communityId ? parseInt(req.params.communityId) : undefined
    };

    if (!isFeatureEnabled(flagName, context)) {
      return res.status(503).json({
        error: 'Feature not available',
        code: 'FEATURE_DISABLED'
      });
    }

    next();
  };
}

// Route with feature flag
app.get('/api/world/:communityId',
  requireFeature('FEATURE_UC_WORLD'),
  worldController.getStatus
);
```

---

## 5. Per-Environment Values

### 5.1 Development Environment

```bash
# .env.development
NODE_ENV=development
PORT=3001
HOST=0.0.0.0

# Database
DATABASE_URL=mysql://dev:dev@localhost:3306/unofficial_dev

# Redis
REDIS_URL=redis://localhost:6379
REDIS_HOST=localhost
REDIS_PORT=6379

# Security
SESSION_SECRET=dev-session-secret-do-not-use-in-prod
OTP_SECRET=dev-otp-secret-12345
CORS_ORIGINS=http://localhost:3000,http://localhost:3001

# Integrations (mock/sandbox)
META_API_VERSION=v21.0
META_API_TOKEN=dev-token
WHATSAPP_PHONE_ID=dev-phone
WEBHOOK_VERIFY_TOKEN=dev-verify-token

# URLs
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=ws://localhost:2567

# Features
FEATURE_UC_WORLD=true
FEATURE_GAMIFICATION=true
FEATURE_WHATSAPP_OTP=false  # Use console logging in dev

# Observability
LOG_LEVEL=debug
LOG_FORMAT=pretty
ENABLE_METRICS=false
```

### 5.2 Staging Environment

```bash
# .env.staging
NODE_ENV=staging
PORT=3001
HOST=0.0.0.0

# Database
DATABASE_URL=mysql://staging:${DB_PASSWORD}@mysql:3306/unofficial_staging

# Redis
REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
REDIS_HOST=redis
REDIS_PORT=6379

# Security
SESSION_SECRET=${SESSION_SECRET}
OTP_SECRET=${OTP_SECRET}
CORS_ORIGINS=https://staging.unofficialbrasil.com.br

# Integrations (sandbox)
META_API_VERSION=v21.0
META_API_TOKEN=${META_API_TOKEN_STAGING}
WHATSAPP_PHONE_ID=${WHATSAPP_PHONE_ID_STAGING}
WEBHOOK_VERIFY_TOKEN=${WEBHOOK_VERIFY_TOKEN}

# URLs
NEXT_PUBLIC_API_URL=https://staging-api.unofficialbrasil.com.br
NEXT_PUBLIC_WS_URL=wss://staging-world.unofficialbrasil.com.br

# Features
FEATURE_UC_WORLD=true
FEATURE_GAMIFICATION=true
FEATURE_WHATSAPP_OTP=true

# Observability
LOG_LEVEL=info
LOG_FORMAT=json
ENABLE_METRICS=true
```

### 5.3 Production Environment

```bash
# .env.production
NODE_ENV=production
PORT=3001
HOST=0.0.0.0

# Database
DATABASE_URL=mysql://prod:${DB_PASSWORD}@mysql:3306/unofficial

# Redis
REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
REDIS_HOST=redis
REDIS_PORT=6379

# Security
SESSION_SECRET=${SESSION_SECRET}
OTP_SECRET=${OTP_SECRET}
CORS_ORIGINS=https://unofficialbrasil.com.br,https://www.unofficialbrasil.com.br

# Rate limiting (stricter in prod)
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX=60

# Integrations (production)
META_API_VERSION=v21.0
META_API_TOKEN=${META_API_TOKEN}
WHATSAPP_PHONE_ID=${WHATSAPP_PHONE_ID}
WHATSAPP_BUSINESS_ID=${WHATSAPP_BUSINESS_ID}
META_APP_ID=${META_APP_ID}
META_APP_SECRET=${META_APP_SECRET}
WEBHOOK_VERIFY_TOKEN=${WEBHOOK_VERIFY_TOKEN}

# URLs
NEXT_PUBLIC_API_URL=https://api.unofficialbrasil.com.br
NEXT_PUBLIC_WS_URL=wss://world.unofficialbrasil.com.br

# Internal
INTERNAL_API_KEY=${INTERNAL_API_KEY}

# Features
FEATURE_UC_WORLD=true
FEATURE_GAMIFICATION=true
FEATURE_WHATSAPP_OTP=true
FEATURE_MAINTENANCE_MODE=false

# Observability
LOG_LEVEL=info
LOG_FORMAT=json
ENABLE_METRICS=true
METRICS_PORT=9090
```

### 5.4 Environment Comparison Matrix

| Variable | Development | Staging | Production |
|----------|-------------|---------|------------|
| `NODE_ENV` | development | staging | production |
| `LOG_LEVEL` | debug | info | info |
| `LOG_FORMAT` | pretty | json | json |
| `CORS_ORIGINS` | localhost:* | staging.* | unofficialbrasil.* |
| `RATE_LIMIT_MAX` | 1000 | 100 | 60 |
| `SESSION_TTL_DAYS` | 30 | 7 | 7 |
| `FEATURE_WHATSAPP_OTP` | false | true | true |
| `ENABLE_METRICS` | false | true | true |
| `DATABASE_URL` | Local MySQL | Docker MySQL | Docker MySQL |
| `REDIS_URL` | Local Redis | Docker Redis | Docker Redis |

---

## 6. Business Configuration

### 6.1 Gamification Parameters

| Variable | Description | Default | Range |
|----------|-------------|---------|-------|
| `XP_BASE_MISSION_REWARD` | Base XP per mission | `50` | 10-200 |
| `XP_STREAK_BONUS_DAY_7` | Bonus XP at 7-day streak | `100` | 50-500 |
| `XP_STREAK_BONUS_DAY_30` | Bonus XP at 30-day streak | `500` | 200-1000 |
| `XP_STREAK_BONUS_DAY_100` | Bonus XP at 100-day streak | `2000` | 1000-5000 |
| `MISSIONS_PER_DAY` | Daily missions assigned | `3` | 1-5 |
| `STREAK_RESET_HOUR_UTC` | Hour when streaks reset | `0` | 0-23 |
| `MAX_LEVEL` | Maximum user level | `10` | 5-100 |

### 6.2 Level Configuration

```typescript
// Level thresholds (XP required)
const LEVEL_THRESHOLDS: Record<number, number> = {
  1: 0,
  2: 100,
  3: 300,
  4: 600,
  5: 1000,
  6: 1500,
  7: 2100,
  8: 2800,
  9: 3600,
  10: 4500
};

// XP sources and amounts
const XP_SOURCES = {
  mission_complete: parseInt(process.env.XP_BASE_MISSION_REWARD || '50'),
  streak_day_7: parseInt(process.env.XP_STREAK_BONUS_DAY_7 || '100'),
  streak_day_30: parseInt(process.env.XP_STREAK_BONUS_DAY_30 || '500'),
  streak_day_100: parseInt(process.env.XP_STREAK_BONUS_DAY_100 || '2000'),
  first_login_of_day: 10,
  world_visit: 5,
  profile_complete: 25
};
```

### 6.3 UC World Parameters

| Variable | Description | Default |
|----------|-------------|---------|
| `WORLD_MAX_PLAYERS_PER_ROOM` | Max concurrent users per room | `50` |
| `WORLD_TICK_RATE` | Server tick rate (Hz) | `20` |
| `WORLD_POSITION_SYNC_INTERVAL` | Position sync frequency (ms) | `100` |
| `WORLD_IDLE_TIMEOUT_MINUTES` | Auto-disconnect after idle | `15` |
| `WORLD_MAP_WIDTH` | Map width in tiles | `80` |
| `WORLD_MAP_HEIGHT` | Map height in tiles | `92` |

---

## 7. Configuration Validation

### 7.1 Startup Validation

```typescript
// Configuration schema
interface ConfigSchema {
  required: string[];
  optional: string[];
  validators: Record<string, (value: string) => boolean>;
}

const CONFIG_SCHEMA: ConfigSchema = {
  required: [
    'NODE_ENV',
    'DATABASE_URL',
    'REDIS_URL',
    'SESSION_SECRET',
    'CORS_ORIGINS'
  ],
  optional: [
    'PORT',
    'HOST',
    'LOG_LEVEL',
    'ENABLE_METRICS'
  ],
  validators: {
    NODE_ENV: (v) => ['production', 'staging', 'development'].includes(v),
    PORT: (v) => !isNaN(parseInt(v)) && parseInt(v) > 0 && parseInt(v) < 65536,
    LOG_LEVEL: (v) => ['debug', 'info', 'warn', 'error'].includes(v),
    DATABASE_URL: (v) => v.startsWith('mysql://'),
    REDIS_URL: (v) => v.startsWith('redis://'),
    CORS_ORIGINS: (v) => v.split(',').every(o => o.startsWith('http')),
    SESSION_SECRET: (v) => v.length >= 32
  }
};

// Validation function
function validateConfiguration(): void {
  const errors: string[] = [];

  // Check required variables
  for (const name of CONFIG_SCHEMA.required) {
    if (!process.env[name]) {
      errors.push(`Missing required variable: ${name}`);
    }
  }

  // Validate values
  for (const [name, validator] of Object.entries(CONFIG_SCHEMA.validators)) {
    const value = process.env[name];
    if (value && !validator(value)) {
      errors.push(`Invalid value for ${name}: ${value}`);
    }
  }

  // Production-specific checks
  if (process.env.NODE_ENV === 'production') {
    const prodRequired = [
      'META_API_TOKEN',
      'WHATSAPP_PHONE_ID',
      'WEBHOOK_VERIFY_TOKEN',
      'INTERNAL_API_KEY'
    ];

    for (const name of prodRequired) {
      if (!process.env[name]) {
        errors.push(`Missing production-required variable: ${name}`);
      }
    }
  }

  if (errors.length > 0) {
    console.error('Configuration validation failed:');
    errors.forEach(e => console.error(`  - ${e}`));
    process.exit(1);
  }

  console.log('Configuration validated successfully');
}
```

### 7.2 Runtime Configuration Access

```typescript
// Type-safe configuration access
class Config {
  private static instance: Config;

  // Infrastructure
  readonly nodeEnv: 'production' | 'staging' | 'development';
  readonly port: number;
  readonly host: string;
  readonly databaseUrl: string;
  readonly redisUrl: string;

  // Security
  readonly sessionSecret: string;
  readonly corsOrigins: string[];
  readonly rateLimitMax: number;
  readonly rateLimitWindowMs: number;

  // Integrations
  readonly metaApiToken?: string;
  readonly whatsappPhoneId?: string;

  // Features
  readonly features: Record<string, boolean>;

  private constructor() {
    this.nodeEnv = process.env.NODE_ENV as any || 'development';
    this.port = parseInt(process.env.PORT || '3001');
    this.host = process.env.HOST || '0.0.0.0';
    this.databaseUrl = process.env.DATABASE_URL!;
    this.redisUrl = process.env.REDIS_URL!;
    this.sessionSecret = process.env.SESSION_SECRET!;
    this.corsOrigins = process.env.CORS_ORIGINS!.split(',');
    this.rateLimitMax = parseInt(process.env.RATE_LIMIT_MAX || '100');
    this.rateLimitWindowMs = parseInt(process.env.RATE_LIMIT_WINDOW_MS || '60000');
    this.metaApiToken = process.env.META_API_TOKEN;
    this.whatsappPhoneId = process.env.WHATSAPP_PHONE_ID;

    // Load feature flags
    this.features = {};
    for (const flag of FEATURE_FLAGS) {
      this.features[flag.name] = isFeatureEnabled(flag.name);
    }
  }

  static getInstance(): Config {
    if (!Config.instance) {
      Config.instance = new Config();
    }
    return Config.instance;
  }

  isProduction(): boolean {
    return this.nodeEnv === 'production';
  }

  isDevelopment(): boolean {
    return this.nodeEnv === 'development';
  }
}

// Usage
const config = Config.getInstance();
console.log(`Running on port ${config.port}`);
```

---

## 8. Docker Compose Configuration

### 8.1 Environment File Mapping

```yaml
# docker-compose.yml (excerpt)
services:
  uc-api:
    env_file:
      - .env.${NODE_ENV:-production}
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}

  uc-webhooks:
    env_file:
      - .env.${NODE_ENV:-production}
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - META_API_TOKEN=${META_API_TOKEN}
      - WEBHOOK_VERIFY_TOKEN=${WEBHOOK_VERIFY_TOKEN}

  uc-world:
    env_file:
      - .env.${NODE_ENV:-production}
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - REDIS_URL=${REDIS_URL}
```

### 8.2 Secret Injection

```yaml
# docker-compose.yml (with secrets)
secrets:
  db_password:
    file: ./secrets/db_password.txt
  session_secret:
    file: ./secrets/session_secret.txt
  meta_api_token:
    file: ./secrets/meta_api_token.txt

services:
  uc-api:
    secrets:
      - db_password
      - session_secret
    environment:
      - DATABASE_URL=mysql://prod:$(cat /run/secrets/db_password)@mysql:3306/unofficial
      - SESSION_SECRET_FILE=/run/secrets/session_secret
```

---

*This document defines all configuration for the system. New configuration variables must be added to this registry before use.*
