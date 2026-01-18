# System Canonical Index

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 2.2.0
**Maintainer:** Engineering Team

---

## 1. Purpose of This System Folder

This folder (`/srv/unofficial/prod/app/uc-system-docs/`) is the **authoritative system-of-record** for the Unofficial Communities SaaS platform.

**It provides:**
1. **Single Source of Truth** - All system-level decisions, specifications, and governance documents
2. **Audit Trail** - Complete history of architectural decisions, risks, and assumptions
3. **Operational Clarity** - Runbooks, deployment procedures, and incident response
4. **Compliance Readiness** - LGPD/GDPR documentation and data protection specifications
5. **Onboarding Efficiency** - New team members can understand the entire system from this folder

**This folder is NOT:**
- A code repository
- A place for application-specific documentation (those live in repo CLAUDE.md files)
- A marketing or product requirements document

---

## 2. Document Registry (20 Files)

| # | Document | Purpose | Status |
|---|----------|---------|--------|
| 01 | `01_SYSTEM_CANONICAL_INDEX.md` | Entry point, document registry (this file) | Active |
| 02 | `02_PRODUCT_BLUEPRINT.md` | Vision, principles, behavioral science | Active |
| 03 | `03_TECHNICAL_ARCHITECTURE.md` | System architecture and integrations | Active |
| 04 | `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md` | Core behavior specification | Active |
| 05 | `05_USER_STATE_MACHINE.md` | User flows and state transitions | Active |
| 06 | `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` | Data handling and lifecycle | Active |
| 07 | `07_EVENT_AND_ANALYTICS_SPEC.md` | Events and analytics | Active |
| 08 | `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md` | Async processing and queues | Active |
| 09 | `09_SECURITY_AND_AUTHORIZATION_SPEC.md` | Security specification | Active |
| 10 | `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md` | Deployment procedures | Active |
| 11 | `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md` | Configuration registry | Active |
| 12 | `12_COMPLIANCE_AND_DATA_PROTECTION.md` | LGPD/GDPR compliance | Active |
| 13 | `13_RISK_REGISTER.md` | Risk management | Active |
| 14 | `14_ASSUMPTION_REGISTER.md` | Documented assumptions | Active |
| 15 | `15_DECISION_LOG.md` | Decision history | Active |
| 16 | `16_SESSION_LOG.md` | Development session history | Active |
| 17 | `17_CLAUDE_CODE_PROMPTS.md` | Session prompts for Claude Code | Active |
| 18 | `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md` | Research reference (studies) | Active |
| 19 | `19_EXECUTION_PLAN.md` | System execution plan and implementation guide | Active |
| 20 | `20_ADULT_BY_DESIGN_SPEC.md` | 18+ age verification and enforcement policy | Active |

---

## 3. Repository Documentation (Per-Repo Files)

Each repository maintains 2 files for repo-specific context:

| Repository | CLAUDE.md | README.md |
|------------|-----------|-----------|
| unofficial-communities | Technical reference | GitHub overview |
| uc-api | Technical reference | GitHub overview |
| uc-webhooks | Technical reference | GitHub overview |
| uc-world | Technical reference | GitHub overview |

**Location:** `/srv/unofficial/prod/app/[repo]/`

**Rule:**
- **CLAUDE.md** - Technical reference for development
- **README.md** - Basic overview for GitHub
- **System folder** - Platform-wide specifications

---

## 4. Document Reading Order

For new team members or comprehensive understanding:

```
START HERE
    │
    ▼
01_SYSTEM_CANONICAL_INDEX.md (this file)
    │
    ├──► 02_PRODUCT_BLUEPRINT.md (vision, principles)
    │
    ├──► 03_TECHNICAL_ARCHITECTURE.md (how it's built)
    │
    ├──► 04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md (core behavior)
    │
    ├──► 05_USER_STATE_MACHINE.md (user flows)
    │
    └──► 09_SECURITY_AND_AUTHORIZATION_SPEC.md (auth, RBAC)

For operations:
    10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md
    11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md

For compliance:
    12_COMPLIANCE_AND_DATA_PROTECTION.md

For audit trail:
    13_RISK_REGISTER.md
    14_ASSUMPTION_REGISTER.md
    15_DECISION_LOG.md
    16_SESSION_LOG.md
```

---

## 5. Key Paths

| Resource | Path |
|----------|------|
| System Documentation | `/srv/unofficial/prod/app/uc-system-docs/` |
| Application Code | `/srv/unofficial/prod/app/` |
| Docker Compose | `/srv/unofficial/prod/app/docker-compose.prod.yml` |
| Secrets | `/etc/uc/secrets/` |
| Backups | `/root/uc-backups/` |

---

## 6. Services

| Service | Port | Technology | Repository |
|---------|------|------------|------------|
| unofficial-communities | 3000 | Next.js 14 | Frontend |
| uc-api | 3010 | Fastify + Prisma | Backend API |
| uc-webhooks | 4101 | Fastify + BullMQ | Webhook processor |
| uc-world | 3005 | Three.js + Colyseus | 3D virtual world |

---

## 7. Update Protocol

### At End of Every Session
1. Update relevant documents with changes made
2. Add "Last Reviewed" stamp to documents not needing changes
3. Add entry to `16_SESSION_LOG.md`
4. Commit and push all changes

### Adding a New Document
1. Log decision in `15_DECISION_LOG.md`
2. Add entry to Document Registry (Section 2)
3. Number file appropriately (##_FILENAME.md)
4. Update `16_SESSION_LOG.md`

---

*This document is the entry point for understanding the Unofficial Communities system.*
*All other documents in this folder are discoverable from here.*

<!-- Last Updated: 2026-01-18 - Added 20_ADULT_BY_DESIGN_SPEC.md to registry -->
