# 22. Launch Execution Plan - Unofficial Brasil

**Document Version:** 1.0
**Created:** 24 de Janeiro de 2026
**Launch Target:** 9 de Fevereiro de 2026 (16 days)
**Status:** ACTIVE EXECUTION

---

## EXECUTIVE SUMMARY

This document consolidates the 6 launch strategy documents into an actionable execution plan. It maps current implementation status against launch requirements and identifies parallel workstreams.

### Source Documents Analyzed
1. `resumo-executivo.md` - Strategic overview, metrics, go/no-go criteria
2. `cronograma-3-semanas.md` - Day-by-day schedule
3. `estrategia-lancamento.md` - LGPD templates, website structure
4. `estrategia-completa-pt1.md` - Code templates, compliance components
5. `guia-tecnico-seguranca-seo.md` - Security checklist, SEO, performance
6. `politicas-conteudo.md` - Legal policies, landing pages, blog content

### Key Findings

| Area | Status | Gap Analysis |
|------|--------|--------------|
| **Backend (uc-api)** | 90% complete | Minor TypeScript fixes done, security audit passed |
| **3D World (uc-world)** | 70% complete | Portal travel, community branding pending |
| **Frontend (unofficial-communities)** | 60% complete | Missing: marketing pages, policies, SEO |
| **Webhooks (uc-webhooks)** | 95% complete | Token validation fixed |
| **LGPD Compliance** | 30% complete | Missing: 4 policy pages, cookie banner, age gate |
| **SEO** | 10% complete | Missing: meta tags, sitemap, GSC setup |
| **Marketing Content** | 0% complete | Missing: landing pages, blog posts |

---

## PARALLEL WORKSTREAMS

### Stream A: UC World 3D Completion (Critical Path)
**Owner:** Development Team
**Timeline:** Jan 24 - Feb 7

```
Week 1 (Jan 24-26):
├── Portal travel implementation
├── Community branding (logos, colors in world)
└── Floor routing finalization

Week 2 (Jan 27 - Feb 2):
├── Avatar customization polish
├── Event spaces implementation
└── Performance optimization (60 FPS target)

Week 3 (Feb 3-7):
├── Final QA
├── Load testing (1k concurrent)
└── Bug fixes
```

### Stream B: Website & Marketing (Parallel)
**Owner:** Can execute NOW
**Timeline:** Jan 24 - Feb 7

```
Week 1 (Jan 24-26):
├── Legal policies (4 pages)
├── Cookie banner implementation
├── Age gate component
└── DPO email setup

Week 2 (Jan 27 - Feb 2):
├── Homepage build
├── Pricing page build
├── 3 landing pages
├── Trust center

Week 3 (Feb 3-7):
├── Blog setup + 3 posts
├── SEO implementation
├── GA4 + GSC setup
└── Cross-browser testing
```

### Stream C: Security & Infrastructure (Parallel)
**Owner:** Can execute NOW
**Timeline:** Jan 24 - Feb 2

```
Week 1 (Jan 24-26):
├── SSL A+ configuration
├── Security headers (Nginx)
├── OWASP Top 10 audit
└── Rate limiting verification

Week 2 (Jan 27 - Feb 2):
├── Sentry setup
├── UptimeRobot setup
├── /api/health endpoint
└── Backup verification
```

---

## CURRENT IMPLEMENTATION STATUS

### Already Implemented (from audit)

#### Backend (uc-api)
- [x] Fastify server with CORS, rate limiting
- [x] Prisma ORM (no raw SQL injection risk)
- [x] JWT authentication
- [x] WhatsApp OTP flow
- [x] Gamification engine (XP, missions, streaks)
- [x] Living Graph analytics
- [x] BullMQ job processing
- [x] Admin authentication (ensureAdminAuth)
- [x] Health check endpoint
- [x] Correlation ID middleware
- [x] Security audit fixes (today)

#### Frontend (unofficial-communities)
- [x] Next.js 14.2.35 (security update done today)
- [x] Member dashboard
- [x] Admin dashboard
- [x] BFF session management (Redis-backed)
- [x] Admin auth library (requireAdminAuth)
- [x] Algorithms page auth fixed (today)
- [x] Health check API route

#### 3D World (uc-world)
- [x] Colyseus server
- [x] Three.js rendering
- [x] Avatar system
- [x] Floor system
- [x] Basic navigation
- [x] WebSocket rooms

