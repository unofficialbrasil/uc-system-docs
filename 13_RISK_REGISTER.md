# Risk Register

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## Purpose

This document tracks all identified risks to the Unofficial Communities platform, including their probability, impact, mitigation strategies, and current status.

---

## Risk Classification

### Probability Scale
| Level | Definition | Likelihood |
|-------|------------|------------|
| **Critical** | Almost certain | >90% |
| **High** | Likely | 60-90% |
| **Medium** | Possible | 30-60% |
| **Low** | Unlikely | 10-30% |
| **Minimal** | Rare | <10% |

### Impact Scale
| Level | Definition | Business Effect |
|-------|------------|-----------------|
| **Critical** | Catastrophic | Platform shutdown, data breach, legal action |
| **High** | Major | Significant revenue loss, major feature failure |
| **Medium** | Moderate | Degraded service, partial feature failure |
| **Low** | Minor | Inconvenience, workaround available |
| **Minimal** | Negligible | Cosmetic, no business impact |

### Risk Score
`Risk Score = Probability × Impact`

| Score Range | Risk Level | Action Required |
|-------------|------------|-----------------|
| 16-25 | Critical | Immediate mitigation required |
| 9-15 | High | Mitigation plan required within 1 week |
| 4-8 | Medium | Mitigation plan required within 1 month |
| 1-3 | Low | Monitor, address opportunistically |

---

## Active Risks

### RSK-001: Meta API Breaking Changes
**Category:** Technical
**Probability:** Low (2)
**Impact:** High (4)
**Risk Score:** 8 (Medium)
**Owner:** Engineering Lead
**Status:** Open

**Description:**
Meta (WhatsApp Cloud API, Instagram Graph API) releases breaking changes that disable core verification flows.

**Trigger Conditions:**
- Meta announces deprecation of endpoints in use
- API version forced upgrade with breaking changes
- Rate limit policy changes

**Mitigation Strategy:**
1. Pin API versions explicitly in code
2. Subscribe to Meta developer changelog
3. Maintain test suite against live API
4. Document manual verification fallback procedure
5. Budget engineering time for API migrations

**Contingency Plan:**
- Temporary manual verification via support
- Communicate proactively with affected users
- Prioritize emergency API migration

---

### RSK-002: Single Point of Failure (VPS)
**Category:** Infrastructure
**Probability:** Medium (3)
**Impact:** Critical (5)
**Risk Score:** 15 (High)
**Owner:** Engineering Lead
**Status:** Open - Mitigation In Progress

**Description:**
All services run on a single VPS. Hardware failure, datacenter issue, or provider problem causes complete outage.

**Trigger Conditions:**
- Hostinger datacenter outage
- Hardware failure on VPS
- Network connectivity loss
- Provider account suspension

**Mitigation Strategy:**
1. Daily automated backups (implemented)
2. Backup restoration procedure documented
3. Monitor Hostinger status page
4. Maintain backup VPS provisioning scripts
5. Consider multi-region deployment for future

**Contingency Plan:**
- Restore from backup to new VPS (RTO: 4 hours)
- Communicate outage via social media
- Post-mortem and consider provider change

---

### RSK-003: Redis Data Loss
**Category:** Technical
**Probability:** Low (2)
**Impact:** High (4)
**Risk Score:** 8 (Medium)
**Owner:** Engineering Lead
**Status:** Open

**Description:**
Redis instance loses data (sessions, cache), causing mass user logouts and temporary service degradation.

**Trigger Conditions:**
- Redis container crash without persistence
- Memory exhaustion
- Disk full preventing RDB save
- Accidental FLUSHALL command

**Mitigation Strategy:**
1. Enable RDB persistence (implemented)
2. Include Redis dump in daily backups (implemented)
3. Memory monitoring alerts at 80%
4. Restrict Redis command access

**Contingency Plan:**
- Users re-authenticate (inconvenient but not catastrophic)
- Restore from backup if needed
- Investigate root cause

---

### RSK-004: Security Breach / Data Leak
**Category:** Security
**Probability:** Low (2)
**Impact:** Critical (5)
**Risk Score:** 10 (High)
**Owner:** Security Lead
**Status:** Open - Mitigations Implemented

**Description:**
Unauthorized access to user data, credentials, or system infrastructure.

**Trigger Conditions:**
- SQL injection exploited
- Authentication bypass discovered
- Secrets exposed in logs or code
- Server compromise via unpatched vulnerability

**Mitigation Strategy:**
1. Security audit completed (12 issues fixed)
2. CSRF protection implemented
3. Rate limiting on sensitive endpoints
4. Security headers configured
5. Secrets stored outside repos in /etc/uc/secrets/
6. Regular dependency updates
7. Security logging enabled

**Contingency Plan:**
- Incident response procedure (see DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md)
- User notification within 72 hours (LGPD requirement)
- Forensic analysis
- Credential rotation

---

### RSK-005: Colyseus Scalability Limit
**Category:** Technical
**Probability:** Medium (3)
**Impact:** Medium (3)
**Risk Score:** 9 (High)
**Owner:** Engineering Lead
**Status:** Open

**Description:**
UC World rooms cannot handle target concurrent user count (50 per room) with acceptable latency.

**Trigger Conditions:**
- Room population exceeds 30 with latency >200ms
- State sync frequency insufficient
- Network bandwidth exceeded

**Mitigation Strategy:**
1. Load testing with synthetic users
2. Monitor room size and latency metrics
3. Implement floor overflow system (>80 users → Floor 2)
4. Optimize state schema for minimal data

**Contingency Plan:**
- Reduce room capacity limit
- Implement aggressive floor splitting
- Consider alternative multiplayer architecture

---

