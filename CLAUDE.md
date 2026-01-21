# CLAUDE.md - UC System Docs

> **This is the system-wide documentation repository for Unofficial Communities.**
> All platform-level specifications, governance, and reference documents live here.

## IMPORTANT: Documentation Rules

**DO NOT create additional documentation files in this repository without updating the index.**

All documents must be:
1. **Numbered** (01-99 prefix) for ordered display
2. **Registered** in `01_SYSTEM_CANONICAL_INDEX.md`
3. **Reviewed** at end of each development session

---

## Repository Overview

**uc-system-docs** is the authoritative system-of-record for the Unofficial Communities platform.

| Property | Value |
|----------|-------|
| Location | `/srv/unofficial/prod/app/uc-system-docs/` |
| Documents | 21 numbered files |
| Purpose | Platform-wide specifications and governance |

---

## Document Registry (21 Files)

| # | File | Purpose | Update When |
|---|------|---------|-------------|
| 01 | `01_SYSTEM_CANONICAL_INDEX.md` | Entry point, registry | Document structure changes |
| 02 | `02_PRODUCT_BLUEPRINT.md` | Vision, principles | Vision/principles change |
| 03 | `03_TECHNICAL_ARCHITECTURE.md` | Architecture | Architecture changes |
| 04 | `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md` | Core behavior | Algorithm changes |
| 05 | `05_USER_STATE_MACHINE.md` | User flows | User flows change |
| 06 | `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` | Data handling | Data handling changes |
| 07 | `07_EVENT_AND_ANALYTICS_SPEC.md` | Events | Event definitions change |
| 08 | `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md` | Async processing | Job configurations change |
| 09 | `09_SECURITY_AND_AUTHORIZATION_SPEC.md` | Security | Security policies change |
| 10 | `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md` | Deployment | Deployment procedures change |
| 11 | `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md` | Configuration | Config changes |
| 12 | `12_COMPLIANCE_AND_DATA_PROTECTION.md` | Compliance | Compliance requirements change |
| 13 | `13_RISK_REGISTER.md` | Risks | New risks identified |
| 14 | `14_ASSUMPTION_REGISTER.md` | Assumptions | Assumptions change |
| 15 | `15_DECISION_LOG.md` | Decisions | Decisions made |
| 16 | `16_SESSION_LOG.md` | Session history | **EVERY session** |
| 17 | `17_CLAUDE_CODE_PROMPTS.md` | Claude prompts | Procedures change |
| 18 | `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md` | Research | Research updates |
| 19 | `19_EXECUTION_PLAN.md` | Execution plan | Execution steps change |
| 20 | `20_ADULT_BY_DESIGN_SPEC.md` | Adult-by-Design policy | Age verification changes |
| 21 | `21_UC_WORLD_ADMIN_CONTROLS.md` | Admin camera & debug tools | Admin features change |

---

## Session Management

### At Session Start
1. Read `01_SYSTEM_CANONICAL_INDEX.md` for overview
2. Read `16_SESSION_LOG.md` for recent history
3. Read `17_CLAUDE_CODE_PROMPTS.md` for procedures

### At Session End
1. Update relevant documents with changes
2. Add "Last Reviewed" stamp to unchanged docs: `<!-- Last Reviewed: YYYY-MM-DD - No updates needed -->`
3. **ALWAYS** add entry to `16_SESSION_LOG.md`
4. Commit and push changes

---

## Related Repositories

| Repository | Port | Purpose |
|------------|------|---------|
| unofficial-communities | 3000 | Next.js frontend |
| uc-api | 3010 | Fastify backend API |
| uc-webhooks | 4101 | Meta webhook processor |
| uc-world | 3005 | 3D virtual world |

Each repository has its own `CLAUDE.md` and `README.md` for repo-specific documentation.

---

## Key Paths

| Resource | Path |
|----------|------|
| This repo | `/srv/unofficial/prod/app/uc-system-docs/` |
| Application repos | `/srv/unofficial/prod/app/` |
| Docker Compose | `/srv/unofficial/prod/app/docker-compose.prod.yml` |
| Secrets | `/etc/uc/secrets/` |
| Backups | `/root/uc-backups/` |

---

## Commands

```bash
# View all documents
ls -la /srv/unofficial/prod/app/uc-system-docs/

# Check document word counts
wc -l /srv/unofficial/prod/app/uc-system-docs/*.md

# Git status
cd /srv/unofficial/prod/app/uc-system-docs && git status

# Commit changes
git add -A && git commit -m "docs: [description]" && git push origin main
```

---

## Current Status

- **21 documents** covering all platform aspects
- **Numbered** (01-21) for ordered display
- **Session prompts** v6.1 with comprehensive checks
- **Backup monitoring** integrated into session prompts

---

<!-- Last Updated: 2026-01-20 - Session close: background strip fix completed -->