#### Webhooks (uc-webhooks)
- [x] Meta WhatsApp integration
- [x] Signature verification
- [x] Event mapping to social events
- [x] BullMQ queue integration
- [x] Token validation fixed (today)

### Missing Implementation

#### Legal/Compliance (Priority: CRITICAL)
- [ ] Privacy Policy page (`/privacidade`)
- [ ] Terms of Use page (`/termos`)
- [ ] Cookie Policy page (`/cookies`)
- [ ] Refund Policy page (`/reembolso`)
- [ ] Cookie consent banner component
- [ ] Age gate component (18+)
- [ ] Footer with company data (CNPJ, address, DPO)
- [ ] DPO email (dpo@unofficialbrasil.com.br)

#### Marketing Pages (Priority: HIGH)
- [ ] Homepage (hero, features, pricing CTA, FAQ)
- [ ] Pricing page (4 tiers, comparison table)
- [ ] Landing: E-commerce (`/solucoes/ecommerce`)
- [ ] Landing: Infoproducers (`/solucoes/infoprodutores`)
- [ ] Landing: Agencies (`/solucoes/agencias`)
- [ ] Trust Center (`/seguranca`)
- [ ] About/Manifesto (`/sobre`)
- [ ] Contact page (`/contato`)

#### Blog (Priority: MEDIUM)
- [ ] Blog infrastructure (MDX or Markdown)
- [ ] Post: Gamificação Ética
- [ ] Post: Guia de Retenção
- [ ] Post: Living Graph Explicado

#### SEO (Priority: HIGH)
- [ ] Meta tags on all pages
- [ ] Dynamic sitemap.xml
- [ ] robots.txt
- [ ] Structured data (Schema.org)
- [ ] OG images (1200x630)
- [ ] Google Search Console verification
- [x] Google Analytics 4 setup

#### Security/Infrastructure (Priority: CRITICAL)
- [ ] SSL A+ rating verification
- [ ] Security headers in Nginx
- [ ] Sentry error tracking
- [ ] UptimeRobot monitoring
- [ ] Performance audit (Lighthouse > 90)

---

## IMMEDIATE ACTIONS (Next 48 Hours)

### Day 1 (Today - Jan 24)

#### Morning (4 hours)
1. **Create policy pages structure**
   ```
   src/app/(site)/privacidade/page.tsx
   src/app/(site)/termos/page.tsx
   src/app/(site)/cookies/page.tsx
   src/app/(site)/reembolso/page.tsx
   ```

2. **Implement Cookie Banner component**
   - Use template from `estrategia-completa-pt1.md`
   - Add to root layout
   - Test consent flow

3. **Implement Age Gate component**
   - Use template from `estrategia-completa-pt1.md`
   - Integrate with signup flow

#### Afternoon (4 hours)
4. **Create footer with legal info**
   - Company data (CNPJ, address)
   - Links to policy pages
   - DPO email
   - Social media links

5. **Setup DPO email**
   - Configure dpo@unofficialbrasil.com.br
   - Setup auto-responder

6. **SSL/Security verification**
   - Test at ssllabs.com
   - Add security headers to Nginx
   - Verify HTTPS redirect

### Day 2 (Jan 25)

#### Morning (4 hours)
1. **Copy policy content to pages**
   - Use templates from `estrategia-lancamento.md`
   - Adapt with real company data
   - Style consistently

2. **Test LGPD flows**
   - Signup with age verification
   - Cookie consent blocks GA4
   - Data export request flow

#### Afternoon (4 hours)
3. **Sentry setup**
   - Create project
   - Install SDK
   - Test error capture

4. **UptimeRobot setup**
   - Monitor https://unofficialbrasil.com.br
   - Monitor /api/health
   - Configure email alerts

5. **Backup verification**
   - Test restore procedure
   - Verify cron jobs running

---

## WEEK 1 DETAILED SCHEDULE (Jan 24-26)

### Friday, Jan 24 (Today)

| Time | Task | Owner | Deliverable |
|------|------|-------|-------------|
| 09:00-13:00 | Policy pages + Cookie Banner + Age Gate | Dev | 4 pages + 2 components |
| 14:00-18:00 | Footer + DPO email + SSL check | Dev | Footer live, SSL A+ |

### Saturday, Jan 25

