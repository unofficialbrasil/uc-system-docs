# Compliance and Data Protection

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## 1. Regulatory Framework

### 1.1 Applicable Regulations

| Regulation | Jurisdiction | Applicability | Status |
|------------|--------------|---------------|--------|
| LGPD | Brazil | Primary - all users | Active |
| GDPR | EU | Secondary - if EU users | Prepared |
| Marco Civil | Brazil | Internet governance | Active |
| CDC | Brazil | Consumer protection | Active |

### 1.2 Compliance Responsibilities

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA PROTECTION ROLES                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────┐                                            │
│  │   Data Controller   │                                            │
│  │   (Unofficial)      │                                            │
│  │                     │                                            │
│  │   Responsibilities: │                                            │
│  │   - Define purposes │                                            │
│  │   - Ensure lawful   │                                            │
│  │     processing      │                                            │
│  │   - Handle DSR      │                                            │
│  │   - Report breaches │                                            │
│  └─────────┬───────────┘                                            │
│            │                                                         │
│            ▼                                                         │
│  ┌─────────────────────┐    ┌─────────────────────┐                │
│  │   Data Processor    │    │   Sub-Processors    │                │
│  │   (Internal)        │    │   (Third Parties)   │                │
│  │                     │    │                     │                │
│  │   - VPS Provider    │    │   - Meta (WhatsApp) │                │
│  │   - Backup Storage  │    │   - (Future) CDN    │                │
│  └─────────────────────┘    └─────────────────────┘                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. LGPD Obligations

### 2.1 Legal Basis for Processing

| Data Category | Purpose | Legal Basis (LGPD Art. 7) | Retention |
|---------------|---------|---------------------------|-----------|
| Phone Number | Authentication | Consent (I) | Account lifetime |
| Display Name | Identification | Contract (V) | Account lifetime |
| Avatar Image | Personalization | Consent (I) | Account lifetime |
| Community Membership | Service delivery | Contract (V) | Membership duration |
| Gamification Data | Feature functionality | Contract (V) | Account lifetime |
| UC World Position | Feature functionality | Contract (V) | Session only |
| Analytics Events | Service improvement | Legitimate Interest (IX) | 90 days |
| Security Logs | Legal compliance | Legal Obligation (II) | 2 years |

### 2.2 Data Processing Principles (Art. 6)

| Principle | Implementation |
|-----------|----------------|
| **Purpose** | All data tied to specific, documented purposes |
| **Adequacy** | Only collect data necessary for stated purposes |
| **Necessity** | Minimize data collection to essential fields |
| **Free Access** | Data export available on request |
| **Quality** | Regular data validation and cleanup |
| **Transparency** | Clear privacy policy, accessible terms |
| **Security** | Encryption, access controls, monitoring |
| **Prevention** | Privacy by design in all features |
| **Non-discrimination** | No discriminatory processing |
| **Accountability** | Audit trails, documented decisions |

### 2.3 Data Subject Rights (Art. 18)

| Right | Implementation | Response Time |
|-------|----------------|---------------|
| **Confirmation** | API endpoint to check data existence | Immediate |
| **Access** | Data export in JSON/CSV format | 15 days |
| **Correction** | Profile edit functionality | Immediate |
| **Anonymization** | Convert to anonymous data | 15 days |
| **Deletion** | Account deletion with data removal | 15 days |
| **Portability** | Structured data export | 15 days |
| **Information** | Privacy policy, data usage disclosure | Immediate |
| **Revocation** | Consent withdrawal mechanism | Immediate |
| **Opposition** | Opt-out of non-essential processing | Immediate |

---

## 3. Consent Management

### 3.1 Consent Model

```typescript
// Consent types
enum ConsentType {
  TERMS_OF_SERVICE = 'terms_of_service',      // Required
  PRIVACY_POLICY = 'privacy_policy',          // Required
  MARKETING = 'marketing',                    // Optional
  ANALYTICS = 'analytics',                    // Optional (but default on)
  THIRD_PARTY_SHARING = 'third_party'         // Optional
}

// Consent record
interface Consent {
  identity_id: number;
  consent_type: ConsentType;
  granted: boolean;
  granted_at: Date | null;
  revoked_at: Date | null;
  version: string;              // Version of terms consented to
  ip_address: string;           // For audit
  user_agent: string;           // For audit
}

// Required consents for service access
const REQUIRED_CONSENTS: ConsentType[] = [
  ConsentType.TERMS_OF_SERVICE,
  ConsentType.PRIVACY_POLICY
];
```

### 3.2 Consent Collection