### RSK-006: Key Person Dependency
**Category:** Operational
**Probability:** Medium (3)
**Impact:** High (4)
**Risk Score:** 12 (High)
**Owner:** Product Owner
**Status:** Open

**Description:**
Critical system knowledge concentrated in single individuals. Their unavailability blocks development or operations.

**Trigger Conditions:**
- Key engineer unavailable (illness, departure)
- Undocumented system knowledge needed
- Emergency requiring specialized knowledge

**Mitigation Strategy:**
1. Comprehensive system documentation (this folder)
2. Repo-specific CLAUDE.md files
3. Pair programming for critical systems
4. SESSION_LOG.md for context transfer

**Contingency Plan:**
- Emergency documentation sprint
- External consultant engagement
- Prioritize knowledge transfer

---

### RSK-007: LGPD Non-Compliance Fine
**Category:** Legal
**Probability:** Low (2)
**Impact:** Critical (5)
**Risk Score:** 10 (High)
**Owner:** Product Owner
**Status:** Open - Mitigations In Progress

**Description:**
LGPD violation resulting in fines (up to 2% of revenue) or operational restrictions.

**Trigger Conditions:**
- User data request not fulfilled within 15 days
- Data breach without proper notification
- Processing without valid legal basis
- Missing privacy policy

**Mitigation Strategy:**
1. Privacy policy published
2. Data deletion endpoint planned
3. Consent tracking in user model
4. Data retention policies defined
5. See COMPLIANCE_AND_DATA_PROTECTION.md

**Contingency Plan:**
- Legal counsel engagement
- Emergency compliance sprint
- User communication

---

### RSK-008: Competitor with Aggressive Tactics
**Category:** Market
**Probability:** Medium (3)
**Impact:** Medium (3)
**Risk Score:** 9 (High)
**Owner:** Product Owner
**Status:** Open

**Description:**
Competitor using dark patterns and engagement hacking captures market share despite ethical concerns.

**Trigger Conditions:**
- Competitor raises significant funding
- Competitor shows rapid user growth
- Users defect citing "better engagement"

**Mitigation Strategy:**
1. Focus on differentiation through ethics (long game)
2. Build community leader relationships
3. Document competitive intelligence
4. Consider targeted feature parity where ethical

**Contingency Plan:**
- User research on competitive switching
- Adjust positioning
- Consider B2B pivot if B2C fails

---

### RSK-009: WhatsApp Policy Change
**Category:** Platform
**Probability:** Low (2)
**Impact:** Critical (5)
**Risk Score:** 10 (High)
**Owner:** Product Owner
**Status:** Open

**Description:**
WhatsApp changes policies to restrict third-party integrations or community management tools.

**Trigger Conditions:**
- WhatsApp announces policy change
- API access revoked
- Terms of service violation alleged

**Mitigation Strategy:**
1. Maintain compliance with WhatsApp policies
2. Diversify to other platforms (Telegram, Discord) as backup
3. Build direct relationships (not solely API-dependent)
4. Monitor policy announcements

**Contingency Plan:**
- Platform diversification sprint
- User migration support
- Legal review of options

---

### RSK-010: Database Corruption
**Category:** Technical
**Probability:** Minimal (1)
**Impact:** Critical (5)
**Risk Score:** 5 (Medium)
**Owner:** Engineering Lead
**Status:** Open

**Description:**
MySQL database corruption causing data loss or inconsistency.

**Trigger Conditions:**
- Disk failure during write
- Improper shutdown
- Bug in MySQL
- Ransomware/malware

**Mitigation Strategy:**
1. Daily automated backups (implemented)
2. MySQL running in Docker with volume persistence
3. Monitor disk health
4. Regular backup restoration tests

**Contingency Plan:**
- Restore from backup (RPO: 24 hours)
- Communicate data loss scope to users
- Post-mortem

---

## Risk Matrix

```
Impact →
     │ Minimal(1)  Low(2)   Medium(3)  High(4)   Critical(5)
─────┼──────────────────────────────────────────────────────
  C  │                                           RSK-002
  r  │                                RSK-005    RSK-006
  i  │                                RSK-008
  t  │
  i  │
  c  │
  a  │
  l  │
 (5) │
─────┤
  H  │
  i  │
  g  │                                RSK-001    RSK-004
  h  │                                RSK-003    RSK-007
 (4) │                                           RSK-009
─────┤
  M  │
  e  │
  d  │
 (3) │
─────┤
  L  │
  o  │
  w  │
 (2) │
─────┤
  M  │                                           RSK-010
  i  │
  n  │
 (1) │
─────┴──────────────────────────────────────────────────────
Probability: Minimal(1) Low(2) Medium(3) High(4) Critical(5)
```

---

## Risk Index

| ID | Description | Category | Prob | Impact | Score | Status |
|----|-------------|----------|------|--------|-------|--------|
| RSK-001 | Meta API Breaking Changes | Technical | Low | High | 8 | Open |
| RSK-002 | Single Point of Failure (VPS) | Infrastructure | Medium | Critical | 15 | Mitigating |
| RSK-003 | Redis Data Loss | Technical | Low | High | 8 | Open |
| RSK-004 | Security Breach / Data Leak | Security | Low | Critical | 10 | Mitigated |
| RSK-005 | Colyseus Scalability Limit | Technical | Medium | Medium | 9 | Open |
| RSK-006 | Key Person Dependency | Operational | Medium | High | 12 | Open |
| RSK-007 | LGPD Non-Compliance Fine | Legal | Low | Critical | 10 | Mitigating |
| RSK-008 | Competitor with Aggressive Tactics | Market | Medium | Medium | 9 | Open |
| RSK-009 | WhatsApp Policy Change | Platform | Low | Critical | 10 | Open |
| RSK-010 | Database Corruption | Technical | Minimal | Critical | 5 | Open |

---

*Next Risk ID: RSK-011*
