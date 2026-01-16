# Product Blueprint

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## 1. Product Vision

### Mission Statement
"We know how to design healthier communities at scale, and we can prove it."

### Vision
Unofficial Communities transforms WhatsApp groups into thriving, measurable communities through ethical gamification and spatial presence, enabling community leaders to build engagement without manipulation.

### Value Proposition

| For | Who | Our Product | Unlike | Key Differentiator |
|-----|-----|-------------|--------|-------------------|
| Community Leaders | Manage WhatsApp groups of 50-500 members | Provides gamification, analytics, and 3D presence | Generic community tools that use dark patterns | Evidence-based ethical design with measurable outcomes |

---

## 2. System Goals

### Primary Goals
| Goal | Metric | Target |
|------|--------|--------|
| Increase community engagement | Daily Active Members / Total Members | >30% DAM/Total |
| Improve member retention | D30 Retention | >40% |
| Enable community discovery | Cross-community visits via portals | >10% of users visit other communities |
| Demonstrate measurable health | Health Score (composite metric) | Average >70/100 |

### Secondary Goals
| Goal | Metric | Target |
|------|--------|--------|
| Reduce admin burden | Time spent on moderation | <30 min/day for 500-member community |
| Enable monetization | Revenue per community | Sustainable unit economics |
| Build platform network effects | Communities inviting other communities | >20% viral coefficient |

---

## 3. Core Principles

### 3.1 Behavioral Science Foundation

All features must map to at least one component of the Fogg Behavior Model:

```
Behavior = Motivation × Ability × Prompt
```

**Priority Order:**
1. **Ability** (HIGH) - Reduce friction, make things easier
2. **Prompt** (MEDIUM) - Contextual, helpful triggers
3. **Motivation** (LOW) - Never artificially inflate

### 3.2 Self-Determination Theory (SDT)

Every feature must support at least one psychological need:

| Need | How We Support It |
|------|-------------------|
| **Autonomy** | Real choices, opt-out controls, no manipulation |
| **Competence** | Clear feedback, achievable goals, visible progress |
| **Belonging** | Recognition, group identity, social proof (real only) |

### 3.3 Ethical Constraints

**Forbidden Patterns:**
- FOMO messaging ("Everyone is doing this!")
- False urgency ("Only 2 spots left!")
- Implicit social pressure
- Notification spam
- Fake activity indicators
- Dark patterns of any kind

**Go / No-Go Framework:**

| ✅ GO | ❌ NO-GO |
|-------|---------|
| Reduces friction or uncertainty | Optimizes for time-on-platform as primary KPI |
| Increases clarity and user agency | Depends on behavioral addiction loops |
| Scales across communities | Cannot be explained transparently |
| Generates insights via aggregation | Would be unacceptable if disclosed publicly |

---

## 4. Feature Taxonomy

### 4.1 Authentication & Identity

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| WhatsApp Verification | Phone number verification via WhatsApp Cloud API | Implemented | P0 |
| Instagram Verification | Optional social verification via Instagram Graph API | Implemented | P1 |
| Profile Creation | Name, avatar, bio setup | Implemented | P0 |
| Session Management | Redis-backed sessions, 7-day TTL | Implemented | P0 |

### 4.2 Gamification Engine

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| XP System | Points earned for meaningful actions | Implemented | P0 |
| Levels (1-10) | Progressive titles from Novato to Lenda Suprema | Implemented | P0 |
| Daily Missions | 3-5 achievable tasks per day | Implemented | P0 |
| Streaks | Consecutive day tracking with bonuses | Implemented | P0 |
| Weekly Ranking | Community leaderboard | Implemented | P0 |
| Badges | Achievement recognition | Planned | P2 |
| Achievements | Milestone celebrations | Planned | P2 |

### 4.3 Dashboard & Analytics

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| Member Dashboard | Personal XP, streak, mission status | Implemented | P0 |
| Community Overview | Aggregate health metrics for admins | Partial | P1 |
| Health Score | Composite engagement metric | Planned | P1 |
| Heatmaps | Spatial analytics for UC World | Planned | P2 |

### 4.4 UC World (3D Presence)

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| Hexagonal Arena | 80×92 tile community space | Implemented | P0 |
| Avatar Movement | Keyboard/touch navigation | Implemented | P0 |
| Multiplayer Sync | Colyseus state synchronization | Implemented | P0 |
| Zone System | 7 interior zones + 6 portal zones | Implemented | P1 |
| Portal Navigation | Cross-community travel | Planned | P1 |
| Text Chat | Zone-based messaging | Planned | P1 |
| Voice Zones | Spatial audio (opt-in) | Planned | P2 |