```typescript
// Consent collection during signup
async function collectConsent(
  identityId: number,
  consents: { type: ConsentType; granted: boolean }[],
  metadata: { ip: string; userAgent: string }
): Promise<void> {
  const timestamp = new Date();
  const currentVersions = await getCurrentConsentVersions();

  for (const consent of consents) {
    await db.consent.upsert({
      where: {
        identity_id_consent_type: {
          identity_id: identityId,
          consent_type: consent.type
        }
      },
      create: {
        identity_id: identityId,
        consent_type: consent.type,
        granted: consent.granted,
        granted_at: consent.granted ? timestamp : null,
        revoked_at: consent.granted ? null : timestamp,
        version: currentVersions[consent.type],
        ip_address: metadata.ip,
        user_agent: metadata.userAgent
      },
      update: {
        granted: consent.granted,
        granted_at: consent.granted ? timestamp : undefined,
        revoked_at: consent.granted ? null : timestamp,
        version: currentVersions[consent.type],
        ip_address: metadata.ip,
        user_agent: metadata.userAgent
      }
    });
  }

  // Log consent change
  await logConsentChange(identityId, consents, metadata);
}
```

### 3.3 Consent Verification

```typescript
// Check consent before processing
async function verifyConsent(
  identityId: number,
  requiredConsent: ConsentType
): Promise<boolean> {
  const consent = await db.consent.findUnique({
    where: {
      identity_id_consent_type: {
        identity_id: identityId,
        consent_type: requiredConsent
      }
    }
  });

  if (!consent || !consent.granted) {
    return false;
  }

  // Check if consent version is current
  const currentVersion = await getCurrentConsentVersion(requiredConsent);
  if (consent.version !== currentVersion) {
    // Consent needs re-confirmation
    return false;
  }

  return true;
}

// Middleware for consent-gated features
function requireConsent(consentType: ConsentType) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const identityId = req.session?.identity_id;

    if (!identityId) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const hasConsent = await verifyConsent(identityId, consentType);
    if (!hasConsent) {
      return res.status(403).json({
        error: 'Consent required',
        consent_type: consentType,
        action: 'redirect_to_consent'
      });
    }

    next();
  };
}
```

### 3.4 Consent Revocation

```typescript
// Revoke consent
async function revokeConsent(
  identityId: number,
  consentType: ConsentType,
  metadata: { ip: string; userAgent: string }
): Promise<void> {
  const timestamp = new Date();

  await db.consent.update({
    where: {
      identity_id_consent_type: {
        identity_id: identityId,
        consent_type: consentType
      }
    },
    data: {
      granted: false,
      revoked_at: timestamp
    }
  });

  // Handle consequences of revocation
  if (REQUIRED_CONSENTS.includes(consentType)) {
    // Required consent revoked - must delete account
    await initiateAccountDeletion(identityId, 'consent_revoked');
  } else if (consentType === ConsentType.ANALYTICS) {
    // Stop analytics tracking
    await disableAnalyticsForUser(identityId);
  } else if (consentType === ConsentType.MARKETING) {
    // Unsubscribe from marketing
    await unsubscribeFromMarketing(identityId);
  }

  await logConsentChange(identityId, [{ type: consentType, granted: false }], metadata);
}
```

---

## 4. Data Subject Rights Implementation

### 4.1 Right of Access (Data Export)

```typescript
// Data export request
interface DataExportRequest {
  identity_id: number;
  requested_at: Date;
  format: 'json' | 'csv';
  status: 'pending' | 'processing' | 'completed' | 'failed';
  download_url?: string;
  expires_at?: Date;
}

// Generate data export
async function generateDataExport(identityId: number): Promise<string> {
  const exportData = {
    export_date: new Date().toISOString(),
    identity: await db.identity.findUnique({
      where: { id: identityId },
      select: {
        phone: true,
        display_name: true,
        avatar_url: true,
        created_at: true,
        last_active_at: true
      }
    }),
    memberships: await db.membership.findMany({
      where: { identity_id: identityId },
      select: {
        community: { select: { name: true } },
        role: true,
        joined_at: true
      }
    }),
    gamification: await db.gamification.findMany({
      where: { identity_id: identityId },
      select: {
        community_id: true,
        xp: true,
        level: true,
        current_streak: true,
        longest_streak: true
      }
    }),
    consents: await db.consent.findMany({
      where: { identity_id: identityId },
      select: {
        consent_type: true,
        granted: true,
        granted_at: true,
        version: true
      }
    }),
    sessions: await db.session.findMany({
      where: { identity_id: identityId },
      select: {
        created_at: true,
        last_activity: true,
        ip_address: true
      }
    })
  };

  // Store export file
  const filename = `export_${identityId}_${Date.now()}.json`;
  const filepath = `/srv/unofficial/exports/${filename}`;
  await fs.writeFile(filepath, JSON.stringify(exportData, null, 2));

  // Generate signed download URL (expires in 24h)
  const downloadUrl = await generateSignedUrl(filepath, 86400);

  return downloadUrl;
}
```

