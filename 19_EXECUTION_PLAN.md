FINAL_EXECUTION_PLAN.md

**System:** Unofficial Communities Ecosystem (UC)
**Date:** 2026-01-16
**Version:** FINAL (supersedes `EXECUTION_PLAN_v3.md` and `UC_EXECUTION_PLAN_V2_COMMUNITY_GRAPH.md`)
**Canonical governance:** `15_DECISION_LOG.md` (decisions), `16_SESSION_LOG.md` (audit trail), `13_RISK_REGISTER.md` (risk)

---

## 0. Scope, Non‑Negotiables, and How This Document Must Be Used

### 0.1 Scope (what this plan definitively delivers)

This plan defines the end-to-end execution of the **Unofficial Communities Ecosystem** as a **science-governed 3D servicescape** that enhances communities via multi-channel engagement (WhatsApp, Instagram, and other connected social channels):

* A **zone-based UC World** (hex layout, proxemic zoning, ethical prompts)
* A **Living Graph** (community-to-community graph + neighborhood portals)
* **Operational rules** for growth, splitting, and satellites (Dunbar-led)
* **Ethics-as-code constraints** (authorization, data minimization, explainability, circuit breakers)

### 0.2 Non‑negotiables (hard constraints)

1. **Science-first implementation**
   All structural decisions must map to:

   * Fogg Behavior Model (MAP) + “bias to Ability over pressure” (DEC-0007)
   * Self-Determination Theory needs: autonomy, competence, belonging (DEC-0007)
   * Proxemics as density/privacy governor (Estudo 066)
   * Dunbar constraints for community size/splitting (Estudo 040; and ASM-009)
   * Network science: community detection + embeddings + guardrails (Estudo 027; Louvain/Node2Vec sections)

2. **No theoretical invention**
   Where numeric thresholds are not present in the primary sources, this plan:

   * defines **configurable defaults** (explicitly labeled as “initial defaults”)
   * requires **calibration experiments** before automation
   * classifies each gap as **Blocking / Non-blocking but risky / Future optimization**

3. **Ethics must be enforced by code**
   Ethics must be expressed as:

   * data minimization rules
   * authorization boundaries
   * algorithmic constraints (k-anon, stability, anti-bubble)
   * feature-flag circuit breakers
     Not as policy-only language.

### 0.3 How to execute without ambiguity

* If a conflict exists between docs, the resolution path is:
  **Decision Log → Session Log → Canonical Specs** (`01_SYSTEM_CANONICAL_INDEX.md`)
* This plan includes a **“Required Canonical File Updates”** section and a **Session Log entry** to append.

---

## 1. Primary Sources Used (Authoritative Inputs)

### 1.1 System canonical documentation (must remain consistent)

* `01_SYSTEM_CANONICAL_INDEX.md`
* `02_PRODUCT_BLUEPRINT.md`
* `03_TECHNICAL_ARCHITECTURE.md`
* `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md`
* `05_USER_STATE_MACHINE.md`
* `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md`
* `07_EVENT_AND_ANALYTICS_SPEC.md`
* `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md`
* `09_SECURITY_AND_AUTHORIZATION_SPEC.md`
* `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md`
* `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md`
* `12_COMPLIANCE_AND_DATA_PROTECTION.md`
* `13_RISK_REGISTER.md`
* `14_ASSUMPTION_REGISTER.md`
* `15_DECISION_LOG.md`
* `16_SESSION_LOG.md`
* `17_CLAUDE_CODE_PROMPTS.md`
* `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md`

### 1.2 Execution plan inputs (mandatory comparison)

* `EXECUTION_PLAN_v3.md` (current draft)
* `UC_EXECUTION_PLAN_V2_COMMUNITY_GRAPH.md` (previous baseline)

### 1.3 Strategic briefing (provided in prompt; operationalized here)

* Proxemics as middleware (servicescape → events → prompts)
* Living Graph loop (events → aggregates → Louvain/Node2Vec → portals)
* Dunbar-led bootstrapping via satellites (Lendas VIP strategy)
* Ethical moat as code (anti-dark patterns, RBAC isolation, no fake social proof)
* Critical path sequence (foundation → telemetry → graph → portals → hardening)

---

## 2. v03 vs v02 Comparison (Regressions, Improvements, and Corrections)

### 2.1 Summary judgment

* **v03** improves strategic coherence and ethics framing, but regresses on **operational specificity** and **canonical alignment** (zones/events/jobs).
* **v02** is more execution-oriented for the graph pipeline but under-specifies the **behavioral translation layer** and depends on **missing referenced materials** not provided in this context.

### 2.2 Structural and conceptual comparison table

| Dimension                         | v02 (baseline)                                                                            | v03 (current draft)                                                                 | Assessment                                                                  | Final Plan Correction                                                                                                                                          |
| --------------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Canonical alignment (zones)       | Mostly aligned to DEC-0003 (Hub/Workshop/Lounges/Brand/Quiet/Onboarding + 6 portal zones) | Introduces new zone naming (Foyer/Hearth/Agora/Portal Ring)                         | **Regression**: naming drift will break implementation + analytics taxonomy | **Lock canonical zone system** (DEC-0003). Allow “UX labels” only as aliases; internal IDs remain canonical.                                                   |
| Event taxonomy                    | Explicitly flags missing chat/message events and need to instrument portal travel         | Mentions new events (e.g., dwell) not present in event spec                         | **Mixed**: v03 gives intent, but drifts from spec                           | Keep `world.zone.entered/exited`, derive dwell in aggregation (no new raw events unless justified). Add minimal WhatsApp + portal travel events to event spec. |
| Graph pipeline detail             | Strong: workstreams, constraints, integration with feature flags and jobs conceptually    | Weaker: less concrete data contracts, fewer explicit artifacts                      | **Regression**: lacks “buildable” specs                                     | Final plan includes schema, job definitions, thresholds, and flags.                                                                                            |
| Behavioral theory → system rules  | Weaker: fewer explicit mappings from Fogg/Proxemics to triggers                           | Stronger: servicescape framing, Fogg prompts, Dunbar satellites, anti-dark patterns | **Improvement** in v03                                                      | Final plan keeps v03 behavioral mapping but forces it into **zone→trigger→metric** rules and code constraints.                                                 |
| Ethics as enforceable logic       | Partial (mentions guardrails, opt-out)                                                    | Stronger (explicit anti-dark patterns; no fake social proof; explainability)        | **Improvement** in v03                                                      | Final plan turns these into enforceable constraints: RBAC, k-anon thresholds, no simulated activity, audit + circuit breakers.                                 |
| Execution sequencing              | Workstreams table but not always step-by-step                                             | Phase narrative, but some steps not testable                                        | **Both insufficient**                                                       | Final plan provides a **critical path** with explicit DOD + go/no-go + rollback gates.                                                                         |
| Missing sources / evidence chain  | References external files not provided (`estudo-graph-theory...`, `UC_EXECUTION_PLAN.md`) | Removes those but also removes some technical anchors                               | v02 has **missing inputs**; v03 has **detail dilution**                     | Final plan: explicitly flags missing sources; uses only provided sources + strategic briefing; reinstates technical anchors.                                   |
| Timezone & scheduling consistency | Suggests 02:30 UTC graph job                                                              | v03 repeats 02:30 UTC but broader system docs conflict (UTC vs BRT)                 | **System inconsistency** not resolved by either                             | Final plan standardizes scheduling rules and mandates canonical doc updates to eliminate UTC/BRT drift.                                                        |

