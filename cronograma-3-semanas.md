# Unofficial Brasil: Cronograma de 3 Semanas (Detalhado)

**Data inicial:** 23 de Janeiro de 2026  
**Data de lançamento:** 9 de Fevereiro de 2026  
**Modelo:** Bootstrap (você fazendo tudo)

---

## SEMANA 1: LEGAL & COMPLIANCE (23-26 Jan)

### Segunda-feira, 23 Jan

**Manhã (4 horas)**
- [ ] Criar políticas usando templates deste documento
  - Política de Privacidade (copiar + adaptar com seus dados)
  - Termos de Uso (copiar + adaptar)
  - Política de Cookies (copiar + adaptar)
  - Política de Reembolso (copiar + adaptar)
- [ ] Preencher dados da empresa:
  - Razão Social
  - CNPJ
  - Endereço
  - Email de contato
  - WhatsApp

**Tarde (4 horas)**
- [ ] Configurar email dpo@unofficialbrasil.com.br
  - Criar conta Gmail/Google Workspace
  - Configurar resposta automática
  - Testar envio/recebimento
- [ ] Criar rodapé do site (HTML/Next.js)
  - Adicionar links para políticas
  - Adicionar dados da empresa
  - Commitar no git

### Terça-feira, 24 Jan

**Manhã (4 horas)**
- [ ] Converter políticas para HTML/Markdown
  - Criar páginas: /privacidade, /termos, /cookies, /reembolso
  - Formatar com CSS básico (legível, profissional)
  - Deploy para staging

**Tarde (4 horas)**
- [ ] Implementar banner de cookies
  - Criar componente JavaScript
  - Testar: Aceitar todos / Apenas essenciais
  - Verificar que GA4 não carrega sem consentimento
- [ ] Testar LGPD flows
  - Signup com data de nascimento
  - Verificar age gate (bloqueia < 18)
  - Testar checkbox de consentimento (não pré-marcado)

### Quarta-feira, 25 Jan