### 4.2 Right to Erasure (Deletion)

```typescript
// Account deletion process
enum DeletionStatus {
  REQUESTED = 'requested',
  PENDING = 'pending',           // Waiting for grace period
  IN_PROGRESS = 'in_progress',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled'
}

interface DeletionRequest {
  id: number;
  identity_id: number;
  reason: string;
  requested_at: Date;
  scheduled_at: Date;            // +7 days grace period
  completed_at?: Date;
  status: DeletionStatus;
}

// Initiate deletion
async function initiateAccountDeletion(
  identityId: number,
  reason: string
): Promise<DeletionRequest> {
  const gracePeriodDays = 7;
  const scheduledAt = new Date();
  scheduledAt.setDate(scheduledAt.getDate() + gracePeriodDays);

  // Create deletion request
  const request = await db.deletionRequest.create({
    data: {
      identity_id: identityId,
      reason,
      requested_at: new Date(),
      scheduled_at: scheduledAt,
      status: DeletionStatus.PENDING
    }
  });

  // Send confirmation email/message
  await notifyDeletionScheduled(identityId, scheduledAt);

  // Schedule deletion job
  await scheduleJob('account-deletion', {
    deletion_request_id: request.id
  }, { delay: gracePeriodDays * 24 * 60 * 60 * 1000 });

  return request;
}

// Execute deletion
async function executeAccountDeletion(requestId: number): Promise<void> {
  const request = await db.deletionRequest.findUnique({
    where: { id: requestId },
    include: { identity: true }
  });

  if (!request || request.status !== DeletionStatus.PENDING) {
    return;
  }

  await db.deletionRequest.update({
    where: { id: requestId },
    data: { status: DeletionStatus.IN_PROGRESS }
  });

  const identityId = request.identity_id;

  // Delete in order (respecting foreign keys)
  await db.$transaction([
    // 1. Anonymize analytics events
    db.analyticsEvent.updateMany({
      where: { identity_id: identityId },
      data: {
        identity_id: null,
        anonymous_id: hashForAnonymity(identityId)
      }
    }),

    // 2. Delete gamification data
    db.gamification.deleteMany({
      where: { identity_id: identityId }
    }),

    // 3. Delete missions
    db.mission.deleteMany({
      where: { identity_id: identityId }
    }),

    // 4. Delete memberships
    db.membership.deleteMany({
      where: { identity_id: identityId }
    }),

    // 5. Delete sessions
    db.session.deleteMany({
      where: { identity_id: identityId }
    }),

    // 6. Delete consents
    db.consent.deleteMany({
      where: { identity_id: identityId }
    }),

    // 7. Delete identity
    db.identity.delete({
      where: { id: identityId }
    })
  ]);

  // Clear Redis data
  await redis.del(`session:*:${identityId}`);
  await redis.del(`user:${identityId}:*`);

  // Update request status
  await db.deletionRequest.update({
    where: { id: requestId },
    data: {
      status: DeletionStatus.COMPLETED,
      completed_at: new Date()
    }
  });

  // Log for compliance
  await logDataDeletion(identityId, 'account_deletion');
}
```

### 4.3 Right to Rectification

```typescript
// Update personal data
async function updatePersonalData(
  identityId: number,
  updates: { display_name?: string; avatar_url?: string },
  metadata: { ip: string; userAgent: string }
): Promise<void> {
  const previousData = await db.identity.findUnique({
    where: { id: identityId },
    select: { display_name: true, avatar_url: true }
  });

  await db.identity.update({
    where: { id: identityId },
    data: updates
  });

  // Log change for audit
  await logDataChange({
    identity_id: identityId,
    action: 'rectification',
    previous_data: previousData,
    new_data: updates,
    ip_address: metadata.ip,
    user_agent: metadata.userAgent
  });
}
```

### 4.4 Right to Data Portability

