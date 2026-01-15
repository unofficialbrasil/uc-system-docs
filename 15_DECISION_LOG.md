# Decision Log

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## Purpose

This document records all significant architectural, technical, and operational decisions made for the Unofficial Communities platform. Each decision includes context, alternatives considered, and revisit criteria.

---

## Decision Format

```
### DEC-XXXX: [Title]
**Date:** YYYY-MM-DD
**Status:** Accepted | Superseded | Deprecated
**Superseded By:** DEC-XXXX (if applicable)

#### Context
[Problem statement and background]

#### Options Considered
1. [Option A] - [Brief description]
2. [Option B] - [Brief description]
3. [Option C] - [Brief description]

#### Decision
[What was decided and why]

#### Trade-offs
- [Trade-off 1]
- [Trade-off 2]

#### Rejected Alternatives
- [Why Option X was rejected]

#### Revisit Criteria
[When should this decision be reconsidered]
```

---

## Decisions

### DEC-0001: System Documentation Location
**Date:** 2026-01-14
**Status:** Accepted

#### Context
The project had documentation scattered across multiple repository CLAUDE.md files, making it difficult to understand the complete system and maintain consistency.

#### Options Considered
1. **Single mega-CLAUDE.md** - One file in the main repository containing everything
2. **Wiki-based documentation** - External wiki (Notion, Confluence)
3. **Dedicated system folder** - `/srv/unofficial/prod/system/app/` outside all repos
4. **Documentation repository** - Separate git repo for docs only

#### Decision
Option 3: Dedicated system folder at `/srv/unofficial/prod/system/app/`. This provides:
- Clear separation between system-level and repo-level documentation
- No dependency on external services
- Audit trail through SESSION_LOG.md
- Single authoritative location

#### Trade-offs
- Requires manual sync if team members work offline
- Not version-controlled by default (could add git if needed)
- Requires server access to read/update

#### Rejected Alternatives
- Wiki: External dependency, potential sync issues, harder to audit
- Mega-CLAUDE.md: Too large, mixes system and repo concerns
- Separate repo: Overhead of another repo, sync complexity

#### Revisit Criteria
- If team grows beyond 5 engineers
- If remote-first workflow becomes primary
- If compliance requires git-based audit trail

---

### DEC-0002: Three.js Migration from Phaser 3
**Date:** 2026-01-12
**Status:** Accepted

#### Context
UC World initially used Phaser 3 for 2D rendering. Requirements evolved to need 3D isometric views, better performance, and future VR/AR compatibility.

#### Options Considered
1. **Phaser 3 + Isometric Plugin** - Keep Phaser, add isometric rendering
2. **Three.js** - Full 3D engine, isometric camera
3. **Babylon.js** - Alternative 3D engine
4. **Unity WebGL** - Game engine export to web

#### Decision
Option 2: Three.js with isometric camera setup. Reasons:
- Lighter weight than Babylon.js or Unity
- Excellent documentation and community
- Direct WebGL control
- Compatible with Colyseus multiplayer
- Future path to true 3D if needed

#### Trade-offs
- Steeper learning curve than Phaser 3
- More manual work for UI elements
- Need to build more from scratch

#### Rejected Alternatives
- Phaser 3 + Isometric: Limited to fake-3D, plugin quality concerns
- Babylon.js: Heavier, more complex than needed
- Unity WebGL: Large bundle size, licensing complexity

#### Revisit Criteria
- If mobile performance becomes critical
- If true VR/AR support is required
- If Babylon.js significantly improves bundle size

---

### DEC-0003: Hexagonal Community Layout
**Date:** 2026-01-13
**Status:** Accepted

#### Context
Initial UC World used rectangular arenas. Research into spatial design for social platforms indicated hexagonal layouts provide better navigation and natural neighbor connections.

#### Options Considered
1. **Rectangular Grid** - Simple, familiar, easy to implement
2. **Hexagonal Honeycomb** - Each community has 6 neighbors
3. **Circular Radial** - Communities arranged in rings
4. **Free-form Graph** - No geometric constraint

#### Decision
Option 2: Hexagonal honeycomb with pointy-top orientation. Specifications:
- 80×92 tiles (2560×2944 pixels)
- 6 portal zones along edges (not corners)
- 7 interior zones (Hub, Workshop, 2 Lounges, Brand, Quiet, Onboarding)

#### Trade-offs
- More complex geometry calculations
- Portal placement less intuitive initially
- Harder to visualize in documentation

