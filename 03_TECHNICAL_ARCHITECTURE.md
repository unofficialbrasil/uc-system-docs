# Technical Architecture

**System:** Unofficial Communities
**Last Updated:** 2026-01-19
**Version:** 1.0.0

---

## 1. System Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            CLOUDFLARE                                        │
│                     (DNS, CDN, DDoS Protection)                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          NGINX REVERSE PROXY                                 │
│                         (SSL Termination, Routing)                           │
│                              72.61.134.35                                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
          ▼                         ▼                         ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │      API        │    │    Webhooks     │
│   Next.js 14    │    │    Fastify      │    │    Fastify      │
│   Port 3000     │    │   Port 3010     │    │   Port 4101     │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         │                      ▼                      │
         │             ┌─────────────────┐             │
         │             │     MySQL       │             │
         └────────────►│      8.0        │◄────────────┘
                       │   Port 3306     │
                       └────────┬────────┘
                                │
                       ┌────────┴────────┐
                       │                 │
                       ▼                 ▼
              ┌─────────────────┐ ┌─────────────────┐
              │     Redis       │ │   BullMQ        │
              │     7.4.6       │ │   (Queues)      │
              │   Port 6379     │ │                 │
              └─────────────────┘ └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                            UC WORLD (3D)                                     │
│    ┌─────────────────┐              ┌─────────────────┐                     │
│    │   Three.js      │◄────────────►│    Colyseus     │                     │
│    │   (Client)      │  WebSocket   │    (Server)     │                     │
│    │   Port 3005     │              │   Port 3006     │                     │
│    └─────────────────┘              └────────┬────────┘                     │
│                                              │                               │
│                                     ┌────────▼────────┐                     │
│                                     │  TimescaleDB    │                     │
│                                     │  (PostgreSQL)   │                     │
│                                     │   Port 5432     │                     │
│                                     └─────────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         MONITORING STACK                                     │
│    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                    │
│    │ Prometheus   │  │   Grafana    │  │  Exporters   │                    │
│    │   :9090      │  │    :3100     │  │   various    │                    │
│    └──────────────┘  └──────────────┘  └──────────────┘                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Inventory

| Service | Technology | Port | Container Name | Purpose |
|---------|------------|------|----------------|---------|
| Frontend | Next.js 14.2.33 | 3000 | unofficial-communities | Web application |
| API | Fastify + Prisma | 3010 | app-uc-api-1 | REST API |
| Webhooks | Fastify + BullMQ | 4101 | app-uc-webhooks-1 | Meta webhook processing |
| UC World Client | Three.js | 3005 | app-uc-world-1 | 3D world frontend |
| UC World Server | Colyseus | 3006 | app-uc-world-1 | Multiplayer server |
| Database (API) | MySQL 8.0 | 3306 | app-uc-api-db-1 | Primary datastore |
| Database (World) | TimescaleDB/PG14 | 5432 | uc-world-db | UC World time-series data |
| Cache | Redis 7.4.6 | 6379 | app-uc-redis-1 | Sessions, cache, queues |
| Metrics | Prometheus | 9090 | uc-prometheus | Metrics collection |
| Dashboards | Grafana | 3100 | uc-grafana | Visualization |

---

## 2. Services and Boundaries

### 2.1 Service Responsibilities

#### Frontend (unofficial-communities)
```
Responsibilities:
├── Server-side rendering (SSR)
├── Static site generation (SSG) for public pages
├── Session management (cookie handling)
├── BFF (Backend-for-Frontend) API routes
├── UI components and interactions
└── Client-side state management

Does NOT:
├── Direct database access
├── Business logic execution
├── Webhook processing
└── Background job execution
```

#### API (uc-api)
```
Responsibilities:
├── REST API endpoints
├── Business logic execution
├── Database operations (via Prisma)
├── Authentication validation
├── Authorization enforcement
└── Data transformation

Does NOT:
├── Serve HTML/UI
├── Handle webhooks (separate service)
├── Real-time WebSocket connections
└── Direct user-facing sessions
```

#### Webhooks (uc-webhooks)
```
Responsibilities:
├── Receive Meta webhooks
├── Signature verification
├── Queue job creation
├── Async job processing
└── Retry handling

Does NOT:
├── Serve user requests
├── Direct database writes (uses API)
├── Session management
└── UI rendering
```