### 2.3 Key gaps between behavioral theory and technical execution (across both v02/v03)

1. **Proxemics indices exist in research (Estudo 066) but are not operationalized into production thresholds**
   → Final plan defines measurable proxies (density, churn, retreat-to-quiet) and requires calibration before auto-enforcement.

2. **Dunbar-led splitting is asserted but not implementable without member-level clustering signals**
   → Final plan stages satellites: manual-first with dashboards; algorithmic suggestions only after data sufficiency + consent + safety validation.

3. **Graph explainability is described but not contract-defined**
   → Final plan defines “reason codes” and user-facing explanation requirements as an API contract.

---

## 3. Canonical Baseline Decisions (What Is Already Locked)

### 3.1 World topology and zones (must not drift)

* **Hexagonal community layout** with:

  * 80×92 tiles
  * **6 portal zones along edges (not corners)**
  * **7 interior zones:** Hub, Workshop, 2 Lounges, Brand, Quiet, Onboarding
    *(Locked by DEC-0003.)*

### 3.2 Behavioral frameworks are mandatory (system-level constraint)

* Fogg MAP + SDT needs as required design frameworks
* Bias toward **Ability** (reduce friction) rather than Motivation pressure
* Forbidden pattern list required
  *(Locked by DEC-0007.)*

### 3.3 Data governance is LGPD-first

* Explicit retention and anonymization paths for analytics
  *(Locked by DEC-0008; implemented in `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` and `12_COMPLIANCE_AND_DATA_PROTECTION.md`.)*

### 3.4 Architecture constraints (initial reality)

* Single VPS is the current operating constraint (ASM-001, ASM-003)
* Multi-service Docker stack (frontend, api, webhooks, world, mysql, redis)
* BullMQ-based jobs and DLQ patterns exist (`08_BACKGROUND_JOBS...`)
* Feature flags are first-class (`11_ENVIRONMENT...`)

---

## 4. Community Design Blueprint (Sizes, Growth, Splits, Satellites, Zones)

## 4.1 Target community size bands (Dunbar-led, operationalized)

**Rationale:** Cohesion degrades as group size increases without structure (Estudo 040); ASM-009 sets 50–500 as target band.

### 4.1.1 Size bands and required system behavior

| Band               |                  Members (total) | “Healthy intent”                 | Primary risks                         | System rules (enforceable)                                                                       | Metrics to monitor                          |
| ------------------ | -------------------------------: | -------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| Seed               |                             0–49 | Establish identity + rituals     | Empty-map anxiety; low belonging      | Require **guided onboarding missions** + leader programming templates                            | Activation rate; time-to-first-interaction  |
| Cohesive core      |                           50–149 | Stable belonging, recognition    | Crowding pockets; cliques             | Maintain **low-friction contribution** (Workshop prompts) + proxemic relief (Quiet/Lounges)      | Reciprocity proxy; retreat rate; reports    |
| Structured network |                          150–300 | Multiple subgroups coexist       | Fragmentation; loss of shared context | Require **sub-structure**: satellites suggested; events segmented by zone                        | Modularity proxy; lounge utilization; churn |
| Upper bound        |                          301–500 | Scaled network, still governable | Mega-community dynamics; safety risk  | **Default max_members=500** (already in schema). Any increase requires DEC entry + safety review | Report rate; admin load; portal toxicity    |
| No-Go              | >500 (without explicit decision) | Not allowed by default           | Toxic growth; cohesion collapse       | Cannot raise `max_members` without governance gate                                               | N/A                                         |

**Go/No-Go:**

* **No-Go** to raising `communities.max_members` above 500 unless:

  * health metrics stable for 30 days
  * moderation capacity confirmed
  * Living Graph guardrails active
  * DEC logged and signed off

## 4.2 Growth, splitting, and satellite spawning rules

### 4.2.1 Definitions

* **Split:** creating a new community that inherits a subset of members and a theme, reducing density in the origin.
* **Satellite:** a related community connected via portals + shared rituals, preserving “small-world” dynamics.

### 4.2.2 Split/satellite trigger policy (science-grounded, implementable)

This plan uses **two-layer triggers**:

1. **Capacity triggers** (Dunbar-led)
2. **Health triggers** (proxemics + safety + cohesion)

| Trigger type           |                     Threshold (initial) | Why (science)                                                                             | System action                                                                                   |
| ---------------------- | --------------------------------------: | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Soft Dunbar trigger    |                     total members ≥ 150 | Cohesion starts degrading without structure (Estudo 040)                                  | Surface **Admin prompt**: “Create satellite(s)?” with playbook + analytics snapshot             |
| Hard structure trigger |                     total members ≥ 300 | Multiple subgroups form; shared context weakens (Estudo 040; network fragmentation logic) | Require “structure plan”: either satellites enabled or community stays capped (no max increase) |
| Crowding trigger       |   sustained high density in Hub (proxy) | Proxemics: crowding reduces comfort and belonging (Estudo 066)                            | Increase Lounge/Quiet affordances; recommend events in Lounges; ops intervention                |
| Safety trigger         | portal-driven reports ↑ above threshold | Ethics + trust; avoid toxic growth                                                        | Circuit breaker: reduce/disable certain portal connections; log                                 |

**Important:** Algorithmic “auto-splitting” is **No-Go** in initial releases. Splitting is **admin/ops mediated** until member-level clustering signals are validated.

### 4.2.3 “Lendas VIP” bootstrapping (satellite-first launch)

Operational rule for launch:

* Launch **one cohesive core** + **2–6 satellites** (pre-created) to avoid “empty map anxiety” and preserve small-world belonging.
* Portals initially **curated** (manual mapping) until the Living Graph produces stable neighborhoods.

**Go/No-Go:** Portals must not be graph-driven until instrumentation + explainability + circuit breakers exist.

## 4.3 Zone taxonomy (canonical) and behavioral design rules

### 4.3.1 Canonical zone list (internal IDs must not change)