#### Rejected Alternatives
- Rectangular: Lacks natural neighbor concept, feels artificial
- Circular: Difficult to tile, inconsistent neighbor counts
- Free-form: Too chaotic, no wayfinding consistency

#### Revisit Criteria
- If user testing shows navigation confusion
- If portal mechanics prove too complex
- If scaling requires different topology

---

### DEC-0004: Redis for Session Storage
**Date:** 2025-12-XX
**Status:** Accepted

#### Context
Need for centralized session storage that works across multiple application instances.

#### Options Considered
1. **JWT-only (Stateless)** - Token contains all session data
2. **MySQL Sessions** - Database-backed sessions
3. **Redis Sessions** - In-memory key-value store
4. **Cookie-only** - Client-side session storage

#### Decision
Option 3: Redis with session ID in httpOnly cookie. Pattern:
- Cookie: `uc_session` (httpOnly, SameSite=lax)
- Redis Key: `uc:sess:<sessionId>`
- TTL: 7 days (604800 seconds)

#### Trade-offs
- Requires Redis to be running
- Session data limited by Redis memory
- Need to handle Redis failures gracefully

#### Rejected Alternatives
- JWT-only: Cannot revoke sessions, token size grows with data
- MySQL: Too slow for session operations, unnecessary persistence
- Cookie-only: Size limits, security concerns

#### Revisit Criteria
- If horizontal scaling requires distributed sessions
- If Redis memory becomes constraint
- If compliance requires session audit logs

---

### DEC-0005: Fastify over Express
**Date:** 2025-XX-XX
**Status:** Accepted

#### Context
Selection of Node.js web framework for the API service.

#### Options Considered
1. **Express** - Most popular, extensive ecosystem
2. **Fastify** - Performance-focused, schema validation built-in
3. **Koa** - Minimalist, async-first
4. **NestJS** - Full framework with TypeScript decorators

#### Decision
Option 2: Fastify. Reasons:
- 2-3x faster than Express in benchmarks
- Built-in JSON schema validation
- First-class TypeScript support
- Plugin architecture matches our modular design

#### Trade-offs
- Smaller ecosystem than Express
- Some middleware needs adaptation
- Team learning curve

#### Rejected Alternatives
- Express: Performance not acceptable for scale targets
- Koa: Too minimal, would need to build too much
- NestJS: Too opinionated, unnecessary abstraction

#### Revisit Criteria
- If Fastify plugin ecosystem becomes limiting
- If team growth requires more familiar framework

---

### DEC-0006: Colyseus for Multiplayer
**Date:** 2025-XX-XX
**Status:** Accepted

#### Context
Need for real-time multiplayer state synchronization in UC World.

#### Options Considered
1. **Socket.io** - General-purpose WebSocket library
2. **Colyseus** - Game-focused multiplayer framework
3. **Photon** - Commercial game networking
4. **Custom WebSocket** - Build from scratch

#### Decision
Option 2: Colyseus. Reasons:
- Designed for game state synchronization
- Automatic delta compression
- Room-based architecture matches our zone concept
- MIT license, no vendor lock-in

#### Trade-offs
- Learning curve for state schema
- Less flexibility than raw WebSockets
- Tied to Node.js server

#### Rejected Alternatives
- Socket.io: Not optimized for game state, manual sync needed
- Photon: Commercial license, external dependency
- Custom: Too much work to build properly

#### Revisit Criteria
- If scaling beyond 50 concurrent users per room becomes difficult
- If cross-platform (mobile native) support needed

---

### DEC-0007: Behavioral Science Framework Integration
**Date:** 2025-XX-XX
**Status:** Accepted

#### Context
Product direction required differentiation through ethical, evidence-based design rather than engagement hacking.

#### Options Considered
1. **Traditional Gamification** - Points, badges, leaderboards
2. **Behavioral Science Framework** - Fogg MAP, SDT, ethical constraints
3. **No Framework** - Ad-hoc feature decisions
4. **External Consultancy** - Hire behavioral scientists

#### Decision
Option 2: Integrate Fogg MAP Model and Self-Determination Theory as mandatory frameworks:
- All features must map to Motivation, Ability, or Prompt
- Bias toward Ability (reduce friction) over Motivation (add pressure)
- SDT needs: Autonomy, Competence, Belonging
- Explicit forbidden patterns list

#### Trade-offs
- Slower feature development (requires framework mapping)
- May reject high-engagement but unethical patterns
- Requires team training on frameworks