#### UC World (uc-world)
```
Responsibilities:
├── 3D scene rendering (client)
├── WebSocket state sync (server)
├── Multiplayer coordination
├── Zone management
└── Presence tracking

Does NOT:
├── Persistent data storage
├── User authentication (delegates to API)
├── Gamification logic
└── Webhook processing
```

### 2.2 Inter-Service Communication

```
┌──────────────┐     HTTP/REST     ┌──────────────┐
│   Frontend   │──────────────────►│     API      │
└──────────────┘                   └──────────────┘
       │                                  │
       │                                  │
       │ HTTP (internal)                  │ Prisma
       │                                  │
       ▼                                  ▼
┌──────────────┐                   ┌──────────────┐
│   UC World   │                   │    MySQL     │
│   (Client)   │                   └──────────────┘
└──────┬───────┘
       │
       │ WebSocket
       │
       ▼
┌──────────────┐     Redis Pub/Sub  ┌──────────────┐
│   UC World   │◄──────────────────►│   Webhooks   │
│   (Server)   │                    └──────────────┘
└──────────────┘
```

### 2.3 Communication Protocols

| From | To | Protocol | Format | Auth |
|------|-----|----------|--------|------|
| Browser | Frontend | HTTPS | HTML/JSON | Cookie |
| Frontend | API | HTTP | JSON | Internal token |
| Browser | UC World Client | HTTPS | HTML/JS | Cookie |
| UC World Client | UC World Server | WSS | Binary (Colyseus) | Session token |
| Meta | Webhooks | HTTPS | JSON | Signature |
| Webhooks | Redis | TCP | Redis protocol | Password |
| API | MySQL | TCP | MySQL protocol | Credentials |
| All Services | Redis | TCP | Redis protocol | Password |

### 2.4 Admin Dashboard API Endpoints (Step 6)

Living Graph dashboards are served via dedicated admin API endpoints.

#### Dashboard Data Endpoints

| Endpoint | Method | Description | Response |
|----------|--------|-------------|----------|
| `/api/admin/dashboard/community-health` | GET | Community health metrics | `v_community_health` data |
| `/api/admin/dashboard/community-health/:id` | GET | Single community health | Filtered by community_id |
| `/api/admin/dashboard/portal-performance` | GET | Portal performance metrics | `v_portal_performance` data |
| `/api/admin/dashboard/portal-performance/:id` | GET | Single community portals | Filtered by source_community_id |
| `/api/admin/dashboard/neighbor-stability` | GET | Churn and stability metrics | `v_neighbor_stability` data |
| `/api/admin/dashboard/graph-builds` | GET | Graph build history | `v_graph_build_metrics` data |
| `/api/admin/dashboard/proxemic-health` | GET | Zone-based health proxies | `v_proxemic_health` data |
| `/api/admin/dashboard/alerts` | GET | Active dashboard alerts | `v_dashboard_alerts` data |

#### Query Parameters

| Parameter | Type | Applies To | Description |
|-----------|------|------------|-------------|
| `date_from` | ISO date | All | Start date filter |
| `date_to` | ISO date | All | End date filter |
| `community_id` | number | Most | Filter by community |
| `limit` | number | All | Max records (default 100) |
| `offset` | number | All | Pagination offset |
| `alert_type` | string | /alerts | Filter by alert type |
| `severity` | string | /alerts | Filter by severity |

#### Export Endpoints

| Endpoint | Method | Description | Format |
|----------|--------|-------------|--------|
| `/api/admin/dashboard/export` | GET | Export dashboard data | CSV or JSON |

```typescript
// Export query parameters
interface ExportParams {
  report_type: 'community_health' | 'portal_performance' | 'graph_stability' | 'activity';
  community_id?: number;
  date_from: string;  // ISO date
  date_to: string;    // ISO date
  format: 'csv' | 'json';
}
```

#### Response Format

All dashboard endpoints return:

```typescript
interface DashboardResponse<T> {
  data: T[];
  meta: {
    total: number;
    limit: number;
    offset: number;
    generated_at: string;  // ISO timestamp
  };
  alerts?: {
    count: number;
    critical: number;
    high: number;
    medium: number;
  };
}
```

#### Rate Limits

| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Dashboard data | 60/min | Per admin |
| Export | 10/hour | Per admin |
| Alerts | 120/min | Per admin |

---

## 3. Datastores

### 3.1 MySQL 8.0 (Primary Database)

**Purpose:** Persistent relational data storage

**Configuration:**
```yaml
Version: 8.0
Character Set: utf8mb4
Collation: utf8mb4_unicode_ci
Max Connections: 100
InnoDB Buffer Pool: 2GB
```

**Databases:**
| Database | Purpose | Size Estimate |
|----------|---------|---------------|
| uc_prod | Production data | 1-10 GB |

**Key Tables:**
| Table | Purpose | Row Estimate |
|-------|---------|--------------|
| identities | User accounts | 10K-100K |
| profiles | User profiles | 10K-100K |
| communities | Community records | 100-1K |
| members | Membership links | 50K-500K |
| gamification_points | XP and levels | 50K-500K |
| gamification_streaks | Streak tracking | 50K-500K |
| daily_missions | Mission instances | 100K-1M |
| webhook_events | Event log | 1M-10M |

### 3.2 Redis 7.4.6 (Cache and Sessions)

**Purpose:** Sessions, caching, job queues

**Configuration:**
```yaml
Version: 7.4.6
Max Memory: 1GB
Max Memory Policy: allkeys-lru
Persistence: RDB (every 60s if 1000+ keys changed)
```

**Key Patterns:**
| Pattern | Purpose | TTL |
|---------|---------|-----|
| `uc:sess:<id>` | User sessions | 7 days |
| `uc:cache:rankings:<communityId>` | Ranking cache | 5 min |
| `uc:cache:profile:<userId>` | Profile cache | 1 hour |
| `uc:ratelimit:<ip>:<endpoint>` | Rate limit counters | Variable |
| `bull:webhooks:*` | BullMQ job data | Variable |

### 3.3 File Storage

**Current:** Local Docker volumes
**Future:** S3-compatible storage for user uploads

| Volume | Purpose | Path in Container |
|--------|---------|-------------------|
| mysql_data | Database files | /var/lib/mysql |
| redis_data | Redis persistence | /data |

---

## 4. Message Brokers

### 4.1 BullMQ (Redis-backed)

**Purpose:** Async job processing

**Queues:**
| Queue | Purpose | Concurrency | Retry |
|-------|---------|-------------|-------|
| webhooks | Meta webhook processing | 5 | 3x exponential |
| notifications | User notifications | 2 | 3x |
| missions | Daily mission generation | 1 | 1x |
| maintenance | System maintenance tasks | 1 | No |

**Job Lifecycle:**
```
Job Created → Waiting → Active → Completed
                │          │
                │          └──► Failed → Delayed → Active (retry)
                │                          │
                │                          └──► Dead Letter Queue
                │
                └──► Delayed (scheduled)
```

### 4.2 Redis Pub/Sub

**Purpose:** Real-time event propagation

**Channels:**
| Channel | Publisher | Subscribers | Purpose |
|---------|-----------|-------------|---------|
| `uc:events:user:<id>` | API | Frontend (SSE) | User notifications |
| `uc:events:community:<id>` | API | UC World | Community events |
| `uc:presence:<communityId>` | UC World | UC World | Presence sync |

---

## 5. Third-Party Integrations

### 5.1 Meta (WhatsApp Cloud API)

**Purpose:** User verification, webhook events

**Endpoints Used:**
| Endpoint | Purpose |
|----------|---------|
| `POST /messages` | Send OTP via WhatsApp |
| `GET /webhooks` | Verify webhook subscription |
| `POST /webhooks` | Receive message events |

**Rate Limits:**
| Type | Limit |
|------|-------|
| Messages per phone/day | 250 (service conversations) |
| API calls per hour | 200 |

### 5.2 Meta (Instagram Graph API)

**Purpose:** Optional social verification

**Endpoints Used:**
| Endpoint | Purpose |
|----------|---------|
| `GET /me` | Verify Instagram account |
| `GET /webhooks` | Receive activity events |