| Zone ID (canonical) | Zone name       | Count | Why it exists                                                  |
| ------------------- | --------------- | ----: | -------------------------------------------------------------- |
| `onboarding`        | Welcome         |     1 | Reduce friction, orient, first "competence win" (Fogg Ability) |
| `central_hub`       | Central Hub     |     1 | Discovery and public presence (servicescape "town square")     |
| `workshop`          | Workshop        |     1 | Contribution and co-creation (Ability prompts)                 |
| `social_west`       | Lounge West     |     1 | Semi-private bonding, reduce crowding                          |
| `social_east`       | Lounge East     |     1 | Semi-private bonding, reduce crowding                          |
| `brand_zone`        | Partner Space   |     1 | Contextual partner layer (brand-safe, opt-in)                  |
| `quiet_refuge`      | Quiet Room      |     1 | Refuge; proxemic relief; safety                                |
| `portal_*`          | Portal zones    |     6 | Exploration + bridging across communities                      |

> **Note:** Zone IDs are defined in `uc-world/shared/src/zones.ts`. Portal zones use the pattern `portal_<direction>` where direction is `ne`, `e`, `se`, `sw`, `w`, or `nw`.

### 4.3.2 Zone → Behavioral intent → Technical trigger → Metric mapping (required)

This is the operational contract the build must satisfy.

| Zone ID | Behavioral intent (science)                                  | Technical triggers (events/UI/system)                                                 | Success metrics (non-manipulative)                                     | Guardrails (No-Go)                                          |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------- |
| `onboarding` | Increase Ability; reduce cognitive load (Fogg)               | `world.zone.entered` → show tutorial UI; assign first mission; show "next step"       | Onboarding completion; time-to-first-hub; first mission completion     | No forced loops; no deceptive countdowns                    |
| `central_hub` | Belonging via light presence; discovery (servicescape + SDT) | `world.zone.entered` → show "what's happening" UI; density indicator; navigation cues | Unique encounters; voluntary movement to lounges/workshop; return rate | No fake "busy" signals; no simulated users                  |
| `workshop` | Competence + contribution (Fogg Facilitator prompts)         | Enter Workshop → show tools UI; mission prompts aligned to context                    | UGC created; mission completion; "help given" proxies                  | No coercive prompts; no public shaming                      |
| `social_*` | Bonding social capital; reduce crowding (proxemics)          | Enter Lounge → enable chat affordances; optional proximity voice (if implemented)     | Reciprocity proxy; repeat lounge visits; reduced hub churn             | No forced pairing; no surveillance of private conversations |
| `brand_zone` | Contextual value exchange; autonomy preserved                | Enter Brand → show opt-in offers; log aggregated visit; explicit lead consent flows   | Opt-in conversions; lead quality; user satisfaction                    | No dark patterns; no targeting; no PII exposure to partners |
| `quiet_refuge` | Refuge; safety; autonomy                                     | Enter Quiet → mute audio; reduce stimuli; offer "report/help" UI                      | Quiet usage when hub crowded; reduction in session drop-offs           | No "time-out punishment"; must be user-controlled           |
| `portal_*` | Exploration without coercion; anti-bubble bridges            | Enter portal zone → show portal card(s) with reason codes; `world.portal.activated`   | Cross-community visits; bounce rate; diversity (entropy)               | No forced travel; no hidden auto-join                       |

---

## 5. World & Living Graph Mechanics (End-to-End, Explainable, Governed)

## 5.1 Definition: “Living Graph”

The Living Graph is the **daily-updated community-to-community graph** that:

1. Ingests behavior signals (world + WhatsApp + Instagram + other connected channels + missions)
2. Aggregates into community-level features
3. Builds weighted edges between communities
4. Produces a **Neighborhood Graph of degree ≤ 6** per community
5. Drives portal assignments in the 3D world
6. Exposes **explainability** and **circuit breakers**

## 5.2 Data pipeline (contracts and components)

### 5.2.1 Pipeline overview (authoritative)

```
[WhatsApp Webhooks]          [UC World Events]
       |                          |
       v                          v
   uc-webhooks                uc-world / client
       |                          |
       +-----------> [Event Ingestion API / Collector]
                         |
                         v
                 MySQL.analytics_events (raw, consent-gated)
                         |
                         v
               Daily Aggregation Job (BullMQ, analytics queue)
                         |
                         v
                MySQL.analytics_aggregates (community-level)
                         |
                         v
          Community Graph Build Job (BullMQ, analytics queue)
                         |
                         v
     Graph Tables: community_edges + community_portals + runs (+ clusters)
                         |
                         v
           uc-world reads portal mapping (DB or API) on room init
                         |
                         v
              User sees portals + “Why this portal?” explanation
```

### 5.2.2 Data minimization rules (hard constraints)

* **No message content** is collected into analytics for graph building.
* Graph inputs must be **community-level aggregates** or **counts** (member overlap counts allowed; no identity lists stored).
* Store only:

  * weights and components (explainability)
  * aggregate counts
  * time windows
* Any “semantic” signal (topic modeling / embeddings from conversation) is **Future optimization** and requires explicit governance gates.

## 5.3 Event ingestion: what must exist (and what is missing today)

### 5.3.1 Required events for Living Graph (minimal viable set)

**World events** (mostly already in taxonomy):

* `world.presence.join`, `world.presence.leave`
* `world.zone.entered`, `world.zone.exited`
* `world.portal.activated` (must be extended for destination)
* `mission.completed` (and/or mission lifecycle events)

**Social channel-derived events** (WhatsApp implemented in `07_EVENT_AND_ANALYTICS_SPEC.md`; Instagram pending):

* `whatsapp.message.sent` (counts only)
* `whatsapp.reaction.added` (counts only)
* `whatsapp.member.joined/left` (membership dynamics)
* `instagram.comment.sent` (counts only) — pending
* `instagram.story.replied` (counts only) — pending
* `instagram.share.sent` (counts only) — pending

**Gap classification:**

* **Blocking:** Without social channel interaction counts, the graph is mostly "world-only" and risks being unrepresentative of real community dynamics. WhatsApp and Instagram are the initial primary data sources, with more channels planned.
* This plan mandates adding these events to the taxonomy with minimization.

## 5.4 Aggregation layer: community-level features (buildable now)

### 5.4.1 Required aggregate types (to create in `analytics_aggregates`)

All aggregates are keyed by: `aggregate_type`, `period_start`, and `dimensions` JSON.