| Time | Task | Owner | Deliverable |
|------|------|-------|-------------|
| 09:00-13:00 | Policy content + LGPD testing | Dev | Policies live, tested |
| 14:00-18:00 | Sentry + UptimeRobot + Backups | Dev | Monitoring active |

### Sunday, Jan 26

| Time | Task | Owner | Deliverable |
|------|------|-------|-------------|
| 09:00-13:00 | Homepage structure | Dev | Homepage live (basic) |
| 14:00-18:00 | Week 1 review + planning | Dev | GO/NO-GO decision |

### Week 1 GO/NO-GO Checklist

- [x] 4 policy pages published
- [x] Cookie banner working (blocks GA4 without consent)
- [x] Age gate blocking <18
- [x] Footer with company data
- [x] SSL A+ rating
- [x] Security headers configured
- [x] Sentry capturing errors
- [x] UptimeRobot monitoring active
- [x] Backups verified

**Week 1: ALL PASSED (verified 2026-01-26)**

---

## WEEK 2 SCHEDULE (Jan 27 - Feb 2)

### Marketing Pages Sprint

| Day | Pages | Hours |
|-----|-------|-------|
| Mon 27 | Homepage complete | 8h |
| Tue 28 | Pricing page | 8h |
| Wed 29 | Landing: E-commerce + Infoproducers | 8h |
| Thu 30 | Landing: Agencies + Trust Center | 8h |
| Fri 31 | Blog setup + SEO basics | 8h |
| Sat 1 | 3 blog posts | 8h |
| Sun 2 | GA4 + GSC + Week 2 review | 8h |

### Week 2 GO/NO-GO Checklist

- [x] Homepage complete (hero, features, pricing CTA, FAQ)
- [x] Pricing page with 3 tiers (Starter R$0, Pro R$149, Business R$499)
- [x] 3 landing pages live (ecommerce, infoprodutores, agencias)
- [x] Trust center published (/seguranca)
- [x] Blog with 3 posts (gamificacao-etica, guia-retencao, living-graph)
- [x] SEO basics (meta tags, sitemap with 16 URLs, robots.txt, JSON-LD structured data)
- [x] GA4 tracking active (G-H2G0PRBYS2 configured, consent-gated)
- [x] GSC verified + sitemap submitted
- [ ] Mobile responsive on all pages (needs manual testing)

---

## WEEK 3 SCHEDULE (Feb 3-9)

### Final Polish & Launch

| Day | Tasks |
|-----|-------|
| Mon 3 | Performance optimization (Lighthouse > 90) |
| Tue 4 | Security re-audit + Cross-browser testing |
| Wed 5 | LGPD end-to-end testing + Accessibility audit |
| Thu 6 | Support structure + Monitoring dashboard |
| Fri 7 | **FINAL GO/NO-GO DECISION** |
| Sat-Sun | Launch prep (announcements, social posts) |
| **Mon 9** | **LAUNCH DAY - 2 PM BRT** |

---

## LAUNCH DAY RUNBOOK

### T-1 Hour (1 PM BRT)
- [ ] Database: Connected ✓
- [ ] Redis: Connected ✓
- [ ] API: Responding ✓
- [ ] Frontend: Loading ✓
- [ ] UC World: Loading ✓
- [ ] Monitoring: Active ✓
- [ ] Backups: Latest successful ✓

### T-30 Min (1:30 PM)
- [ ] Notify team: "Going live in 30 min"
- [ ] Stand by for problems
- [ ] Prepare rollback if needed

### T-0 (2 PM) - LAUNCH
- [ ] Verify: unofficialbrasil.com.br loads
- [ ] Test: Signup flow
- [ ] Test: UC World access
- [ ] Send launch email (if waitlist exists)
- [ ] Post on social media

### T+15 Min
- [ ] Check error rate < 0.5%
- [ ] Check response time < 200ms
- [ ] Verify signups tracking

### T+1 Hour
- [ ] First status update on social
- [ ] Update status page

### T+4 Hours (6 PM)
- [ ] End of day review
- [ ] Total signups count
- [ ] Error analysis
- [ ] Support tickets count
- [ ] Team standup

---

## SUCCESS METRICS (Month 1)