```typescript
// Portable data format (compatible with other services)
interface PortableUserData {
  schema_version: '1.0';
  export_date: string;
  platform: 'unofficial_communities';

  profile: {
    phone_hash: string;         // Hashed for privacy
    display_name: string;
    avatar_url: string | null;
    created_at: string;
  };

  communities: Array<{
    name: string;
    role: string;
    joined_at: string;
  }>;

  activity: {
    total_xp: number;
    level: number;
    missions_completed: number;
    streak_days: number;
  };
}

async function generatePortableExport(identityId: number): Promise<PortableUserData> {
  const identity = await db.identity.findUnique({
    where: { id: identityId },
    include: {
      memberships: { include: { community: true } },
      gamification: true
    }
  });

  return {
    schema_version: '1.0',
    export_date: new Date().toISOString(),
    platform: 'unofficial_communities',

    profile: {
      phone_hash: hashForAnonymity(identity!.phone),
      display_name: identity!.display_name,
      avatar_url: identity!.avatar_url,
      created_at: identity!.created_at.toISOString()
    },

    communities: identity!.memberships.map(m => ({
      name: m.community.name,
      role: m.role,
      joined_at: m.joined_at.toISOString()
    })),

    activity: {
      total_xp: identity!.gamification.reduce((sum, g) => sum + g.xp, 0),
      level: Math.max(...identity!.gamification.map(g => g.level)),
      missions_completed: await db.mission.count({
        where: { identity_id: identityId, status: 'completed' }
      }),
      streak_days: Math.max(...identity!.gamification.map(g => g.current_streak))
    }
  };
}
```

---

## 5. Audit and Reporting

### 5.1 Audit Log Schema

```sql
CREATE TABLE compliance_audit_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    timestamp TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3),
    event_type VARCHAR(50) NOT NULL,
    identity_id INT NULL,                  -- NULL for system events
    actor_id INT NULL,                     -- Who performed action (admin, system)
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NULL,
    resource_id VARCHAR(100) NULL,
    previous_value JSON NULL,
    new_value JSON NULL,
    ip_address VARCHAR(45) NULL,
    user_agent TEXT NULL,
    request_id VARCHAR(36) NULL,           -- For tracing
    compliance_category VARCHAR(50) NOT NULL,  -- LGPD, GDPR, Security

    INDEX idx_timestamp (timestamp),
    INDEX idx_identity (identity_id, timestamp),
    INDEX idx_event_type (event_type, timestamp),
    INDEX idx_compliance (compliance_category, timestamp)
) ENGINE=InnoDB;
```

### 5.2 Auditable Events

| Event | Category | Data Logged |
|-------|----------|-------------|
| `consent.granted` | LGPD | consent_type, version, IP |
| `consent.revoked` | LGPD | consent_type, reason |
| `data.exported` | LGPD | data_types, format |
| `data.deleted` | LGPD | scope, reason |
| `data.rectified` | LGPD | field, old_value, new_value |
| `access.granted` | Security | resource, permission |
| `access.denied` | Security | resource, reason |
| `auth.login` | Security | method, IP, success |
| `auth.logout` | Security | session_duration |
| `admin.action` | Security | action, target, details |

### 5.3 Compliance Reporting

```typescript
// Generate compliance report
interface ComplianceReport {
  period_start: Date;
  period_end: Date;
  generated_at: Date;

  data_subject_requests: {
    access_requests: number;
    deletion_requests: number;
    rectification_requests: number;
    avg_response_time_days: number;
  };

  consent_statistics: {
    total_users: number;
    consented_analytics: number;
    consented_marketing: number;
    consent_rate: number;
  };

  data_breaches: {
    count: number;
    reported_to_authority: number;
    reported_to_subjects: number;
  };

  security_events: {
    failed_logins: number;
    suspicious_activities: number;
    access_violations: number;
  };
}

async function generateComplianceReport(
  startDate: Date,
  endDate: Date
): Promise<ComplianceReport> {
  // Query audit logs and aggregate data
  const dsrStats = await db.$queryRaw`
    SELECT
      COUNT(CASE WHEN action = 'data.exported' THEN 1 END) as access_requests,
      COUNT(CASE WHEN action = 'data.deleted' THEN 1 END) as deletion_requests,
      COUNT(CASE WHEN action = 'data.rectified' THEN 1 END) as rectification_requests
    FROM compliance_audit_log
    WHERE timestamp BETWEEN ${startDate} AND ${endDate}
  `;

  // ... aggregate other metrics

  return {
    period_start: startDate,
    period_end: endDate,
    generated_at: new Date(),
    data_subject_requests: dsrStats[0],
    consent_statistics: await getConsentStatistics(),
    data_breaches: await getDataBreachStats(startDate, endDate),
    security_events: await getSecurityEventStats(startDate, endDate)
  };
}
```

### 5.4 Retention Schedule