| Aggregate Type             | Window | Dimensions                             | Metrics JSON (examples)                            | Purpose                             |
| -------------------------- | ------ | -------------------------------------- | -------------------------------------------------- | ----------------------------------- |
| `community_activity_daily` | 1 day  | `{community_id}`                       | `{active_identities, sessions, dwell_total_sec}`   | Baseline activity normalization     |
| `zone_dwell_daily`         | 1 day  | `{community_id, zone_id}`              | `{enter_count, exit_count, dwell_sec_bucketed}`    | Proxemics + servicescape evaluation |
| `portal_activity_daily`    | 1 day  | `{from_community_id, to_community_id}` | `{travels, bounces}`                               | Social flow signal E2               |
| `social_activity_daily`    | 1 day  | `{community_id, channel}`              | `{messages, reactions, join_events, leave_events}` | Social rhythm (WhatsApp, Instagram, other channels) |
| `mission_activity_daily`   | 1 day  | `{community_id, mission_type}`         | `{completed}`                                      | Interest/competence signature       |

**Implementation rule:**

* Dwell is derived from `zone.entered/exited` timestamps in aggregation (no new raw dwell event required).

## 5.5 Graph construction: signals, weights, stability, guardrails

### 5.5.1 Edge signals (community A ↔ community B)

**E1 — Member Overlap (Jaccard / overlap count)**

* Inputs: `members` table (status=active)
* Output: `{shared_members, jaccard}`
* Storage: store only counts and jaccard (no identity list)

**E2 — Cross-community travel flow**

* Inputs: `portal_activity_daily` aggregates
* Output: `{flow_ab, flow_ba, normalized_flow}`

**E3 — Co-activity similarity (community signatures)**

* Inputs: vectors derived from:

  * `zone_dwell_daily` distributions
  * `mission_activity_daily` distributions
  * `social_activity_daily` normalized rhythm (WhatsApp, Instagram, other channels)
* Output: cosine similarity across community vectors

**E4 — Semantic similarity (future / high risk)**

* Not implemented until governance gate and minimization proof.

### 5.5.2 Affinity formula (science-grounded; operational)

Use the Estudo 027 guidance: affinity is a fusion of components.

**Initial production formula (no semantic):**
[
A_{final} = \alpha \cdot A_{interest} + \beta \cdot A_{social}
\quad \text{with} \quad \alpha + \beta = 1
]

* (A_{social}) = normalized function of E1 + E2
* (A_{interest}) = E3 baseline (co-activity signature)
* (A_{semantic}) = 0 (disabled)

**Initial default weights (must be configurable):**

* α = 0.40 (interest/co-activity)
* β = 0.60 (social overlap/flows)

**Gap classification:**

* **Non-blocking but risky:** weights require calibration to prevent echo-chamber reinforcement.

### 5.5.3 Evidence sufficiency + privacy thresholds (hard constraints)

To prevent reidentification and low-signal noise, edges are only eligible if:

| Rule                                 |                                      Default | Why                                    |
| ------------------------------------ | -------------------------------------------: | -------------------------------------- |
| Minimum active members per community |                          ≥ 30 (k-anon proxy) | Prevent inference on small communities |
| Minimum edge evidence                | shared_members ≥ 5 OR travel_events_28d ≥ 10 | Avoid connecting by noise              |
| Rolling window                       |                                      28 days | Stability + recency balance            |
| Decay                                |   Exponential decay (half-life configurable) | Reflect evolution; avoid stale edges   |

### 5.5.4 Stability rules (anti-flapping)

Portals must not feel like an “airport” that changes constantly.

| Stability guardrail  |                                 Default | Enforcement                                        |
| -------------------- | --------------------------------------: | -------------------------------------------------- |
| Neighbor churn limit | max 2 portal changes / community / week | Graph job refuses update beyond limit; logs reason |
| Hysteresis           |        add threshold > remove threshold | Prevent oscillation                                |
| Mutuality preference |      prefer edges in each other’s top-6 | Avoid one-way “spammy” connections                 |

### 5.5.5 Anti-bubble rule (exploration slot)

Each community has 6 portal slots. The default assignment is:

* 4 slots: strongest stable neighbors (exploitation)
* 1 slot: **bridge/exploration** neighbor (diversity injection)
* 1 slot: reserved (either curated, safety override, or left empty)

**Operationalization of “20% exploration”:**

* 1 out of 6 portals = ~16.7% of slots; with UI labeling and no coercion.
* Traffic share target (20%) is a **measurement goal**, not an algorithmic forcing function.

**No-Go:** Do not optimize for portal clicks by increasing manipulation; only offer.

## 5.6 Portal appearance logic (user-facing explainability requirements)

### 5.6.1 Why portals appear (what the system must be able to say)

A portal is shown only if:

1. There is an assigned neighbor in the slot
2. The destination community is **public** OR the user is already a member OR there is an allowed “request to join” flow
3. The portal has a **reason code payload** that can be rendered to users

### 5.6.2 Explainability contract (“reason codes”)

Every portal must return:

* `reason_codes[]` (max 3)
* `reason_details` (safe, aggregate)
* `data_window_days` (e.g., 28)
* `confidence` (low/med/high based on evidence sufficiency)

**Allowed reason codes (v1):**

* `shared_members_high` (aggregate count range, not exact if small)
* `cross_visits_high` (flow-based)
* `similar_activity_patterns` (co-activity similarity)
* `curated` (manual bootstrapping / ops)
* `safety_override` (explained as “temporarily limited”)

**No-Go:** “Because we think you’ll like it” without evidence and control.

---

## 6. Ethics as System Logic (Constraints, Boundaries, and Enforcement)

## 6.1 Dark-pattern prohibitions (enforceable rules)

| Pattern                  | System rule                                                             | Enforcement location                                        |
| ------------------------ | ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| Punitive streak coercion | No public shaming; no paid streak protection; allow pause/grace         | Gamification engine + UI; feature flag for streak reminders |
| Fake social proof        | No simulated activity; NPCs must be labeled AI and excluded from counts | World rendering + analytics pipeline                        |
| Engagement hacking       | Do not optimize for time spent or message volume as success metrics     | KPI governance + dashboards                                 |
| Forced exploration       | No auto-travel; no forced portal clicks                                 | World client + server                                       |
| Deceptive opt-out        | Opt-out must be one-step accessible                                     | Consent UI + API enforcement                                |

## 6.2 Authorization boundaries (must hold system-wide)

* Cross-community isolation is default: no reading private community data without membership (`09_SECURITY_AND_AUTHORIZATION_SPEC.md`).
* Portals must not bypass membership rules.

### 6.2.1 Portal travel gating (implementable now)

When a user attempts travel from community A to B:

* Must be **ACTIVE member** of A
* For B:

  * if `visibility=public`: allow “visitor mode” **without community permissions** (read-only world presence; no member list access; no admin actions)
  * if `visibility=private/invite_only`: show join request flow; deny travel until membership granted

**Required security change:** Define an explicit permission concept for “public visit” that does not grant community resource access (see Section 10).

## 6.3 Data minimization and retention (enforced)

* Analytics raw events retention/anonymization rules must remain intact (`06_DATA_ARCHITECTURE...`).
* Graph outputs store **no PII**; only aggregates and weights.