| Metric | Target | Tracking |
|--------|--------|----------|
| Trial Signups | ≥ 50 | Dashboard |
| Activation Rate | ≥ 50% | % completing onboarding |
| D7 Retention | ≥ 40% | % returning after 7 days |
| D30 Retention | ≥ 20% | % still active after 30 days |
| Paid Conversion | ≥ 8% | % trials → paid |
| Uptime | ≥ 99.5% | UptimeRobot |
| Error Rate | < 0.5% | Sentry |
| Support Response | < 2h | Email tracking |
| Security Incidents | 0 | Manual monitoring |
| LGPD Violations | 0 | Compliance audit |

---

## RISK REGISTER (Launch-Specific)

### Critical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| LGPD violation | Low (2/10) | Game-over | Use templates, test e2e |
| Security breach | Low (2/10) | High | OWASP audit, rate limiting |
| UC World not ready | Medium (4/10) | High | Parallel workstreams, MVP scope |
| Low conversion | Medium (4/10) | High | A/B test, iterate fast |

### Contingency Plans

**If UC World not ready by Feb 7:**
- Launch with "Coming Soon" for 3D world
- Focus on gamification + dashboard
- UC World goes live in Week 2 post-launch

**If critical security issue found:**
- Delay launch
- Fix within 24-48h
- Re-audit before launch

**If conversion < 1%:**
- A/B test copy immediately
- Simplify signup flow
- Add social proof
- Consider pricing adjustment

---

## DOCUMENT CROSS-REFERENCE

| Need | Reference Document |
|------|-------------------|
| Policy templates | `estrategia-lancamento.md` Section 1 |
| Cookie banner code | `estrategia-completa-pt1.md` Section 1.3 |
| Age gate code | `estrategia-completa-pt1.md` Section 1.4 |
| Homepage JSX | `estrategia-completa-pt1.md` Section 2.2 |
| Pricing page JSX | `estrategia-completa-pt1.md` Section 2.3 |
| Security checklist | `guia-tecnico-seguranca-seo.md` Part 1 |
| SEO implementation | `guia-tecnico-seguranca-seo.md` Part 2 |
| Performance tuning | `guia-tecnico-seguranca-seo.md` Part 3 |
| Monitoring setup | `guia-tecnico-seguranca-seo.md` Part 4 |
| Landing pages copy | `politicas-conteudo.md` Marketing section |
| Blog post content | `politicas-conteudo.md` Blog section |
| Day-by-day tasks | `cronograma-3-semanas.md` |
| Go/no-go criteria | `resumo-executivo.md` |

---

## VALIDATION NOTES

### Content Quality Assessment

1. **LGPD Templates** - Complete and accurate for Brazilian law
2. **Security Checklist** - Comprehensive OWASP Top 10 coverage with code solutions
3. **SEO Guide** - Practical Next.js implementation
4. **Marketing Copy** - Professional, benefit-focused, CTA-driven
5. **Pricing Tiers** - Clear differentiation, competitive for Brazil market

### Gaps Identified

1. **Payment Gateway** - No integration code provided (Stripe, Pagar.me, etc.)
2. **Email Sequences** - Templates mentioned but not provided
3. **Referral Program** - Mentioned in post-launch but no spec
4. **White-label** - Enterprise feature mentioned but no implementation guide

### Recommendations

1. Start with Stripe or Pagar.me for payments (established in Brazil)
2. Use Resend or SendGrid for transactional emails
3. Defer referral program to Month 2
4. White-label can wait for Enterprise customers

---

## NEXT IMMEDIATE ACTION

**Start NOW with Day 1 Morning tasks:**

1. Create policy page files
2. Implement CookieBanner component
3. Implement AgeGate component

**Command to begin:**
```bash
cd /srv/unofficial/prod/app/unofficial-communities
# Create policy pages
mkdir -p src/app/\(site\)/privacidade
mkdir -p src/app/\(site\)/termos
mkdir -p src/app/\(site\)/cookies
mkdir -p src/app/\(site\)/reembolso
```

---

**Document Status:** Ready for execution
**Next Review:** Jan 26 (Week 1 GO/NO-GO)
**Owner:** Development Team
**Approval:** Pending user confirmation

---

*This document consolidates estrategia-lancamento.md, cronograma-3-semanas.md, guia-tecnico-seguranca-seo.md, estrategia-completa-pt1.md, politicas-conteudo.md, and resumo-executivo.md into an actionable execution plan.*

<!-- Last Updated: 2026-01-28 - GA4 measurement ID configured (G-H2G0PRBYS2), Week 2 checklist 8/9 done (mobile testing pending) -->