**Manhã (4 horas)**
- [ ] Configurar SSL/TLS (Let's Encrypt)
  - Instalar certbot
  - Gerar certificado para unofficialbrasil.com.br
  - Configurar Nginx
  - Testar: https funciona, redirect HTTP→HTTPS

**Tarde (4 horas)**
- [ ] Configurar security headers (Nginx)
  - Adicionar todos headers do documento
  - Testar em https://securityheaders.com
  - Meta: Grade A mínimo
- [ ] Setup monitoring básico
  - Sentry: criar projeto, instalar SDK
  - Testar: forçar erro, verificar aparece no Sentry

### Quinta-feira, 26 Jan

**Manhã (4 horas)**
- [ ] Auditoria de segurança interna (OWASP Top 10)
  - Executar checklist deste documento
  - Testar SQL injection, access control, rate limiting
  - Documentar findings

**Tarde (4 horas)**
- [ ] Corrigir issues encontrados na auditoria
- [ ] Verificar backups funcionando
  - Testar restore de backup
  - Confirmar cron de backup diário ativo
- [ ] GO/NO-GO Week 1:
  - [ ] Políticas publicadas?
  - [ ] SSL A+ rating?
  - [ ] DPO email funcionando?
  - [ ] Security headers OK?
  - [ ] LGPD flows testados?
  - **Se tudo OK → Semana 2. Se não → Fix bloqueadores.**

---

## SEMANA 2: WEBSITE & MARKETING (27 Jan - 2 Fev)

### Segunda-feira, 27 Jan

**Manhã (4 horas)**
- [ ] Build homepage - Hero section
  - Headline + subheadline
  - 2 CTAs (Começar Grátis + Ver Demo)
  - Screenshot/vídeo do produto
  - Deploy para staging

**Tarde (4 horas)**
- [ ] Build homepage - Como Funciona
  - 5 cards com ícones
  - Texto benefício-focado (não feature-focado)
  - Links internos

### Terça-feira, 28 Jan

**Manhã (4 horas)**
- [ ] Build página de Preços
  - 4 tiers (Starter, Professional, Business, Enterprise)
  - Tabela de comparação
  - FAQ de cobrança
  - CTAs para cada plano

**Tarde (4 horas)**
- [ ] Build landing page: Para E-commerce
  - Copiar estrutura do documento
  - Problema + Solução + Caso de uso
  - CTA principal
  - Deploy

### Quarta-feira, 29 Jan

**Manhã (4 horas)**
- [ ] Build landing page: Para Infoprodutores
  - Copiar estrutura do documento
  - Casos de uso específicos
  - Deploy
- [ ] Build landing page: Para Agências
  - Copiar estrutura do documento
  - Deploy

**Tarde (4 horas)**
- [ ] Build Trust Center (/seguranca)
  - Copiar conteúdo do documento
  - Adicionar seções: Segurança, Privacidade, 18+, Status
  - Deploy

### Quinta-feira, 30 Jan

**Manhã (4 horas)**
- [ ] Setup blog infraestrutura
  - Next.js MDX ou Markdown
  - Layout de post
  - Lista de posts
- [ ] Publicar 3 blog posts
  - Post 1: Gamificação Ética
  - Post 2: Guia de Retenção
  - Post 3: Living Graph Explicado

**Tarde (4 horas)**
- [ ] SEO básico
  - Adicionar meta tags em todas páginas (title, description)
  - Criar sitemap.xml
  - Criar robots.txt
  - Adicionar structured data (Schema.org)

### Sexta-feira, 31 Jan

**Manhã (4 horas)**
- [ ] Google Search Console setup
  - Verificar domínio
  - Submeter sitemap
- [ ] Google Analytics 4 setup
  - Criar propriedade
  - Instalar tracking code
  - Testar eventos

**Tarde (4 horas)**
- [ ] Cross-browser testing
  - Chrome, Firefox, Safari, Edge
  - Desktop + mobile (3 tamanhos)
  - Documentar bugs
- [ ] GO/NO-GO Week 2:
  - [ ] Homepage completa?
  - [ ] Preços + 3 landing pages OK?
  - [ ] Trust Center publicado?
  - [ ] Blog com 3 posts?
  - [ ] SEO básico implementado?
  - [ ] Mobile responsivo?
  - **Se tudo OK → Semana 3. Se não → Fix critical items.**

---

## SEMANA 3: TESTING & LAUNCH (3-9 Fev)

### Segunda-feira, 3 Fev

**Manhã (4 horas)**
- [ ] Performance optimization
  - Lighthouse audit (meta: >90)
  - Comprimir imagens (WebP)
  - Lazy loading
  - Minify CSS/JS

**Tarde (4 horas)**
- [ ] Core Web Vitals check
  - LCP < 2.5s
  - FID < 100ms
  - CLS < 0.1
  - Fix issues

### Terça-feira, 4 Fev

**Manhã (4 horas)**
- [ ] Security re-audit
  - Re-run OWASP Top 10 checklist
  - npm audit (fix all critical/high)
  - Secrets audit (nenhum secret em código)

**Tarde (4 horas)**
- [ ] Rate limiting implementation
  - Implementar código do documento
  - Testar endpoints críticos
  - Verificar Redis funcionando

### Quarta-feira, 5 Fev

**Manhã (4 horas)**
- [ ] LGPD end-to-end testing
  - Test 1: Signup → Age check → Consent → Database
  - Test 2: Data export request
  - Test 3: Data deletion request
  - Test 4: Cookie consent blocks analytics

**Tarde (4 horas)**
- [ ] Accessibility audit (WCAG AA)
  - Contrast checker (4.5:1 mínimo)
  - Keyboard navigation
  - Screen reader test (básico)
  - Alt text em todas imagens

### Quinta-feira, 6 Fev

**Manhã (4 horas)**
- [ ] Setup support structure
  - Email contato@ monitored
  - WhatsApp Business configurado
  - FAQ page criada
  - Auto-responder configurado

**Tarde (4 horas)**
- [ ] Monitoring dashboard setup
  - Sentry: Alertas configurados
  - Uptime monitor: Status page público
  - Datadog/New Relic (free tier) OR custom logs
  - Test: Forçar erro, verificar alerta

### Sexta-feira, 7 Fev

**Dia todo (8 horas)**
- [ ] Final QA walkthrough
  - Signup flow completo (OTP via WhatsApp)
  - Age verification funciona
  - Gamification básica funciona
  - UC World 3D carrega (sem erros)
  - Database conectado
  - Redis conectado
  - Backups testados
  - Rollback procedure testado
- [ ] Performance final check
  - Lighthouse: All green (90+)
  - SSLLabs: A+ rating
  - SecurityHeaders: A rating
  - Core Web Vitals: All green
- [ ] FINAL GO/NO-GO GATE:
  - [ ] Uptime staging > 99% última semana?
  - [ ] Error rate < 0.5%?
  - [ ] Zero critical security issues?
  - [ ] LGPD flows 100% testados?
  - [ ] Mobile + desktop funcionando?
  - [ ] Support channels prontos?
  - [ ] Monitoring live?
  - **Se TUDO OK → LAUNCH Segunda (9 Fev)**
  - **Se 1+ blocker → Delay launch, fix issues**

### Sábado-Domingo, 8-9 Fev

**Preparação final**
- [ ] Launch announcement preparado
  - Email para waitlist (se existir)
  - Posts sociais agendados (Instagram, LinkedIn, Twitter)
  - Press release draft (opcional)
- [ ] Team briefing
  - Todos sabem horário de launch
  - On-call schedule definido (quem responde incidentes)
  - Communication channels prontos (Slack, WhatsApp)

---

## SEGUNDA-FEIRA, 9 FEV - LAUNCH DAY 🚀

### 2 PM BRT (Horário de Launch)

**T-1 hora (1 PM)**
- [ ] Final health check
  - Database: Connected ✓
  - Redis: Connected ✓
  - API: Responding ✓
  - Frontend: Loading ✓
  - Monitoring: Active ✓
  - Backups: Latest successful ✓

**T-30 min (1:30 PM)**
- [ ] Notificar team: "Going live in 30 min"
- [ ] Stand by para problemas

**T-0 (2 PM) - LAUNCH**
- [ ] Deploy para produção (se não já está)
- [ ] Verify: unofficialbrasil.com.br loads
- [ ] Test critical flow: Signup
- [ ] Send launch email (se tiver lista)
- [ ] Post em redes sociais
  - Instagram: "🚀 Oficial Brasil está AO VIVO!"
  - LinkedIn: Post profissional sobre launch
  - Twitter: Thread explicando produto

**T+15 min (2:15 PM)**
- [ ] Monitor dashboard
  - Error rate: < 0.5%? ✓
  - Response time: < 200ms? ✓
  - Signups: Tracking? ✓

**T+1 hora (3 PM)**
- [ ] First status update
  - Twitter: "All systems operational 🟢"
  - Status page atualizado

**T+4 horas (6 PM)**
- [ ] End of day review
  - Total signups: [#]
  - Errors: [#] (investigar se > 10)
  - Support tickets: [#]
  - Performance: OK/Issues?
  - Team standup: Discuss findings

---

## PRIMEIRA SEMANA PÓS-LAUNCH (10-16 Fev)

### Daily Tasks

**Todo dia (Manhã)**
- [ ] Check metrics dashboard
  - Signups: [#]
  - Error rate: [%]
  - Uptime: [%]
  - Support tickets: [#]
- [ ] Review Sentry errors
  - Prioritize critical
  - Fix high-impact bugs
- [ ] Respond support tickets (< 2h SLA)

**Todo dia (Tarde)**
- [ ] Product improvements
  - Based on user feedback
  - Fix UX issues
  - Iterate copy se conversão baixa
- [ ] Content marketing
  - Engage on social media
  - Respond comments
  - Share user stories

**Sexta-feira (Week 1 Review)**
- [ ] Metrics review meeting
  - Signups: Total + daily rate
  - Activation: % completing onboarding
  - Retention: D7 retention rate
  - Churn: % leaving in first week
  - CAC: Cost per signup (if running ads)
- [ ] Decisions:
  - Continue as is? (if metrics good)
  - Pivot strategy? (if metrics bad)
  - Focus areas for Week 2?

---

## MÉTRICAS DE SUCESSO (MÊS 1)

**If these metrics hit → Ready to scale**

| Metric | Target | Status |
|--------|--------|--------|
| **Trial Signups** | ≥ 50 | [ ] |
| **Activation Rate** | ≥ 50% | [ ] |
| **D7 Retention** | ≥ 40% | [ ] |
| **D30 Retention** | ≥ 20% | [ ] |
| **Paid Conversion** | ≥ 8% | [ ] |
| **Uptime** | ≥ 99.5% | [ ] |
| **Error Rate** | < 0.5% | [ ] |
| **Support Response** | < 2h | [ ] |
| **Security Incidents** | 0 | [ ] |
| **LGPD Violations** | 0 | [ ] |

**Dashboard:** Track daily em spreadsheet ou Datadog.

---

## CONTINGENCY PLANS

### Se Error Rate > 1%

1. Identify source (Sentry)
2. If critical (blocks signup/login): **ROLLBACK**
3. If non-critical: Fix within 24h
4. Post-mortem: Document what happened

### Se Uptime < 95%

1. Check server health: `docker ps`, `docker logs`
2. Check database: `docker exec mysql mysql -u root -p -e "SELECT 1"`
3. If database down: Restore from backup
4. If API down: Restart containers
5. If recurring: Investigate bottleneck (CPU, memory, disk)

### Se Conversion < 1%

1. A/B test copy (hero headline, CTA text)
2. Simplify signup flow (remove friction)
3. Add social proof (testimonials, logos)
4. Pricing: Consider lower barrier (R$ 0 → R$ 49)

### Se Support Overwhelmed (> 20 tickets/day)

1. Create FAQ for common issues
2. Setup automated responses
3. Prioritize critical issues
4. Consider hiring support person (VA, R$ 15-20/h)

---

## RECURSOS NECESSÁRIOS

### Time Estimate (Total)

| Week | Hours | Tasks |
|------|-------|-------|
| Week 1 | 40h | Legal, compliance, security |
| Week 2 | 40h | Website, landing pages, SEO |
| Week 3 | 40h | Testing, QA, monitoring, launch prep |
| **Total** | **120h** | **3 weeks @ 40h/week** |

**Se você tem equipe:** Dividir tarefas entre frontend, backend, marketing.
**Se você está sozinho:** Focar no critical path, aceitar MVP imperfeito.

### Tools (Free/Low-Cost)

| Tool | Use | Cost |
|------|-----|------|
| Let's Encrypt | SSL certificate | Grátis |
| Google Analytics 4 | Website analytics | Grátis |
| Google Search Console | SEO tracking | Grátis |
| Sentry | Error tracking | Grátis (5k errors/mo) |
| Uptime Robot | Uptime monitoring | Grátis (50 monitors) |
| Canva | Design (OG images, posts) | Grátis |
| Gmail/Google Workspace | Email (DPO, contato) | R$ 12/mês |
| Hostinger VPS | Server | R$ 50-100/mês |

**Total recurring:** ~R$ 100-150/mês

---

## CHECKLIST FINAL (Launch Readiness)

**Legal/Compliance**
- [ ] Política de Privacidade (PT-BR)
- [ ] Termos de Uso (18+ explícito)
- [ ] Política de Cookies (com banner)
- [ ] Política de Reembolso
- [ ] Dados da empresa no rodapé (CNPJ, endereço, DPO)
- [ ] LGPD data flows testados

**Website**
- [ ] Home page completa (hero, features, pricing CTA, FAQ)
- [ ] Pricing page (4 tiers, claro, CTAs)
- [ ] 3 landing pages (/ecommerce, /infoprodutores, /agencias)
- [ ] Trust Center (/seguranca)
- [ ] About/Manifesto (/sobre)
- [ ] Blog (3+ posts)
- [ ] SEO (meta tags, sitemap, structured data)
- [ ] Mobile responsive (all pages)
- [ ] Accessibility (WCAG AA)

**Security**
- [ ] SSL A+ rating
- [ ] Security headers configurados
- [ ] Rate limiting ativo
- [ ] Nenhum secret em código
- [ ] OWASP Top 10 checklist completo
- [ ] Backups testados

**Product/Engineering**
- [ ] Signup flow testado (OTP via WhatsApp)
- [ ] Age verification funciona (bloqueia < 18)
- [ ] Gamificação básica funciona (XP, missões)
- [ ] UC World 3D carrega (sem erros)
- [ ] Database backups automáticos
- [ ] Monitoring live (Sentry + Uptime)
- [ ] Error tracking configurado
- [ ] Load testing passou (1k concurrent users)

**Marketing**
- [ ] Social media accounts criadas (Instagram, LinkedIn, Twitter)
- [ ] Email sequences prontas (welcome, features, upgrade)
- [ ] Launch announcement preparado
- [ ] Influencer outreach list criada (20-30 nomes)
- [ ] Analytics tracking (GA4) instalado
- [ ] Support email monitorado (contato@)

**Operations**
- [ ] Incident response plan documentado
- [ ] Rollback procedure testado
- [ ] On-call schedule definido (Week 1 pós-launch)
- [ ] Communication channels prontos (Slack, status page)
- [ ] Support structure ready (email + WhatsApp)

**Stakeholder Sign-Off**
- [ ] Você revisou toda estratégia
- [ ] Time (se tiver) alinhado
- [ ] Go/no-go decision: **GO ✓**

**If ALL checked → LAUNCH 🚀**

---

## APÓS LANÇAMENTO: MÊS 1 ROADMAP

### Week 1 (10-16 Fev)
- Monitor tudo obsessivamente
- Fix bugs críticos imediatamente
- Responder feedback de usuários
- Gather primeiros testimonials

### Week 2 (17-23 Fev)
- Iterar copy/landing pages baseado em dados
- Começar influencer partnerships
- Publicar 2 novos blog posts
- A/B test pricing (se conversão baixa)

### Week 3 (24 Fev - 2 Mar)
- Launch paid ads (teste pequeno, R$ 500-1k)
- Refinar messaging baseado em customer interviews
- Setup referral program básico
- Plan Month 2 features

### Week 4 (3-9 Mar)
- Month 1 review meeting
- Decide: Continue, pivot, or iterate?
- Plan Month 2 strategy
- Consider: Contratar primeiro funcionário (se receita permitir)

---

**VOCÊ ESTÁ PRONTO. EXECUTE O PLANO. 🚀**