## 6.4 Circuit breakers (required for production safety)

Circuit breakers must exist as:

* feature flags (manual kill switches)
* automated thresholds (job refuses to update, or disables portals for a community)

**Minimum required circuit breakers:**

* Disable all portals globally: `FEATURE_PORTALS=false`
* Disable graph-driven assignments: `FEATURE_COMMUNITY_GRAPH=false` (fallback to curated/empty)
* Per-community safety override (blocklist table or config)
* Neighbor churn limiter (hard stop)

---

## 6.5 Tabletop Exercise Scenarios (Step 7 Requirement)

Before pilot exit, conduct tabletop exercises to validate circuit breaker responses.

### Exercise 1: Report Spike Scenario

**Scenario:** Portal report rate suddenly exceeds 10 per 1000 travels.

| Time | Event | Expected Response |
|------|-------|-------------------|
| T+0 | Report rate crosses threshold | Alert fires, 5-min grace period starts |
| T+5min | Threshold still exceeded | `FEATURE_PORTAL_TRAVEL` auto-disabled |
| T+5min | Ops paged | On-call acknowledges |
| T+30min | Ops reviews reported portals | Identifies 3 bad portals from new community |
| T+45min | Safety override applied | `block_neighbor` for problematic community |
| T+1h | Manual re-enable | `FEATURE_PORTAL_TRAVEL=true` |
| T+24h | Verify | Report rate normalized, monitor for 24h |

**Success Criteria:**
- [ ] Alert fires within 1 minute of threshold breach
- [ ] Feature disabled automatically without human intervention
- [ ] Recovery procedure executed in <2 hours
- [ ] No user data leaked during incident

---

### Exercise 2: Edge Collapse Scenario

**Scenario:** Graph build produces 60% fewer edges than previous day.

| Time | Event | Expected Response |
|------|-------|-------------------|
| T+0 | Graph build starts (04:15) | Normal |
| T+8min | Post-build check detects edge collapse | Build flagged for rollback |
| T+8min | Previous graph restored | `community_portals` unchanged from yesterday |
| T+8min | Alert fires | Ops paged with full context |
| T+30min | Investigation | Data issue found: aggregation job partial failure |
| T+1h | Fix aggregation | Backfill missing data |
| T+2h | Manual rebuild | `dry_run=true` first |
| T+3h | Verify edge count | Within 10% of expected |
| T+4h | Commit new graph | Production updated |

**Success Criteria:**
- [ ] Rollback automatic, no broken portals served
- [ ] Root cause identified within 2 hours
- [ ] Recovery completed same day
- [ ] Dashboard shows rollback clearly

---

### Exercise 3: K-Anon Cascade Scenario

**Scenario:** Seasonal event causes 35% of communities to drop below k=30 active members.

| Time | Event | Expected Response |
|------|-------|-------------------|
| T+0 | Holiday week, activity drops | Gradual decline |
| Day 2 | 25% communities below threshold | Dashboard alert (medium) |
| Day 3 | 35% communities below threshold | Build paused by circuit breaker |
| Day 3 | Alert: K-Anon Cascade | Ops reviews, decides to wait |
| Day 5 | Activity recovering | 20% communities still below |
| Day 6 | Decision: lower threshold temporarily | `COMMUNITY_GRAPH_MIN_ACTIVE=20` |
| Day 6 | Manual build with new threshold | Success |
| Day 14 | Activity normalized | Restore `MIN_ACTIVE=30` |

**Success Criteria:**
- [ ] Portal assignments frozen (not cleared) during cascade
- [ ] Decision to adjust threshold documented
- [ ] No privacy violation from lowered threshold
- [ ] Return to normal threshold within 2 weeks

---

### Exercise 4: Coordinated Abuse Scenario

**Scenario:** Bad actor creates fake members across communities to manipulate portal assignments.

| Time | Event | Expected Response |
|------|-------|-------------------|
| T+0 | 50 new accounts join 10 communities each | Normal signup |
| Day 2 | Graph build shows unexpected edges | New edges to previously unconnected communities |
| Day 2 | Dashboard: "Edge count +40%" | Anomaly alert fires |
| Day 2 | Investigation | Common signup pattern detected |
| Day 3 | Safety override | `block_neighbor` for suspicious communities |
| Day 3 | Account review | 45 accounts flagged as suspicious |
| Day 4 | Accounts suspended | Removed from member counts |
| Day 5 | Graph rebuild | Edges normalized |

**Success Criteria:**
- [ ] Anomaly detected within 48 hours
- [ ] Bad portals blocked before users report
- [ ] No legitimate communities harmed
- [ ] Process documented for future incidents

---

### Exercise Checklist (Pre-Pilot Exit)

| Exercise | Completed | Date | Issues Found | Resolved |
|----------|-----------|------|--------------|----------|
| Report Spike | [ ] | - | - | - |
| Edge Collapse | [ ] | - | - | - |
| K-Anon Cascade | [ ] | - | - | - |
| Coordinated Abuse | [ ] | - | - | - |

**Go/No-Go Gate:** All 4 exercises must be completed with all success criteria met before pilot exit.

---

## 7. Execution Sequencing (Critical Path, Step-by-Step, Testable)

### 7.1 Critical path overview (dependencies)

| Step | Output                                                    | Depends on |
| ---: | --------------------------------------------------------- | ---------- |
|    0 | Spec alignment + repo readiness                           | none       |
|    1 | Event taxonomy complete (world + social channels + portals) | 0          |
|    2 | Consent-gated ingestion to `analytics_events`             | 1          |
|    3 | Daily aggregates (community-level features)               | 2          |
|    4 | Graph schema + build job (edges + portals + runs)         | 3          |
|    5 | Portal rendering + travel gating + explanation UI         | 4          |
|    6 | Admin/ops dashboards + health metrics                     | 5          |
|    7 | Guardrails: stability, exploration slot, circuit breakers | 6          |
|    8 | Pilot (Lendas VIP satellites) + A/B validation            | 7          |
|    9 | Scale rollout + Louvain/Node2Vec (optional gates)         | 8          |

---

## 7.2 Step-by-step execution cards (engineering + product + community ops)

### STEP 0 — Canonical spec alignment and reality audit (Blocking)

**Objective:** Eliminate known doc inconsistencies that will cause implementation drift.
**Systems involved:** system docs (`04`, `08`, `06`, `16`), deployment stack (`10`).
**Behavioral outcome:** None (this is foundation).
**Definition of done (DOD):**

* Single authoritative timezone policy chosen (UTC scheduling + explicit business timezone mapping)
* Confirm whether TimescaleDB is in use; either document it or remove from stack
* Confirm zone IDs and event names match across `04` and `07`
  **Go / No-Go:**