#### Rejected Alternatives
- Traditional Gamification: Proven to cause burnout, not differentiated
- No Framework: Inconsistent decisions, no competitive moat
- External Consultancy: Cost, dependency, slower iteration

#### Revisit Criteria
- If user research shows framework is too constraining
- If competitive pressure requires faster feature velocity
- If academic research updates underlying frameworks

---

### DEC-0008: LGPD-First Data Architecture
**Date:** 2025-XX-XX
**Status:** Accepted

#### Context
Brazilian market requires LGPD compliance. Data architecture needed to support privacy rights from the start.

#### Options Considered
1. **Compliance Retrofit** - Build features first, add compliance later
2. **LGPD-First** - Design data architecture with compliance from day one
3. **Third-Party Compliance** - Use compliance SaaS tools

#### Decision
Option 2: LGPD-First architecture:
- All PII has explicit retention rules
- Data deletion cascades defined upfront
- Consent tracking built into user model
- Anonymization paths for analytics

#### Trade-offs
- More complex data model
- Slower initial development
- Some features harder to implement

#### Rejected Alternatives
- Compliance Retrofit: Technical debt, potential legal exposure
- Third-Party: Dependency, cost, may not fit custom needs

#### Revisit Criteria
- If LGPD regulations change significantly
- If entering EU market requires GDPR-specific adjustments

---

### DEC-0009: Monorepo vs Multi-Repo
**Date:** 2025-XX-XX
**Status:** Accepted

#### Context
Organization of source code across multiple services.

#### Options Considered
1. **Single Monorepo** - All code in one repository
2. **Multi-Repo** - Separate repository per service
3. **Hybrid** - Monorepo for related services, separate for others

#### Decision
Option 2: Multi-Repo with 4 repositories:
- `unofficial-communities` - Next.js frontend
- `uc-api` - Fastify API
- `uc-webhooks` - Webhook processor
- `uc-world` - 3D World (Three.js + Colyseus)

#### Trade-offs
- Cross-repo changes require multiple PRs
- Shared code requires npm packages or copy
- Version coordination more complex

#### Rejected Alternatives
- Monorepo: Build complexity, CI/CD setup overhead
- Hybrid: Unclear boundaries, worst of both

#### Revisit Criteria
- If shared code between repos grows significantly
- If team size makes monorepo tooling worthwhile

---

### DEC-0010: Prometheus + Grafana for Monitoring
**Date:** 2026-01-11
**Status:** Accepted

#### Context
Need for observability into production system health and performance.

#### Options Considered
1. **Datadog** - Commercial APM
2. **New Relic** - Commercial APM
3. **Prometheus + Grafana** - Open source stack
4. **CloudWatch** - AWS-native (not applicable, using Hostinger)

#### Decision
Option 3: Prometheus + Grafana stack:
- Prometheus for metrics collection (port 9090)
- Grafana for visualization (port 3100)
- Exporters: node-exporter, cadvisor, redis-exporter, mysql-exporter

#### Trade-offs
- Self-managed infrastructure
- No automatic distributed tracing
- Manual dashboard creation

#### Rejected Alternatives
- Datadog/New Relic: Cost at scale, external dependency
- CloudWatch: Not using AWS

#### Revisit Criteria
- If debugging distributed issues becomes too difficult
- If team time spent on monitoring infra exceeds SaaS cost

---

## Decision Index

| ID | Title | Date | Status |
|----|-------|------|--------|
| DEC-0001 | System Documentation Location | 2026-01-14 | Accepted |
| DEC-0002 | Three.js Migration from Phaser 3 | 2026-01-12 | Accepted |
| DEC-0003 | Hexagonal Community Layout | 2026-01-13 | Accepted |
| DEC-0004 | Redis for Session Storage | 2025-12-XX | Accepted |
| DEC-0005 | Fastify over Express | 2025-XX-XX | Accepted |
| DEC-0006 | Colyseus for Multiplayer | 2025-XX-XX | Accepted |
| DEC-0007 | Behavioral Science Framework Integration | 2025-XX-XX | Accepted |
| DEC-0008 | LGPD-First Data Architecture | 2025-XX-XX | Accepted |
| DEC-0009 | Monorepo vs Multi-Repo | 2025-XX-XX | Accepted |
| DEC-0010 | Prometheus + Grafana for Monitoring | 2026-01-11 | Accepted |

---

*Next Decision ID: DEC-0011*