| Data Category | Retention Period | Basis | Action After |
|---------------|------------------|-------|--------------|
| Active user data | Account lifetime | Contract | Delete on account deletion |
| Inactive user data | 2 years | Legitimate interest | Anonymize or delete |
| Analytics (raw) | 90 days | Legitimate interest | Anonymize |
| Analytics (aggregated) | 5 years | Legitimate interest | Retain |
| Security logs | 2 years | Legal obligation | Delete |
| Consent records | 5 years after revocation | Legal obligation | Delete |
| Backup data | 30 days | Recovery | Overwrite |
| Deletion request records | 5 years | Legal obligation | Delete |

---

## 6. Data Breach Response

### 6.1 Breach Classification

| Severity | Definition | Notification |
|----------|------------|--------------|
| **Critical** | Personal data exposure affecting >1000 users | Authority + Subjects (72h) |
| **High** | Sensitive data access, <1000 users | Authority (72h), Subjects if risk |
| **Medium** | Non-sensitive data, limited scope | Internal only |
| **Low** | Potential breach, no confirmed exposure | Log and monitor |

### 6.2 Breach Response Procedure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA BREACH RESPONSE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HOUR 0-4: CONTAINMENT                                              │
│  ├─ Identify breach source                                          │
│  ├─ Stop ongoing breach                                             │
│  ├─ Preserve evidence                                               │
│  └─ Notify internal team                                            │
│                                                                      │
│  HOUR 4-24: ASSESSMENT                                              │
│  ├─ Determine affected data                                         │
│  ├─ Count affected users                                            │
│  ├─ Assess risk level                                               │
│  └─ Document findings                                               │
│                                                                      │
│  HOUR 24-72: NOTIFICATION                                           │
│  ├─ If high risk: notify ANPD (Brazil)                             │
│  ├─ If high risk: notify affected users                            │
│  └─ Prepare public statement if needed                              │
│                                                                      │
│  WEEK 1-4: REMEDIATION                                              │
│  ├─ Implement fixes                                                 │
│  ├─ Review security measures                                        │
│  └─ Update documentation                                            │
│                                                                      │
│  ONGOING: POST-INCIDENT                                             │
│  ├─ Monitor for misuse                                              │
│  ├─ Complete ANPD reporting                                         │
│  └─ Lessons learned review                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.3 Breach Notification Template

```markdown
## Data Breach Notification

**Date of Discovery:** [DATE]
**Date of Breach:** [DATE if known]
**Notification Date:** [DATE]

### What Happened
[Clear description of the incident]

### What Information Was Involved
- [ ] Phone numbers
- [ ] Display names
- [ ] Profile pictures
- [ ] Community membership
- [ ] Gamification data
- [ ] Other: [specify]

### What We Are Doing
1. [Containment action]
2. [Investigation action]
3. [Remediation action]

### What You Can Do
1. [User action recommendation]
2. [Contact information]

### Contact Us
Email: [privacy@unofficialbrasil.com.br]
```

---

## 7. Privacy by Design

### 7.1 Development Checklist

```markdown
## Privacy Review Checklist for New Features

### Data Collection
- [ ] Is all collected data necessary for the feature?
- [ ] Can the feature work with less data?
- [ ] Are there anonymous alternatives?

### Data Storage
- [ ] Is data encrypted at rest?
- [ ] Is retention period defined?
- [ ] Is deletion procedure documented?

### Data Access
- [ ] Is access limited to necessary personnel?
- [ ] Are access logs enabled?
- [ ] Is there role-based access control?

### User Control
- [ ] Can users view their data?
- [ ] Can users edit their data?
- [ ] Can users delete their data?
- [ ] Can users export their data?

### Third Parties
- [ ] Are third parties involved?
- [ ] Are DPAs in place?
- [ ] Is data transfer documented?

### Consent
- [ ] Is consent required?
- [ ] Is consent granular?
- [ ] Can consent be withdrawn?
```

### 7.2 Data Minimization Principles

```typescript
// Apply data minimization in queries
// BAD: Fetch all fields
const user = await db.identity.findUnique({
  where: { id: userId }
});

// GOOD: Fetch only needed fields
const user = await db.identity.findUnique({
  where: { id: userId },
  select: {
    id: true,
    display_name: true
    // Only what's needed for this operation
  }
});

// GOOD: Use projections in aggregations
const stats = await db.gamification.aggregate({
  where: { community_id: communityId },
  _avg: { xp: true },
  _count: { id: true }
  // No personal data in aggregation
});
```

---

*This document defines compliance requirements. All features must undergo privacy review before launch.*

<!-- Last Reviewed: 2026-01-16 - No updates needed -->
