# 23. Monitoring Setup Guide

**Document Version:** 1.0
**Created:** 24 de Janeiro de 2026
**Status:** Ready for implementation

---

## HEALTH ENDPOINTS

All services expose health endpoints for monitoring:

| Service | URL | Deep Check |
|---------|-----|------------|
| Frontend | `https://unofficialcommunities.com.br/api/health` | `?deep=true` |
| API | `https://api.unofficialcommunities.com.br/health` | N/A |
| World | `https://world.unofficialcommunities.com.br/health` | N/A |

### Response Format

**Basic Health Check:**
```json
{
  "status": "ok",
  "service": "unofficial-communities",
  "version": "2.7.0",
  "timestamp": 1769227278255
}
```

**Deep Health Check (Frontend only):**
```json
{
  "status": "ok",
  "service": "unofficial-communities",
  "version": "2.7.0",
  "timestamp": 1769227278255,
  "services": [
    {"name": "uc-api", "status": "ok", "latency": 12},
    {"name": "uc-world", "status": "ok", "latency": 8}
  ]
}
```

### Status Codes
- `200` - All systems healthy
- `503` - Service degraded (one or more downstream services failing)

---

## UPTIMEROBOT SETUP

### Step 1: Create Account
1. Go to https://uptimerobot.com
2. Sign up for free account (50 monitors free)
3. Verify email

### Step 2: Add Monitors

**Monitor 1: Frontend**
- Monitor Type: HTTP(s)
- Friendly Name: UC Frontend
- URL: `https://unofficialcommunities.com.br/api/health`
- Monitoring Interval: 5 minutes
- Monitor Timeout: 30 seconds
- Keyword: `"status":"ok"` (Type: exists)

**Monitor 2: API**
- Monitor Type: HTTP(s)
- Friendly Name: UC API
- URL: `https://api.unofficialcommunities.com.br/health`
- Monitoring Interval: 5 minutes
- Monitor Timeout: 30 seconds
- Keyword: `"status":"ok"` (Type: exists)

**Monitor 3: World (3D)**
- Monitor Type: HTTP(s)
- Friendly Name: UC World
- URL: `https://world.unofficialcommunities.com.br/health`
- Monitoring Interval: 5 minutes
- Monitor Timeout: 30 seconds
- Keyword: `"status":"ok"` (Type: exists)

**Monitor 4: Deep Health Check**
- Monitor Type: HTTP(s)
- Friendly Name: UC Full Stack
- URL: `https://unofficialcommunities.com.br/api/health?deep=true`
- Monitoring Interval: 15 minutes
- Monitor Timeout: 60 seconds
- Keyword: `"status":"ok"` (Type: exists)

### Step 3: Configure Alerts

1. Go to My Settings > Alert Contacts
2. Add email: `alerts@unofficialbrasil.com.br`
3. (Optional) Add Slack/Discord webhook
4. (Optional) Add SMS for critical alerts

### Step 4: Create Status Page (Optional)

1. Go to Status Pages
2. Create new status page
3. Name: "Unofficial Brasil Status"
4. Custom domain: `status.unofficialcommunities.com.br`
5. Add all monitors
6. Customize appearance with brand colors

---

## SENTRY SETUP

### Step 1: Create Sentry Account
1. Go to https://sentry.io
2. Sign up (free tier: 5K errors/month)
3. Create new project: Next.js

### Step 2: Get DSN
1. Go to Settings > Projects > [your project] > Client Keys (DSN)
2. Copy the DSN URL

### Step 3: Configure Environment Variables

Add to your `.env` or Docker environment:

```bash
# Sentry Error Tracking
NEXT_PUBLIC_SENTRY_DSN=https://xxxxx@o123456.ingest.sentry.io/123456

# For source map uploads (optional but recommended)
SENTRY_ORG=unofficialbrasil
SENTRY_PROJECT=unofficial-communities
SENTRY_AUTH_TOKEN=sntrys_xxx...
```

### Step 4: Verify Setup
1. Rebuild the application: `pnpm build`
2. Restart the service
3. Trigger a test error in browser console:
   ```javascript
   throw new Error("Sentry test error");
   ```
4. Check Sentry dashboard for the error

### Sentry Files Created
- `sentry.client.config.ts` - Client-side initialization
- `sentry.server.config.ts` - Server-side initialization
- `sentry.edge.config.ts` - Edge runtime initialization
- `src/instrumentation.ts` - Next.js instrumentation hook
- `src/app/global-error.tsx` - Global error boundary

---

## GOOGLE ANALYTICS 4 SETUP

### Step 1: Create GA4 Property
1. Go to https://analytics.google.com
2. Create new property: "Unofficial Brasil"
3. Create web data stream for `unofficialcommunities.com.br`

### Step 2: Get Measurement ID
1. Go to Admin > Data Streams > Web
2. Copy Measurement ID (format: `G-XXXXXXXXXX`)

### Step 3: Configure Environment Variable
```bash
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

### Step 4: Verify Setup
1. The CookieBanner component handles GA4 initialization
2. GA4 only loads when user consents ("Aceitar Todos")
3. Test with Google Analytics Debugger extension

---

## SSL MONITORING

### SSL Labs Test
Periodically check SSL rating:
- URL: https://www.ssllabs.com/ssltest/
- Enter: `unofficialcommunities.com.br`
- Target: A+ rating

### Certificate Expiry
Let's Encrypt certificates expire every 90 days.
Certbot auto-renews, but monitor:

```bash
# Check certificate expiry
sudo certbot certificates
```

Add UptimeRobot monitor for SSL:
- Monitor Type: SSL Certificate
- URL: `unofficialcommunities.com.br`
- Alert: 14 days before expiry

---

## QUICK REFERENCE

### Health Check URLs (Internal)
```bash
# Basic checks
curl http://localhost:3000/api/health
curl http://localhost:3010/health
curl http://localhost:3005/health

# Deep check (all services)
curl "http://localhost:3000/api/health?deep=true"
```

### Health Check URLs (External)
```bash
# Basic checks
curl https://unofficialcommunities.com.br/api/health
curl https://api.unofficialcommunities.com.br/health
curl https://world.unofficialcommunities.com.br/health
```

### Required Environment Variables
```bash
# Sentry (Error Tracking)
NEXT_PUBLIC_SENTRY_DSN=
SENTRY_ORG=
SENTRY_PROJECT=
SENTRY_AUTH_TOKEN=

# Google Analytics 4
NEXT_PUBLIC_GA_ID=

# UptimeRobot (no env vars needed - configured via web UI)
```

---

## IMPLEMENTATION CHECKLIST

- [x] Security headers added to Nginx
- [x] SSL configured with Let's Encrypt (A+ target)
- [x] Health endpoints created
- [x] Sentry SDK installed and configured
- [x] Global error handler created
- [x] Monitoring guide documented
- [ ] UptimeRobot monitors created (requires web UI)
- [ ] Sentry DSN configured (requires Sentry account)
- [ ] GA4 ID configured (requires GA account)
- [ ] Status page created (optional)

---

*Document created as part of LGPD compliance and infrastructure hardening for Feb 9 launch.*