### 4.5 Administration

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| Admin Panel | Community management interface | Partial | P1 |
| Member Management | View, edit, remove members | Partial | P1 |
| Moderation Tools | Report handling, warnings | Planned | P2 |
| Event Creation | Schedule community events | Planned | P2 |

### 4.6 Integration

| Feature | Description | Status | Priority |
|---------|-------------|--------|----------|
| WhatsApp Webhooks | Receive group messages for XP | Implemented | P0 |
| Instagram Webhooks | Social activity tracking | Partial | P2 |
| API for Partners | External integrations | Planned | P3 |

---

## 5. Explicit Non-Goals

The following are explicitly **NOT** goals for this product:

| Non-Goal | Rationale |
|----------|-----------|
| Replace WhatsApp | We complement, not compete with WhatsApp |
| Build a chat platform | Chat happens in WhatsApp; we add value on top |
| Maximize time-on-platform | We optimize for meaningful engagement, not attention capture |
| Support all community sizes | We focus on 50-500 members; micro or mega communities are out of scope |
| Global launch immediately | Brazil-first; internationalization is a future phase |
| Mobile native apps | Web-first; native apps only if web proves insufficient |
| Enterprise features | B2C/B2B2C focus; enterprise sales requires different product |
| AI-generated content | AI for analysis only; content must be human-created |

---

## 6. Constraints

### 6.1 Technical Constraints

| Constraint | Description | Impact |
|------------|-------------|--------|
| Single VPS | All services on one Hostinger VPS | Scaling limited, single point of failure |
| Portuguese Only | No i18n infrastructure | Brazil-only market |
| Web Only | No native mobile apps | Mobile experience via responsive web |
| Meta API Dependency | WhatsApp/Instagram features depend on Meta | Platform risk |

### 6.2 Business Constraints

| Constraint | Description | Impact |
|------------|-------------|--------|
| Bootstrap Funded | No external investment currently | Feature velocity limited |
| Small Team | <5 engineers | Scope must be focused |
| LGPD Compliance | Brazilian data protection law | Data architecture constrained |

### 6.3 Ethical Constraints

| Constraint | Description | Impact |
|------------|-------------|--------|
| No Dark Patterns | Forbidden patterns list | Some "growth hacks" unavailable |
| Transparent Analytics | Users can see how data is used | Some optimization techniques unavailable |
| Opt-Out Everything | All engagement features optional | Lower engagement ceiling possible |

---

## 7. Open Questions

### Product Questions

| ID | Question | Impact | Status |
|----|----------|--------|--------|
| OQ-001 | What is the optimal mission refresh cadence? | Engagement patterns | Needs A/B testing |
| OQ-002 | Should streaks have caps to prevent unhealthy behavior? | User wellbeing | Needs user research |
| OQ-003 | How to handle dormant communities? | Platform health | Needs policy decision |
| OQ-004 | What triggers portal recommendations? | Discovery quality | Needs algorithm design |

### Technical Questions

| ID | Question | Impact | Status |
|----|----------|--------|--------|
| OQ-005 | When to implement horizontal scaling? | Infrastructure | Depends on growth |
| OQ-006 | Should UC World support mobile touch controls? | Accessibility | Needs UX research |
| OQ-007 | How to handle state sync for 50+ users in room? | Performance | Needs load testing |

### Market Questions

| ID | Question | Impact | Status |
|----|----------|--------|--------|
| OQ-008 | What price point for premium features? | Revenue | Needs pricing research |
| OQ-009 | Which community verticals to target first? | Go-to-market | Needs market research |
| OQ-010 | How to acquire first 100 communities? | Growth | Needs growth strategy |

---

## 8. Success Criteria

### Launch Success (Phase 1)
- [ ] 10 active communities onboarded
- [ ] D7 retention >30%
- [ ] No critical bugs in production
- [ ] UC World functional for basic presence

### Growth Success (Phase 2)
- [ ] 100 active communities
- [ ] D30 retention >40%
- [ ] Health Score feature live
- [ ] Portal navigation between communities

### Scale Success (Phase 3)
- [ ] 1000 active communities
- [ ] Sustainable unit economics
- [ ] Self-service onboarding
- [ ] Platform network effects visible

---

## 9. Roadmap Reference

Detailed roadmap with phases and milestones is maintained in the repository CLAUDE.md files and tracked in SESSION_LOG.md.

Current Phase: **Foundation** (UC World Phase 1)

| Phase | Focus | Status |
|-------|-------|--------|
| Foundation | Core rendering, avatars, multiplayer | In Progress |
| Social Layer | Text chat, presence zones, UC API integration | Planned |
| Expression | Custom avatars, emotes, events | Planned |

---

*This document defines what we build and why. Implementation details live in SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md and repository CLAUDE.md files.*

<!-- Last Reviewed: 2026-01-16 - No updates needed -->