* **No-Go** to Living Graph work until timezone + event naming conflicts are resolved.
  **Risks if skipped:** Instrumentation mismatch, incorrect daily windows, unreliable analytics.

---

### STEP 1 — Event taxonomy upgrade (Blocking for Living Graph)

**Objective:** Make the ecosystem observable with minimal, ethical signals across all connected channels.
**Systems involved:** `uc-world`, `uc-webhooks`, `07_EVENT_AND_ANALYTICS_SPEC.md`.
**Behavioral outcome:** None directly; creates measurable signals.
**DOD:**

* Add social channel event types to taxonomy (WhatsApp + Instagram; counts-only, no content)
* Extend `world.portal.activated` to include destination community (or add `world.portal.traveled`)
* Ensure zone events use canonical zone IDs (`onboarding`, `hub`, `workshop`, `lounge_w`, `lounge_e`, `brand`, `quiet`, `portal_[dir]`)
  **Go / No-Go:**
* **No-Go** to graph build if social channel events are not instrumented (graph becomes "world-biased").
  **Risks if rushed:** Overcollection (privacy violation) or undercollection (graph useless).

---

### STEP 2 — Consent-gated event ingestion into `analytics_events` (Blocking)

**Objective:** Ensure analytics pipeline respects consent and data minimization.
**Systems involved:** `uc-api` / collector, MySQL (`analytics_events`), `12_COMPLIANCE...`.
**Behavioral outcome:** Trust; user autonomy.
**DOD:**

* Ingestion checks consent flags (at least telemetry/analytics)
* Events stored with required schema (`event_name`, `event_version`, `identity_id`, `community_id`, `properties`)
* Hard guarantee: no message content fields stored
  **Go / No-Go:**
* **No-Go** to portal recommendations without consent enforcement and audit logs.
  **Risks if skipped:** Compliance exposure; reputational harm.

---

### STEP 3 — Daily aggregation job (Blocking)

**Objective:** Produce community-level features required for graph building.
**Systems involved:** BullMQ (`analytics` queue), MySQL `analytics_aggregates`, `08_BACKGROUND_JOBS...`.
**Behavioral outcome:** Enables health measurement without profiling.
**DOD:**

* Aggregates created: `community_activity_daily`, `zone_dwell_daily`, `social_activity_daily`, `mission_activity_daily`, `portal_activity_daily`
* Derived dwell computed reliably
* Backfill for last 28 days possible (for pilot)
  **Go / No-Go:**
* **No-Go** to graph job until aggregates are stable for at least 7 consecutive daily runs.
  **Risks if skipped:** Graph instability; false connections.

---

### STEP 4 — Graph schema + build job (Edges + Portals + Runs) (Blocking)

**Objective:** Implement Living Graph outputs as durable, explainable data products.
**Systems involved:** MySQL schema migrations, BullMQ analytics worker, `FEATURE_*` flags.
**Behavioral outcome:** Community discovery that preserves autonomy.
**DOD:**

* Create graph tables (Section 8)
* Implement `communityGraphBuildDaily` job:

  * reads aggregates + member overlap
  * computes edges with evidence thresholds
  * assigns ≤6 neighbors via portal mapping
  * enforces churn limit
  * logs `community_graph_runs` with metrics + failures
* Reason codes generated per portal assignment
  **Go / No-Go:**
* **No-Go** to enabling portals if explainability payload is missing.
  **Risks if rushed:** Echo chambers; unstable neighborhoods; unsafe cross-community exposure.

---

### STEP 5 — Portal rendering + travel gating + explanation UI (Blocking)

**Objective:** Make graph outputs visible and safe inside the world.
**Systems involved:** `uc-world` (Colyseus state), client UI, security checks.
**Behavioral outcome:** Safe exploration; reduced empty-map anxiety.
**DOD:**

* World loads portal mapping on room init
* Portal card shows:

  * destination community name + visibility
  * reason codes (max 3) + data window
  * user controls (“hide this portal”, “why?”, “report”)
* Travel gating enforces visibility/membership rules
  **Go / No-Go:**
* **No-Go** to portal travel if private communities can be entered without membership.
  **Risks if skipped:** Privacy breach; trust collapse.

---

### STEP 6 — Admin/ops dashboards (Required before scale)

**Objective:** Provide operational visibility into community health and graph effects.
**Systems involved:** `uc-api`, admin dashboard, aggregates + graph tables.
**Behavioral outcome:** Leaders can intervene responsibly (not via manipulation).
**DOD:**

* Dashboards for:

  * community health (activity, onboarding, proxemic proxies)
  * portal performance (travel, bounce, report rate from portal flows)
  * neighbor stability (churn metric)
* Exportable summaries (aggregated)
  **Go / No-Go:**
* **No-Go** to expanding beyond pilot without dashboards and alert thresholds.
  **Risks if skipped:** Flying blind; safety incidents undiscovered.

---

### STEP 7 — Guardrails & circuit breakers (Required before pilot exit)

**Objective:** Enforce ethics and stability under real use.
**Systems involved:** feature flags, graph job rules, security, risk register.
**Behavioral outcome:** Trust and predictable exploration.
**DOD:**

* Implement:

  * global kill switches (`FEATURE_PORTALS`, `FEATURE_COMMUNITY_GRAPH`)
  * per-community safety overrides
  * neighbor churn limiter
  * exploration slot assignment
  * k-anon gating (min active members)
    **Go / No-Go:**
* **No-Go** to production rollout without circuit breakers tested via tabletop exercises.
  **Risks if skipped:** Irreversible reputational harm.

---

### STEP 8 — Pilot: Lendas VIP satellite-first + A/B validation (Go/No-Go gate)

**Objective:** Validate the Living Graph improves discovery and health without manipulation.
**Systems involved:** all above + community ops programming.
**Behavioral outcome:** Belonging + autonomy preserved while enabling discovery.
**DOD:**

* Pilot communities: 1 core + 2–6 satellites (curated portals initially)
* Run A/B:

  * A: graph-driven neighbor portals
  * B: curated/static portals (baseline)
* Monitor:

  * cross-community travel rate (not time spent)
  * bounce rate
  * user complaints
  * report rate changes
  * diversity proxy (entropy across destinations)
    **Go / No-Go:**
* **No-Go** if any of these occur:

  * portal-related reports exceed threshold for 7 days
  * neighbor churn feels unstable (qual + quant)
  * evidence of coercive UX patterns
    **Risks if rushed:** Confusing, unsafe neighborhood experience.

---

### STEP 9 — Scale rollout + advanced graph (Optional, gated)

**Objective:** Expand to 1,000+ communities while keeping health measurable.
**Systems involved:** infra, analytics worker scale, optional Louvain/Node2Vec.
**Behavioral outcome:** Sustainable network effects, not toxic growth.
**DOD:**