### 5.3 Cloudflare

**Purpose:** DNS, CDN, DDoS protection

**Services Used:**
| Service | Purpose |
|---------|---------|
| DNS | Domain name resolution |
| Proxy | Traffic proxying |
| WAF | Web application firewall |
| Rate Limiting | Additional rate limits |

---

## 6. Trust Boundaries

### 6.1 Trust Boundary Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           UNTRUSTED (Internet)                               │
│                                                                              │
│    ┌──────────┐        ┌──────────┐        ┌──────────┐                    │
│    │ Browser  │        │   Meta   │        │ Attacker │                    │
│    │  Users   │        │  Servers │        │          │                    │
│    └────┬─────┘        └────┬─────┘        └────┬─────┘                    │
│         │                   │                   │                           │
└─────────┼───────────────────┼───────────────────┼───────────────────────────┘
          │                   │                   │
          │ HTTPS             │ HTTPS             │ Blocked
          │                   │                   │
┌─────────┼───────────────────┼───────────────────┼───────────────────────────┐
│         ▼                   ▼                   ▼                           │
│    ┌─────────────────────────────────────────────────┐                     │
│    │              NGINX (Boundary Enforcer)          │                     │
│    │  • TLS termination                              │                     │
│    │  • Rate limiting                                │                     │
│    │  • Request validation                           │                     │
│    └─────────────────────────────────────────────────┘                     │
│                              │                                              │
│                       DMZ (Semi-Trusted)                                    │
│         ┌────────────────────┼────────────────────┐                        │
│         │                    │                    │                        │
│         ▼                    ▼                    ▼                        │
│    ┌──────────┐        ┌──────────┐        ┌──────────┐                   │
│    │ Frontend │        │   API    │        │ Webhooks │                   │
│    │          │        │          │        │          │                   │
│    └────┬─────┘        └────┬─────┘        └────┬─────┘                   │
│         │                   │                   │                          │
└─────────┼───────────────────┼───────────────────┼──────────────────────────┘
          │                   │                   │
          │            TRUSTED (Internal)         │
          │                   │                   │
┌─────────┼───────────────────┼───────────────────┼──────────────────────────┐
│         │                   │                   │                          │
│         ▼                   ▼                   ▼                          │
│    ┌─────────────────────────────────────────────────┐                    │
│    │                    MySQL                        │                    │
│    │  • Only accepts localhost connections           │                    │
│    │  • Credential-based access                      │                    │
│    └─────────────────────────────────────────────────┘                    │
│                                                                            │
│    ┌─────────────────────────────────────────────────┐                    │
│    │                    Redis                        │                    │
│    │  • Only accepts localhost connections           │                    │
│    │  • Password-protected                           │                    │
│    └─────────────────────────────────────────────────┘                    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Trust Levels

| Level | Components | Trust | Validation Required |
|-------|------------|-------|---------------------|
| 0 (None) | Internet, Browsers | None | Full input validation |
| 1 (Verified) | Authenticated users | Limited | Session validation |
| 2 (Internal) | Internal services | Medium | Service token |
| 3 (Privileged) | Database, Redis | High | Credentials |
| 4 (Admin) | Admin users | Full | Multi-factor |

---

## 7. Sync vs Async Interactions

### 7.1 Synchronous Operations

| Operation | Path | Timeout | Fallback |
|-----------|------|---------|----------|
| Page load | Browser → Frontend | 30s | Error page |
| API call | Frontend → API | 10s | Retry with backoff |
| DB query | API → MySQL | 5s | Error response |
| Cache read | API → Redis | 1s | Skip cache |
| Auth validation | API → Redis | 1s | Reject request |

### 7.2 Asynchronous Operations

| Operation | Mechanism | SLA | Dead Letter |
|-----------|-----------|-----|-------------|
| Webhook processing | BullMQ queue | 5 min | Yes |
| XP award | Direct + Event emit | Immediate | N/A |
| Notification send | BullMQ queue | 10 min | Yes |
| Daily mission gen | Cron + BullMQ | 1 hour | Yes |
| Analytics aggregation | Cron job | 1 day | N/A |

### 7.3 Event Flow Example