* Scale triggers defined (CPU, DB load, queue depth)
* Louvain clusters and Node2Vec embeddings only enabled when:

  * stability metrics pass
  * explainability remains meaningful
  * privacy risk assessment complete
    **Go / No-Go:**
* **No-Go** to semantic signals until separate consent/governance exists.
  **Risks:** Model-driven bubbles; reidentification; operational overload.

---

## 8. Build Specs (Schemas, Jobs, Flags) — Directly Executable

## 8.1 Database schema: Living Graph tables (MySQL)

**Goal:** Durable, queryable, explainable graph outputs.

```sql
-- Graph run tracking
CREATE TABLE community_graph_runs (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  started_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  finished_at TIMESTAMP NULL,
  status ENUM('running','success','failed') NOT NULL DEFAULT 'running',
  config JSON NULL,
  metrics JSON NULL,
  error_message TEXT NULL,
  INDEX idx_period (period_start, period_end),
  INDEX idx_status (status)
);

-- Undirected edges (store canonical ordering a<b)
CREATE TABLE community_edges (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  period_start DATE NOT NULL,
  community_a INT NOT NULL,
  community_b INT NOT NULL,
  weight DECIMAL(6,5) NOT NULL,
  components JSON NOT NULL,      -- {A_social:..., A_interest:..., E1:..., E2:..., E3:...}
  evidence JSON NOT NULL,        -- {shared_members:..., travel_28d:..., active_a:..., active_b:...}
  status ENUM('eligible','filtered','blocked') NOT NULL DEFAULT 'eligible',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_edge (period_start, community_a, community_b),
  INDEX idx_a (community_a),
  INDEX idx_b (community_b),
  FOREIGN KEY (community_a) REFERENCES communities(id),
  FOREIGN KEY (community_b) REFERENCES communities(id)
);

-- Portal mapping (degree ≤ 6)
CREATE TABLE community_portals (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  period_start DATE NOT NULL,
  community_id INT NOT NULL,
  slot ENUM('N','NE','SE','S','SW','NW') NOT NULL,
  neighbor_community_id INT NULL,
  neighbor_weight DECIMAL(6,5) NULL,
  reason_codes JSON NULL,        -- ["shared_members_high","similar_activity_patterns"]
  reason_details JSON NULL,      -- safe aggregates, ranges
  confidence ENUM('low','med','high') DEFAULT 'med',
  assignment_type ENUM('graph','curated','safety_override','empty') NOT NULL DEFAULT 'graph',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_portal (period_start, community_id, slot),
  INDEX idx_comm (community_id),
  FOREIGN KEY (community_id) REFERENCES communities(id),
  FOREIGN KEY (neighbor_community_id) REFERENCES communities(id)
);

-- Optional: per-community safety overrides
CREATE TABLE community_graph_overrides (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  community_id INT NOT NULL,
  neighbor_community_id INT NULL,
  action ENUM('block_all','block_neighbor','force_neighbor','freeze') NOT NULL,
  reason VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by INT NULL,
  INDEX idx_comm (community_id),
  FOREIGN KEY (community_id) REFERENCES communities(id),
  FOREIGN KEY (neighbor_community_id) REFERENCES communities(id),
  FOREIGN KEY (created_by) REFERENCES identities(id)
);
```

## 8.2 Job: `communityGraphBuildDaily` (BullMQ analytics queue)

**Schedule:** After daily aggregation (see Section 10 for timezone unification).
**Inputs:** `analytics_aggregates`, `members`, `communities`, overrides.
**Outputs:** `community_edges`, `community_portals`, `community_graph_runs`.

**Pseudo-logic (buildable):**

1. Determine `period_start = today-28d`, `period_end = yesterday`
2. Build community feature vectors from aggregates
3. Compute candidate edges from:

   * member overlap pairs (shared_members >= 5)
   * portal flows pairs (travel_28d >= 10)
   * co-activity similarity for candidate pairs
4. Apply eligibility rules:

   * active_members >= 30 for both communities
   * visibility constraints (optional; private communities can still connect but travel gating applies)
5. Compute weights with configurable α/β
6. For each community:

   * sort edges by weight
   * select top 4 stable neighbors (respect hysteresis + churn limit)
   * select 1 bridge neighbor (diversity slot)
   * 1 slot empty/curated/safety override
7. Write portal assignments with reason codes
8. Record run metrics + failures

## 8.3 Feature flags and configuration (must be added)

Add to `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md`:

| Flag                              | Purpose                                  | Default |
| --------------------------------- | ---------------------------------------- | ------- |
| `FEATURE_COMMUNITY_GRAPH`         | Enables graph job outputs to be consumed | false   |
| `FEATURE_PORTALS`                 | Enables portals rendering in world       | false   |
| `FEATURE_PORTAL_TRAVEL`           | Enables cross-community travel flow      | false   |
| `FEATURE_PORTAL_EXPLORATION_SLOT` | Enables 1 exploration slot               | true    |

Config parameters (env vars):

* `COMMUNITY_GRAPH_WINDOW_DAYS=28`
* `COMMUNITY_GRAPH_ALPHA=0.40`
* `COMMUNITY_GRAPH_BETA=0.60`
* `COMMUNITY_GRAPH_MIN_ACTIVE=30`
* `COMMUNITY_GRAPH_CHURN_MAX_WEEKLY=2`

---

## 9. Explicit Constraints & Anti-Patterns (What Must Not Be Built)

### 9.1 Forbidden build list (system-level No-Go)

| Item                                                       | Why it is forbidden (science/ethics)           | How it is prevented technically                                                       |
| ---------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------- |
| “Mega-community” growth beyond 500 without structure       | Dunbar cohesion degradation; toxic growth risk | `communities.max_members` default 500; increases require DEC + safety gate            |
| Optimizing for “time in world” or message volume           | Engagement ≠ health; incentivizes manipulation | Dashboards exclude attention KPIs as primary; success metrics focus on health/proxies |
| Punitive streaks / FOMO countdowns                         | Loss-aversion dark patterns                    | Remove punitive UI; allow pause; do not monetize streak protection                    |
| Fake activity, bots pretending to be users                 | Brand safety + user trust                      | NPC labeling + exclude from analytics; audit                                          |
| Portal auto-travel or forced prompts                       | Autonomy violation                             | Client disallows auto navigation; server requires explicit action                     |
| Partner access to raw PII or cross-community identity data | Privacy breach + reputational risk             | RBAC + separate analytics endpoints + aggregation-only outputs                        |
| Semantic analysis of chats without governance              | High reidentification + sensitive inference    | Feature flag off; requires new consent + DEC + compliance review                      |

---

## 10. Required Updates to Canonical Files (Specific, Actionable)

This plan requires edits to keep the system coherent. These are not optional; otherwise implementation will drift.

### 10.1 `07_EVENT_AND_ANALYTICS_SPEC.md` — REQUIRED updates

**Add:**

* Social channel event taxonomy — WhatsApp (done) + Instagram (pending) — counts-only; no message content
* Portal travel destination field:

  * Option A: extend `world.portal.activated` properties with `to_community_id`
  * Option B: add new event `world.portal.traveled`
    **Clarify:**
* Canonical zone IDs mapping (must match DEC-0003)

### 10.2 `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md` — REQUIRED updates

**Add job:**

* `community-graph-build` (analytics queue) after `event-aggregation`
  **Add alerts:**
* graph run failure rate
* neighbor churn threshold breaches
* portal safety overrides count

### 10.3 `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md` — REQUIRED updates

**Add feature flags and configs** listed in Section 8.3.

### 10.4 `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md` and `08_BACKGROUND_JOBS...` — REQUIRED conflict fix

There is a documented mismatch: `04` lists jobs in BRT; `08` lists jobs in UTC.
**Action:** Choose one canonical scheduling standard and update both docs to match.

**Mandatory rule in this plan:**

* Cron schedules are stored and executed in **UTC**, with explicit business-day conversions where needed.

### 10.5 `09_SECURITY_AND_AUTHORIZATION_SPEC.md` — REQUIRED updates

**Add explicit “public visit” authorization model** for portal travel to public communities without granting full community permissions:

* a constrained permission set for “visitor mode”
* explicit denial for private/invite_only communities

### 10.6 `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` — REQUIRED updates

**Add Living Graph tables** from Section 8.1 to the canonical schema list.
**Optional (future):** add `communities.tags JSON` if interest metadata is needed beyond co-activity.

### 10.7 `13_RISK_REGISTER.md` — REQUIRED updates

Add new risks (IDs continuing sequence):

* Echo chamber / bubble formation via portal graph
* Reidentification risk in small communities (graph inference)
* Neighborhood instability (“airport effect”)
* Portal-driven safety incidents (harassment migration)

---

## 11. Gaps, Missing Inputs, and Research Requirements

| Gap                                                                                                                      | Why it matters                                                  | Classification                        | Required action                                                                |
| ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| v02 references missing files (`estudo-graph-theory-plataformas-comunidade.md`, `UC_EXECUTION_PLAN.md`) not provided here | Evidence chain incomplete                                       | Non-blocking but must be logged       | Remove references from future docs or add those sources to canonical set       |
| Instagram events absent from analytics taxonomy (WhatsApp done)                                                          | Graph under-represents multi-channel community behavior        | **Blocking**                          | Add Instagram events (counts-only) to `07` and implement ingestion             |
| Timezone mismatch across specs (UTC vs BRT)                                                                              | Wrong daily windows → wrong missions/aggregates/graph           | **Blocking**                          | Decide and align across `04`, `08`, `06`                                       |
| Proxemics numeric thresholds not defined                                                                                 | Density guardrails can’t auto-enforce safely                    | Non-blocking but risky                | Pilot calibration: measure crowding proxies and set thresholds                 |
| Minor protection logic not representable (no age signals in schema)                                                      | Strategic briefing calls out minors; no system mechanism exists | Future optimization (high importance) | Decide whether age gating exists; if yes, add schema + consent + safety design |
| Community “interest tags” not present in schema                                                                          | Limits A_interest beyond co-activity                            | Future optimization                   | Add `communities.tags` only if needed; otherwise rely on co-activity vectors   |

---

## 12. Final Validation Checklist (Self-Verification)

* [ ] Every major decision in this plan is grounded in a canonical decision or Behavioral Science Framework (Fogg/SDT/Proxemics/Dunbar/Network science).
* [ ] v03 conceptual improvements (ethics, servicescape, satellites) are retained and operationalized into enforceable rules.
* [ ] v03 regressions (zone naming drift, event drift, missing concrete data contracts) are corrected.
* [ ] v02 strengths (pipeline thinking, flagging missing signals, governance emphasis) are retained without relying on missing external documents.
* [ ] Execution path includes step-by-step DOD + Go/No-Go criteria and is testable.
* [ ] Ethical constraints are expressed as code-enforceable boundaries (RBAC, minimization, k-anon, circuit breakers).

---

## 13. Session Log Entry to Append (Required by Governance)

Append the following to `16_SESSION_LOG.md` as a new session entry:

### SESS-2026-01-16-01 — Final Execution Plan: Living Graph + Portals + Governance

**Date:** 2026-01-16
**Owner:** Product/Systems/Science
**Status:** Proposed (pending merge into system docs)

#### Objective

Produce definitive, executable plan that unifies UC World (servicescape), Living Graph (community neighborhood portals), and ethics-as-code governance.

#### Key Decisions Captured

* Canonical zone taxonomy locked to DEC-0003; UX aliases permitted but internal IDs fixed.
* Living Graph implemented as community-level aggregates + explainable neighborhood mapping (degree ≤ 6).
* Exploration slot enforced as 1/6 portal slots; no coercive optimization.
* Portals require reason codes + travel gating; no private-community access without membership.

#### Required Canonical Doc Updates

1. `07_EVENT_AND_ANALYTICS_SPEC.md`

   * Add social channel analytics events — WhatsApp (done) + Instagram (pending) — counts-only; no content.
   * Extend portal events to capture destination community.
   * Lock zone IDs to canonical set.
2. `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md`

   * Add `community-graph-build` job and monitoring.
3. `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md`

   * Add feature flags: `FEATURE_COMMUNITY_GRAPH`, `FEATURE_PORTALS`, `FEATURE_PORTAL_TRAVEL`, `FEATURE_PORTAL_EXPLORATION_SLOT`
   * Add graph config env vars (window, weights, churn limits).
4. `09_SECURITY_AND_AUTHORIZATION_SPEC.md`

   * Add “public visit / visitor mode” permission model for portal travel.
5. `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md`

   * Add Living Graph tables: runs, edges, portals, overrides.
6. `13_RISK_REGISTER.md`

   * Add risks: echo chambers, reidentification in small communities, airport effect, portal safety migration.

#### Go/No-Go Gates Logged

* No graph-driven portals until event taxonomy + consent-gated ingestion + daily aggregates are stable.
* No portal travel until membership/visibility gating is implemented and tested.
* No semantic/chat-derived signals until governance + consent + minimization proof exists.
<!-- Last Updated: 2026-01-29 - Reframed from WhatsApp-centric to multi-channel (WhatsApp + Instagram + others); renamed whatsapp_activity_daily to social_activity_daily -->
<!-- Last Reviewed: 2026-01-26 - Full audit: Steps 0-3 100%, Step 4 85%, Step 5 75%, Step 6 80%, Step 7 80%, Step 8 50%, Step 9 15% -->