```
User completes mission (sync):
    │
    ├── 1. Validate request (sync, 10ms)
    │
    ├── 2. Update database (sync, 50ms)
    │       └── INSERT completed_mission
    │       └── UPDATE gamification_points
    │
    ├── 3. Return success (sync)
    │
    └── 4. Emit event (async, fire-and-forget)
            │
            ├── Update cache (async)
            │
            ├── Check achievements (async)
            │
            └── Send notification (async queue)
```

---

## 8. Scalability Assumptions

### 8.1 Current Capacity

| Resource | Current | Limit | Scaling Trigger |
|----------|---------|-------|-----------------|
| CPU (4 cores) | ~20% avg | 70% sustained | Add cores or VPS |
| Memory (16GB) | ~6GB used | 12GB | Add RAM or optimize |
| Disk (200GB) | ~18% used | 80% | Add storage |
| MySQL connections | ~10 | 100 | Connection pooling |
| Redis memory | ~100MB | 1GB | Increase or cluster |

### 8.2 Scaling Strategy

**Phase 1 (Current):** Vertical scaling on single VPS
- Add CPU/RAM as needed
- Optimize queries and caching
- Limit: ~1000 concurrent users

**Phase 2 (Future):** Horizontal scaling
- Separate database server
- Redis cluster or managed service
- Load balancer + multiple app servers
- Limit: ~10,000 concurrent users

**Phase 3 (Future):** Cloud-native
- Managed database (RDS equivalent)
- Managed Redis (ElastiCache equivalent)
- Kubernetes or managed containers
- CDN for static assets
- Limit: ~100,000+ concurrent users

### 8.3 Bottleneck Analysis

| Component | Bottleneck | Mitigation |
|-----------|------------|------------|
| MySQL | Write throughput | Read replicas, query optimization |
| Redis | Memory | Cluster mode, LRU eviction |
| Colyseus | Rooms per server | Multiple game servers |
| API | Request throughput | Horizontal scaling |
| Frontend | SSR CPU | ISR/SSG, edge caching |

---

## 9. Infrastructure Diagram

### 9.1 Docker Compose Topology

```
docker-compose.prod.yml
│
├── unofficial-communities (Next.js)
│   ├── depends_on: uc-api, uc-redis
│   ├── networks: uc-network
│   └── ports: 3000:3000
│
├── uc-api (Fastify)
│   ├── depends_on: uc-api-db, uc-redis
│   ├── networks: uc-network
│   └── ports: 3010:3010
│
├── uc-webhooks (Fastify)
│   ├── depends_on: uc-api, uc-redis
│   ├── networks: uc-network
│   └── ports: 4101:4101
│
├── uc-world (Three.js + Colyseus)
│   ├── depends_on: uc-api, uc-redis
│   ├── networks: uc-network
│   └── ports: 3005:3005, 3006:3006
│
├── uc-api-db (MySQL)
│   ├── volumes: mysql_data
│   ├── networks: uc-network
│   └── ports: 3306:3306 (localhost only)
│
├── uc-redis (Redis)
│   ├── volumes: redis_data
│   ├── networks: uc-network
│   └── ports: 6379:6379 (localhost only)
│
└── monitoring stack
    ├── uc-prometheus
    ├── uc-grafana
    ├── uc-node-exporter
    ├── uc-cadvisor
    ├── uc-redis-exporter
    └── uc-mysql-exporter
```

### 9.2 Network Configuration

| Network | Purpose | Components |
|---------|---------|------------|
| uc-network | Internal communication | All services |
| host | External access | Nginx proxy |

### 9.3 Volume Configuration

| Volume | Path | Backup |
|--------|------|--------|
| mysql_data | /var/lib/docker/volumes/mysql_data | Daily |
| redis_data | /var/lib/docker/volumes/redis_data | Daily |

---

*This document describes the technical architecture as implemented. Changes require updates to this document and logging in DECISION_LOG.md.*

<!-- Last Updated: 2026-01-19 - Step 6: Added Admin Dashboard API Endpoints (Section 2.4) with data, export, and rate limits -->
<!-- Last Reviewed: 2026-01-27 - No updates needed. Architecture matches running services. -->
