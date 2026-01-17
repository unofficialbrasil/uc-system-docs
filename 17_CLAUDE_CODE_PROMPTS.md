# Claude Code Session Prompts

**Last Updated:** 2026-01-16
**Version:** 6.1

---

## Files Registry (29 Files Total)

### Repository Files (10 files)

| Repo | CLAUDE.md | README.md |
|------|-----------|-----------|
| unofficial-communities | `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md` | `/srv/unofficial/prod/app/unofficial-communities/README.md` |
| uc-api | `/srv/unofficial/prod/app/uc-api/CLAUDE.md` | `/srv/unofficial/prod/app/uc-api/README.md` |
| uc-webhooks | `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md` | `/srv/unofficial/prod/app/uc-webhooks/README.md` |
| uc-world | `/srv/unofficial/prod/app/uc-world/CLAUDE.md` | `/srv/unofficial/prod/app/uc-world/README.md` |
| uc-system-docs | `/srv/unofficial/prod/app/uc-system-docs/CLAUDE.md` | `/srv/unofficial/prod/app/uc-system-docs/README.md` |

### System Documentation (19 files)

Location: `/srv/unofficial/prod/app/uc-system-docs/`

| # | File |
|---|------|
| 01 | `01_SYSTEM_CANONICAL_INDEX.md` |
| 02 | `02_PRODUCT_BLUEPRINT.md` |
| 03 | `03_TECHNICAL_ARCHITECTURE.md` |
| 04 | `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md` |
| 05 | `05_USER_STATE_MACHINE.md` |
| 06 | `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` |
| 07 | `07_EVENT_AND_ANALYTICS_SPEC.md` |
| 08 | `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md` |
| 09 | `09_SECURITY_AND_AUTHORIZATION_SPEC.md` |
| 10 | `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md` |
| 11 | `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md` |
| 12 | `12_COMPLIANCE_AND_DATA_PROTECTION.md` |
| 13 | `13_RISK_REGISTER.md` |
| 14 | `14_ASSUMPTION_REGISTER.md` |
| 15 | `15_DECISION_LOG.md` |
| 16 | `16_SESSION_LOG.md` |
| 17 | `17_CLAUDE_CODE_PROMPTS.md` |
| 18 | `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md` |
| 19 | `19_EXECUTION_PLAN.md` |

---

## 1. Open Session Prompt

**Copy and paste at the beginning of EVERY development session:**

```markdown
Hi Claude, let's start a development session. Execute the COMPLETE startup checklist below. Do NOT skip any step.

## PHASE 1: READ ALL FILES (MANDATORY - NO EXCEPTIONS)

You MUST read ALL 29 files to have full context. No exceptions.

### 1.1 Repository CLAUDE.md Files (5 files)
Read each file completely:
- `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-api/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-world/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-system-docs/CLAUDE.md`

### 1.2 Repository README.md Files (5 files)
Read each file completely:
- `/srv/unofficial/prod/app/unofficial-communities/README.md`
- `/srv/unofficial/prod/app/uc-api/README.md`
- `/srv/unofficial/prod/app/uc-webhooks/README.md`
- `/srv/unofficial/prod/app/uc-world/README.md`
- `/srv/unofficial/prod/app/uc-system-docs/README.md`

### 1.3 System Documentation (18 files)
Read ALL files in `/srv/unofficial/prod/app/uc-system-docs/`:
- `01_SYSTEM_CANONICAL_INDEX.md`
- `02_PRODUCT_BLUEPRINT.md`
- `03_TECHNICAL_ARCHITECTURE.md`
- `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md`
- `05_USER_STATE_MACHINE.md`
- `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md`
- `07_EVENT_AND_ANALYTICS_SPEC.md`
- `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md`
- `09_SECURITY_AND_AUTHORIZATION_SPEC.md`
- `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md`
- `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md`
- `12_COMPLIANCE_AND_DATA_PROTECTION.md`
- `13_RISK_REGISTER.md`
- `14_ASSUMPTION_REGISTER.md`
- `15_DECISION_LOG.md`
- `16_SESSION_LOG.md`
- `17_CLAUDE_CODE_PROMPTS.md`
- `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md`
- `19_EXECUTION_PLAN.md`

## PHASE 2: VPS & GITHUB ACCESS CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    VPS & GITHUB ACCESS CHECK                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ SSH Connection Test                                             │"
echo "└─────────────────────────────────────────────────────────────────┘"
ssh -o ConnectTimeout=5 -o BatchMode=yes caue@72.61.134.35 "echo '✓ SSH: Connected to $(hostname)'" 2>/dev/null || echo "✗ SSH: Connection failed"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ GitHub Authentication                                           │"
echo "└─────────────────────────────────────────────────────────────────┘"
ssh -T git@github.com 2>&1 | head -1 || echo "✗ GitHub: Auth failed"
```

## PHASE 3: REPOSITORY STATUS CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                     REPOSITORY STATUS CHECK                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
cd /srv/unofficial/prod/app
for repo in unofficial-communities uc-api uc-webhooks uc-world uc-system-docs; do
  echo "┌─────────────────────────────────────────────────────────────────┐"
  printf "│ %-63s │\n" "$repo"
  echo "├─────────────────────────────────────────────────────────────────┤"
  if [ -d "$repo/.git" ]; then
    cd "$repo"
    # Uncommitted changes
    CHANGES=$(git status --porcelain | wc -l)
    if [ "$CHANGES" -gt 0 ]; then
      printf "│ ⚠️  Uncommitted: %-47s │\n" "$CHANGES files"
    else
      printf "│ ✓  Working tree: %-46s │\n" "Clean"
    fi
    # Unpushed commits
    UNPUSHED=$(git log @{u}..HEAD --oneline 2>/dev/null | wc -l)
    if [ "$UNPUSHED" -gt 0 ]; then
      printf "│ ⚠️  Unpushed: %-49s │\n" "$UNPUSHED commits"
    else
      printf "│ ✓  Remote sync: %-47s │\n" "Up to date"
    fi
    # Last commit
    LAST=$(git log -1 --format='%h %s' | cut -c1-55)
    printf "│ 📝 Last: %-54s │\n" "$LAST"
    cd ..
  else
    printf "│ ✗  Not a git repository                                       │\n"
  fi
  echo "└─────────────────────────────────────────────────────────────────┘"
  echo ""
done
```

## PHASE 4: SYSTEM HEALTH CHECK

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      SYSTEM HEALTH CHECK                         ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ DISK USAGE                                                      │"
echo "├─────────────────────────────────────────────────────────────────┤"
DISK_PCT=$(df -h / | tail -1 | awk '{print $5}' | tr -d '%')
DISK_USED=$(df -h / | tail -1 | awk '{print $3}')
DISK_TOTAL=$(df -h / | tail -1 | awk '{print $2}')
if [ "$DISK_PCT" -gt 90 ]; then
  printf "│ 🔴 CRITICAL: %s/%s (%s%%) - CLEANUP NEEDED                  │\n" "$DISK_USED" "$DISK_TOTAL" "$DISK_PCT"
elif [ "$DISK_PCT" -gt 80 ]; then
  printf "│ 🟡 WARNING: %s/%s (%s%%)                                    │\n" "$DISK_USED" "$DISK_TOTAL" "$DISK_PCT"
else
  printf "│ 🟢 OK: %s/%s (%s%%)                                         │\n" "$DISK_USED" "$DISK_TOTAL" "$DISK_PCT"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ MEMORY                                                          │"
echo "├─────────────────────────────────────────────────────────────────┤"
MEM_USED=$(free -h | grep Mem | awk '{print $3}')
MEM_TOTAL=$(free -h | grep Mem | awk '{print $2}')
MEM_PCT=$(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100)}')
if [ "$MEM_PCT" -gt 90 ]; then
  printf "│ 🔴 RAM: %s/%s (%s%%) - HIGH USAGE                           │\n" "$MEM_USED" "$MEM_TOTAL" "$MEM_PCT"
else
  printf "│ 🟢 RAM: %s/%s (%s%%)                                        │\n" "$MEM_USED" "$MEM_TOTAL" "$MEM_PCT"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ SWAP                                                            │"
echo "├─────────────────────────────────────────────────────────────────┤"
SWAP_USED=$(free -h | grep Swap | awk '{print $3}')
SWAP_TOTAL=$(free -h | grep Swap | awk '{print $2}')
printf "│ 💾 Swap: %s/%s                                               │\n" "$SWAP_USED" "$SWAP_TOTAL"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ SYSTEM LOAD                                                     │"
echo "├─────────────────────────────────────────────────────────────────┤"
LOAD=$(uptime | awk -F'load average:' '{print $2}' | xargs)
UPTIME=$(uptime -p)
printf "│ ⚡ Load: %-54s │\n" "$LOAD"
printf "│ 🕐 %s                                           │\n" "$UPTIME"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 5: DOCKER CONTAINERS STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    DOCKER CONTAINERS STATUS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Container                          │ Status                     │"
echo "├─────────────────────────────────────────────────────────────────┤"
docker ps --format "│ {{.Names}}" | while read line; do
  NAME=$(echo "$line" | cut -d'│' -f2 | xargs)
  STATUS=$(docker ps --filter "name=$NAME" --format "{{.Status}}" | cut -d' ' -f1-2)
  printf "│ %-35s │ %-27s │\n" "$NAME" "$STATUS"
done
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
# Check for unhealthy containers
UNHEALTHY=$(docker ps --filter "status=restarting" --format "{{.Names}}" | wc -l)
EXITED=$(docker ps -a --filter "status=exited" --filter "exited=1" --format "{{.Names}}" | wc -l)
if [ "$UNHEALTHY" -gt 0 ] || [ "$EXITED" -gt 0 ]; then
  echo "⚠️  WARNING: $UNHEALTHY restarting, $EXITED exited with errors"
fi
```

## PHASE 6: SERVICE HEALTH CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                     SERVICE HEALTH CHECK                         ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Service                   │ Port  │ Status                      │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Frontend
FE_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 2>/dev/null || echo "DOWN")
[ "$FE_STATUS" = "200" ] && FE_ICON="🟢" || FE_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "unofficial-communities" "3000" "$FE_ICON" "$FE_STATUS"
# API
API_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3010/health 2>/dev/null || echo "DOWN")
[ "$API_STATUS" = "200" ] && API_ICON="🟢" || API_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "uc-api" "3010" "$API_ICON" "$API_STATUS"
# Webhooks (internal service - check via Docker network)
WH_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://uc-webhooks:3010/health 2>/dev/null || echo "DOWN")
[ "$WH_STATUS" = "200" ] && WH_ICON="🟢" || WH_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "uc-webhooks" "int" "$WH_ICON" "$WH_STATUS"
# UC World
WORLD_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3005 2>/dev/null || echo "DOWN")
[ "$WORLD_STATUS" = "200" ] || [ "$WORLD_STATUS" = "101" ] && WORLD_ICON="🟢" || WORLD_ICON="🟡"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "uc-world" "3005" "$WORLD_ICON" "$WORLD_STATUS"
# Redis
REDIS_STATUS=$(docker exec app-uc-redis-1 redis-cli -a "$(cat /etc/uc/secrets/redis_password 2>/dev/null)" ping 2>/dev/null || echo "DOWN")
[ "$REDIS_STATUS" = "PONG" ] && REDIS_ICON="🟢" || REDIS_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "Redis" "6379" "$REDIS_ICON" "$REDIS_STATUS"
# Prometheus
PROM_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9090/-/healthy 2>/dev/null || echo "DOWN")
[ "$PROM_STATUS" = "200" ] && PROM_ICON="🟢" || PROM_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "Prometheus" "9090" "$PROM_ICON" "$PROM_STATUS"
# Grafana
GRAF_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3100/api/health 2>/dev/null || echo "DOWN")
[ "$GRAF_STATUS" = "200" ] && GRAF_ICON="🟢" || GRAF_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "Grafana" "3100" "$GRAF_ICON" "$GRAF_STATUS"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 7: SITE ANALYTICS & METRICS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    SITE ANALYTICS & METRICS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ USER ACTIVITY (from Prometheus metrics)                         │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Login attempts
LOGINS=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_login_total" | grep -v "^#" | awk '{print $2}' | head -1)
printf "│ 🔐 Total Logins: %-46s │\n" "${LOGINS:-0}"
# Signups
SIGNUPS=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_signup_total" | grep -v "^#" | awk '{print $2}' | head -1)
printf "│ 📝 Total Signups: %-45s │\n" "${SIGNUPS:-0}"
# Logouts
LOGOUTS=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_logout_total" | grep -v "^#" | awk '{print $2}' | head -1)
printf "│ 🚪 Total Logouts: %-45s │\n" "${LOGOUTS:-0}"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ WEBHOOK ACTIVITY                                                │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Social events from webhooks (internal service - check via Docker network)
EVENTS=$(curl -s http://uc-webhooks:3010/metrics 2>/dev/null | grep "social_events_enqueued_total" | grep -v "^#" | awk '{sum+=$2} END {print sum}')
printf "│ 📨 Social Events Processed: %-35s │\n" "${EVENTS:-0}"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ PUSH NOTIFICATIONS                                              │"
echo "├─────────────────────────────────────────────────────────────────┤"
PUSH_SENT=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_push_sent_total" | grep -v "^#" | awk '{print $2}' | head -1)
PUSH_FAILED=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_push_failed_total" | grep -v "^#" | awk '{print $2}' | head -1)
printf "│ ✅ Push Sent: %-49s │\n" "${PUSH_SENT:-0}"
printf "│ ❌ Push Failed: %-47s │\n" "${PUSH_FAILED:-0}"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 8: BACKUP STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                        BACKUP STATUS                             ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ LAST BACKUP                                                     │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Get last backup folder
LAST_BACKUP=$(sudo ls -td /root/uc-backups/2026-*/ 2>/dev/null | head -1)
if [ -n "$LAST_BACKUP" ]; then
  BACKUP_DATE=$(basename "$LAST_BACKUP")
  BACKUP_SIZE=$(sudo du -sh "$LAST_BACKUP" 2>/dev/null | awk '{print $1}')
  FILE_COUNT=$(sudo ls "$LAST_BACKUP" 2>/dev/null | wc -l)
  printf "│ 📅 Date: %-54s │\n" "$BACKUP_DATE"
  printf "│ 📦 Size: %-54s │\n" "$BACKUP_SIZE"
  printf "│ 📁 Files: %-53s │\n" "$FILE_COUNT files"
else
  printf "│ ⚠️  No backups found!                                          │\n"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BACKUP SUCCESS STATUS                                           │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Check if last backup was successful
LAST_LOG=$(sudo tail -5 /root/uc-backups/logs/cron.log 2>/dev/null | grep -i "completed successfully")
if [ -n "$LAST_LOG" ]; then
  printf "│ 🟢 Last backup: SUCCESSFUL                                     │\n"
else
  LAST_ERROR=$(sudo tail -10 /root/uc-backups/logs/cron.log 2>/dev/null | grep -iE "error|failed" | tail -1)
  if [ -n "$LAST_ERROR" ]; then
    printf "│ 🔴 Last backup: FAILED                                         │\n"
    printf "│    %s │\n" "$(echo "$LAST_ERROR" | cut -c1-60)"
  else
    printf "│ 🟡 Last backup: Status unknown (check logs)                    │\n"
  fi
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BACKUP SCHEDULE                                                 │"
echo "├─────────────────────────────────────────────────────────────────┤"
printf "│ 🕐 Schedule: Daily at 03:00 AM                                  │\n"
# Calculate next backup
CURRENT_HOUR=$(date +%H)
if [ "$CURRENT_HOUR" -lt 3 ]; then
  NEXT_BACKUP="Today at 03:00 AM"
else
  NEXT_BACKUP="Tomorrow at 03:00 AM"
fi
printf "│ ⏭️  Next backup: %-46s │\n" "$NEXT_BACKUP"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BACKUP CONTENTS (Latest)                                        │"
echo "├─────────────────────────────────────────────────────────────────┤"
if [ -n "$LAST_BACKUP" ]; then
  sudo ls -lh "$LAST_BACKUP" 2>/dev/null | grep -v "^total" | grep -v "^d" | while read line; do
    FILE=$(echo "$line" | awk '{print $9}')
    SIZE=$(echo "$line" | awk '{print $5}')
    printf "│ 💾 %-45s %s │\n" "$FILE" "$SIZE"
  done
fi
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 9: DATABASE HEALTH & INTEGRITY

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  DATABASE HEALTH & INTEGRITY                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Initialize status tracking
DB_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ MySQL Container & Connectivity                                  │"
echo "├─────────────────────────────────────────────────────────────────┤"

# Check MySQL container
MYSQL_CONTAINER=$(docker ps --filter "name=mysql" --filter "status=running" --format "{{.Names}}" | head -1)
if [ -n "$MYSQL_CONTAINER" ]; then
  printf "│ 🟢 Container: %-49s │\n" "$MYSQL_CONTAINER running"

  # Test connectivity (read credentials safely)
  DB_URL=$(grep -E "^DATABASE_URL=" /etc/uc/secrets/prod/uc-api.env 2>/dev/null | cut -d'=' -f2- | tr -d '"')
  if [ -n "$DB_URL" ]; then
    # Extract host/port from URL (mysql://user:pass@host:port/db)
    DB_HOST=$(echo "$DB_URL" | sed -E 's|.*@([^:]+):([0-9]+)/.*|\1|')
    DB_PORT=$(echo "$DB_URL" | sed -E 's|.*@([^:]+):([0-9]+)/.*|\2|')

    # Test port connectivity
    if timeout 3 bash -c "echo >/dev/tcp/$DB_HOST/$DB_PORT" 2>/dev/null; then
      printf "│ 🟢 Port %s: %-51s │\n" "$DB_PORT" "Accepting connections"
    else
      printf "│ 🔴 Port %s: %-51s │\n" "$DB_PORT" "Connection refused"
      DB_STATUS="RED"
    fi

    # Test actual query via docker
    PING_RESULT=$(docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null || echo "FAILED")
    if echo "$PING_RESULT" | grep -q "alive"; then
      printf "│ 🟢 MySQL ping: %-48s │\n" "Server alive"
    else
      printf "│ 🔴 MySQL ping: %-48s │\n" "FAILED"
      DB_STATUS="RED"
    fi
  else
    printf "│ 🟡 DATABASE_URL: %-46s │\n" "Not found (CHECK REQUIRED)"
    DB_STATUS="YELLOW"
  fi
else
  printf "│ 🔴 MySQL container: %-43s │\n" "NOT RUNNING"
  DB_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Prisma Migration Status                                         │"
echo "├─────────────────────────────────────────────────────────────────┤"
cd /srv/unofficial/prod/app/uc-api
if [ -f "prisma/schema.prisma" ]; then
  # Check for pending migrations
  MIGRATION_STATUS=$(npx prisma migrate status 2>&1 || echo "CHECK_REQUIRED")
  if echo "$MIGRATION_STATUS" | grep -q "Database schema is up to date"; then
    printf "│ 🟢 Schema: %-52s │\n" "Up to date"
  elif echo "$MIGRATION_STATUS" | grep -q "have not yet been applied"; then
    PENDING=$(echo "$MIGRATION_STATUS" | grep -c "not yet been applied" || echo "?")
    printf "│ 🟡 Pending migrations: %-40s │\n" "$PENDING"
    [ "$DB_STATUS" != "RED" ] && DB_STATUS="YELLOW"
  elif echo "$MIGRATION_STATUS" | grep -q "drift"; then
    printf "│ 🔴 Schema drift: %-46s │\n" "DETECTED"
    DB_STATUS="RED"
  else
    printf "│ 🟡 Migration status: %-42s │\n" "CHECK REQUIRED"
    [ "$DB_STATUS" != "RED" ] && DB_STATUS="YELLOW"
  fi
else
  printf "│ 🟡 Prisma schema: %-45s │\n" "Not found"
fi
cd /srv/unofficial/prod/app
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Core Tables Integrity Probe                                     │"
echo "├─────────────────────────────────────────────────────────────────┤"
if [ -n "$MYSQL_CONTAINER" ]; then
  # Quick row count checks on critical tables (non-zero)
  for TABLE in identity community mission_template; do
    COUNT=$(docker exec "$MYSQL_CONTAINER" mysql -N -e "SELECT COUNT(*) FROM $TABLE;" uc 2>/dev/null || echo "ERR")
    if [ "$COUNT" = "ERR" ]; then
      printf "│ 🟡 %-20s %-38s │\n" "$TABLE:" "Query failed"
    elif [ "$COUNT" -eq 0 ] 2>/dev/null; then
      printf "│ 🟡 %-20s %-38s │\n" "$TABLE:" "0 rows (expected data?)"
    else
      printf "│ 🟢 %-20s %-38s │\n" "$TABLE:" "$COUNT rows"
    fi
  done
else
  printf "│ 🔴 Cannot probe: %-46s │\n" "MySQL not running"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ DATABASE STATUS: "
case "$DB_STATUS" in
  GREEN)  printf "🟢 %-48s │\n" "HEALTHY" ;;
  YELLOW) printf "🟡 %-48s │\n" "DEGRADED - Review warnings" ;;
  RED)    printf "🔴 %-48s │\n" "CRITICAL - Immediate attention" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 10: ASYNC / QUEUE VERIFICATION

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                   ASYNC / QUEUE VERIFICATION                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

QUEUE_STATUS="GREEN"
REDIS_PASS=$(cat /etc/uc/secrets/redis_password 2>/dev/null)

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Redis Connectivity                                              │"
echo "├─────────────────────────────────────────────────────────────────┤"
REDIS_PING=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ping 2>/dev/null || echo "FAILED")
if [ "$REDIS_PING" = "PONG" ]; then
  printf "│ 🟢 Redis: %-53s │\n" "Connected"
  REDIS_INFO=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" info memory 2>/dev/null | grep "used_memory_human" | cut -d: -f2 | tr -d '\r')
  printf "│ 💾 Memory: %-52s │\n" "${REDIS_INFO:-unknown}"
else
  printf "│ 🔴 Redis: %-53s │\n" "CONNECTION FAILED"
  QUEUE_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BullMQ Queue Status (social-events-queue)                       │"
echo "├─────────────────────────────────────────────────────────────────┤"
if [ "$REDIS_PING" = "PONG" ]; then
  # Check queue depths using BullMQ key patterns
  WAITING=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" LLEN "bull:social-events-queue:wait" 2>/dev/null || echo "0")
  ACTIVE=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" LLEN "bull:social-events-queue:active" 2>/dev/null || echo "0")
  DELAYED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:delayed" 2>/dev/null || echo "0")
  FAILED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:failed" 2>/dev/null || echo "0")

  printf "│ ⏳ Waiting: %-51s │\n" "${WAITING:-0} jobs"
  printf "│ ⚡ Active: %-52s │\n" "${ACTIVE:-0} jobs"
  printf "│ 🕐 Delayed: %-51s │\n" "${DELAYED:-0} jobs"

  # Failed jobs warning
  if [ "${FAILED:-0}" -gt 0 ]; then
    printf "│ 🔴 Failed: %-52s │\n" "$FAILED jobs (ATTENTION)"
    [ "$QUEUE_STATUS" != "RED" ] && QUEUE_STATUS="YELLOW"
  else
    printf "│ 🟢 Failed: %-52s │\n" "0 jobs"
  fi

  # Check for stuck jobs (waiting > 100)
  if [ "${WAITING:-0}" -gt 100 ]; then
    printf "│ 🟡 WARNING: %-51s │\n" "Queue backlog detected"
    [ "$QUEUE_STATUS" != "RED" ] && QUEUE_STATUS="YELLOW"
  fi
else
  printf "│ 🔴 Cannot check: %-46s │\n" "Redis unavailable"
  QUEUE_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ QUEUE STATUS: "
case "$QUEUE_STATUS" in
  GREEN)  printf "🟢 %-50s │\n" "HEALTHY" ;;
  YELLOW) printf "🟡 %-50s │\n" "DEGRADED - Review backlog/failures" ;;
  RED)    printf "🔴 %-50s │\n" "CRITICAL" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 11: E2E FUNCTIONAL SMOKE TESTS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  E2E FUNCTIONAL SMOKE TESTS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

E2E_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ API Auth Flow (Protected Endpoint Test)                         │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Test that protected endpoint returns 401 without auth (expected behavior)
AUTH_TEST=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3010/v37/me 2>/dev/null || echo "ERR")
if [ "$AUTH_TEST" = "401" ]; then
  printf "│ 🟢 Auth guard: %-48s │\n" "Working (401 on /v37/me)"
elif [ "$AUTH_TEST" = "ERR" ]; then
  printf "│ 🔴 Auth test: %-49s │\n" "API unreachable"
  E2E_STATUS="RED"
else
  printf "│ 🟡 Auth test: %-49s │\n" "Unexpected: HTTP $AUTH_TEST"
  [ "$E2E_STATUS" != "RED" ] && E2E_STATUS="YELLOW"
fi

# Test health endpoint returns proper JSON
HEALTH_BODY=$(curl -s http://localhost:3010/health 2>/dev/null)
if echo "$HEALTH_BODY" | grep -q "ok\|healthy\|status" 2>/dev/null; then
  printf "│ 🟢 Health endpoint: %-43s │\n" "Returns valid response"
else
  printf "│ 🟡 Health endpoint: %-43s │\n" "Unexpected response format"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Webhooks Service (Internal - via Docker network)                │"
echo "├─────────────────────────────────────────────────────────────────┤"
WH_HEALTH=$(curl -s -o /dev/null -w "%{http_code}" http://uc-webhooks:3010/health 2>/dev/null || echo "ERR")
if [ "$WH_HEALTH" = "200" ]; then
  printf "│ 🟢 Health: %-52s │\n" "OK"

  # Check metrics endpoint returns valid Prometheus format
  WH_METRICS=$(curl -s http://uc-webhooks:3010/metrics 2>/dev/null | head -1)
  if echo "$WH_METRICS" | grep -q "^#\|^[a-z]" 2>/dev/null; then
    printf "│ 🟢 Metrics: %-51s │\n" "Prometheus format OK"
  else
    printf "│ 🟡 Metrics: %-51s │\n" "CHECK REQUIRED"
  fi
else
  printf "│ 🔴 Webhooks: %-50s │\n" "Service down (HTTP $WH_HEALTH)"
  E2E_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ UC World (Colyseus Server Check)                                │"
echo "├─────────────────────────────────────────────────────────────────┤"
WORLD_HTTP=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3005 2>/dev/null || echo "ERR")
if [ "$WORLD_HTTP" = "200" ] || [ "$WORLD_HTTP" = "101" ] || [ "$WORLD_HTTP" = "426" ]; then
  printf "│ 🟢 HTTP: %-54s │\n" "Responding (HTTP $WORLD_HTTP)"

  # Attempt WebSocket upgrade check (connection test)
  WS_CHECK=$(curl -s -o /dev/null -w "%{http_code}" \
    -H "Connection: Upgrade" \
    -H "Upgrade: websocket" \
    -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
    -H "Sec-WebSocket-Version: 13" \
    http://localhost:3005 2>/dev/null || echo "ERR")
  if [ "$WS_CHECK" = "101" ] || [ "$WS_CHECK" = "426" ]; then
    printf "│ 🟢 WebSocket: %-49s │\n" "Upgrade path available"
  else
    printf "│ 🟡 WebSocket: %-49s │\n" "CHECK REQUIRED (HTTP $WS_CHECK)"
  fi
else
  printf "│ 🟡 UC World: %-50s │\n" "Not responding (expected if disabled)"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Frontend Render Check                                           │"
echo "├─────────────────────────────────────────────────────────────────┤"
FE_BODY=$(curl -s http://localhost:3000 2>/dev/null | head -50)
if echo "$FE_BODY" | grep -q "<html\|<!DOCTYPE\|__NEXT" 2>/dev/null; then
  printf "│ 🟢 Frontend: %-50s │\n" "Renders HTML (Next.js detected)"
else
  FE_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 2>/dev/null)
  printf "│ 🔴 Frontend: %-50s │\n" "Not rendering (HTTP $FE_STATUS)"
  E2E_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ E2E STATUS: "
case "$E2E_STATUS" in
  GREEN)  printf "🟢 %-52s │\n" "ALL SMOKE TESTS PASSED" ;;
  YELLOW) printf "🟡 %-52s │\n" "PARTIAL - Review warnings" ;;
  RED)    printf "🔴 %-52s │\n" "FAILURES DETECTED" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 12: RUNTIME VS GIT CONSISTENCY

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  RUNTIME VS GIT CONSISTENCY                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Container Image vs Git HEAD Comparison                          │"
echo "├─────────────────────────────────────────────────────────────────┤"

cd /srv/unofficial/prod/app

# Check each service
for SERVICE in unofficial-communities uc-api uc-webhooks; do
  CONTAINER_NAME=$(docker ps --filter "name=$SERVICE" --format "{{.Names}}" | head -1)

  if [ -n "$CONTAINER_NAME" ]; then
    # Try to get image label (if set during build)
    IMAGE_SHA=$(docker inspect "$CONTAINER_NAME" --format '{{index .Config.Labels "git.commit"}}' 2>/dev/null || echo "")

    # Get current repo HEAD
    REPO_HEAD=$(git -C "$SERVICE" rev-parse --short HEAD 2>/dev/null || echo "unknown")

    if [ -n "$IMAGE_SHA" ] && [ "$IMAGE_SHA" != "" ]; then
      if [ "$IMAGE_SHA" = "$REPO_HEAD" ]; then
        printf "│ 🟢 %-22s MATCH: %-26s │\n" "$SERVICE" "$REPO_HEAD"
      else
        printf "│ 🟡 %-22s MISMATCH                       │\n" "$SERVICE"
        printf "│    Image: %-15s Repo: %-21s │\n" "$IMAGE_SHA" "$REPO_HEAD"
      fi
    else
      # No label, check by image creation time vs last commit
      IMAGE_CREATED=$(docker inspect "$CONTAINER_NAME" --format '{{.Created}}' 2>/dev/null | cut -d'T' -f1)
      LAST_COMMIT=$(git -C "$SERVICE" log -1 --format='%ci' 2>/dev/null | cut -d' ' -f1)
      printf "│ 🟡 %-22s UNKNOWN (no git label)           │\n" "$SERVICE"
      printf "│    Image: %-15s Commit: %-19s │\n" "${IMAGE_CREATED:-?}" "${LAST_COMMIT:-?}"
    fi
  else
    printf "│ ⚪ %-22s Container not found              │\n" "$SERVICE"
  fi
done

cd /srv/unofficial/prod/app
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Docker Compose File Hash                                        │"
echo "├─────────────────────────────────────────────────────────────────┤"
if [ -f "docker-compose.prod.yml" ]; then
  COMPOSE_HASH=$(md5sum docker-compose.prod.yml | cut -c1-8)
  COMPOSE_DATE=$(stat -c %y docker-compose.prod.yml | cut -d' ' -f1)
  printf "│ 📄 docker-compose.prod.yml                                     │\n"
  printf "│    Hash: %-15s Modified: %-23s │\n" "$COMPOSE_HASH" "$COMPOSE_DATE"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 13: SECURITY DRIFT INDICATORS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                   SECURITY DRIFT INDICATORS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

SEC_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Auth Failure Metrics                                            │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Check for auth failure metrics from Prometheus
AUTH_FAILURES=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_failed_total" | grep -v "^#" | awk '{print $2}' | head -1)
if [ -n "$AUTH_FAILURES" ] && [ "$AUTH_FAILURES" != "0" ]; then
  # Check if high (arbitrary threshold: 100)
  if [ "${AUTH_FAILURES%.*}" -gt 100 ] 2>/dev/null; then
    printf "│ 🟡 Failed auth attempts: %-38s │\n" "$AUTH_FAILURES (elevated)"
    [ "$SEC_STATUS" != "RED" ] && SEC_STATUS="YELLOW"
  else
    printf "│ 🟢 Failed auth attempts: %-38s │\n" "${AUTH_FAILURES:-0}"
  fi
else
  printf "│ 🟢 Failed auth attempts: %-38s │\n" "${AUTH_FAILURES:-0}"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Environment Variables Check                                     │"
echo "├─────────────────────────────────────────────────────────────────┤"
# List of required env vars (check existence only, never print values)
REQUIRED_VARS="DATABASE_URL REDIS_URL JWT_SECRET PSEUDONYM_SECRET"
MISSING_VARS=0

if [ -f "/etc/uc/secrets/prod/uc-api.env" ]; then
  for VAR in $REQUIRED_VARS; do
    if grep -q "^$VAR=" /etc/uc/secrets/prod/uc-api.env 2>/dev/null; then
      : # exists
    else
      MISSING_VARS=$((MISSING_VARS + 1))
      printf "│ 🔴 Missing: %-51s │\n" "$VAR"
      SEC_STATUS="RED"
    fi
  done

  if [ "$MISSING_VARS" -eq 0 ]; then
    printf "│ 🟢 Required env vars: %-41s │\n" "All present (4/4)"
  fi
else
  printf "│ 🟡 Env file: %-50s │\n" "Not found (CHECK REQUIRED)"
  SEC_STATUS="YELLOW"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Secrets in Git Check (Safe Patterns)                            │"
echo "├─────────────────────────────────────────────────────────────────┤"
cd /srv/unofficial/prod/app
SECRETS_FOUND=0

for repo in unofficial-communities uc-api uc-webhooks uc-world; do
  if [ -d "$repo/.git" ]; then
    # Check staged/modified files for common secret patterns (safe grep)
    STAGED=$(git -C "$repo" diff --cached --name-only 2>/dev/null)
    MODIFIED=$(git -C "$repo" diff --name-only 2>/dev/null)

    for file in $STAGED $MODIFIED; do
      if [ -f "$repo/$file" ]; then
        # Check for obvious secret patterns (password=, api_key=, etc.)
        if grep -qiE "(password|secret|api_key|private_key)\s*[:=]" "$repo/$file" 2>/dev/null; then
          printf "│ 🔴 Potential secret: %-42s │\n" "$repo/$file"
          SECRETS_FOUND=$((SECRETS_FOUND + 1))
          SEC_STATUS="RED"
        fi
      fi
    done
  fi
done

if [ "$SECRETS_FOUND" -eq 0 ]; then
  printf "│ 🟢 No secrets detected in staged/modified files              │\n"
fi
cd /srv/unofficial/prod/app
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ SECURITY STATUS: "
case "$SEC_STATUS" in
  GREEN)  printf "🟢 %-46s │\n" "OK" ;;
  YELLOW) printf "🟡 %-46s │\n" "REVIEW REQUIRED" ;;
  RED)    printf "🔴 %-46s │\n" "ISSUES DETECTED" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 14: OBSERVABILITY TREND CHECKS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  OBSERVABILITY TREND CHECKS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

OBS_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Container Restart Counts (Last 24h)                             │"
echo "├─────────────────────────────────────────────────────────────────┤"
TOTAL_RESTARTS=0

for CONTAINER in $(docker ps --format "{{.Names}}"); do
  RESTARTS=$(docker inspect "$CONTAINER" --format '{{.RestartCount}}' 2>/dev/null || echo "0")
  if [ "${RESTARTS:-0}" -gt 0 ]; then
    printf "│ 🟡 %-30s Restarts: %-14s │\n" "$CONTAINER" "$RESTARTS"
    TOTAL_RESTARTS=$((TOTAL_RESTARTS + RESTARTS))
  fi
done

if [ "$TOTAL_RESTARTS" -eq 0 ]; then
  printf "│ 🟢 All containers: %-44s │\n" "0 restarts"
else
  [ "$OBS_STATUS" != "RED" ] && OBS_STATUS="YELLOW"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Latency Probes (curl timing)                                    │"
echo "├─────────────────────────────────────────────────────────────────┤"

# Frontend latency
FE_TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:3000 2>/dev/null || echo "ERR")
if [ "$FE_TIME" != "ERR" ]; then
  FE_MS=$(echo "$FE_TIME * 1000" | bc 2>/dev/null | cut -d'.' -f1 || echo "?")
  if [ "${FE_MS:-999}" -gt 2000 ] 2>/dev/null; then
    printf "│ 🟡 Frontend: %-50s │\n" "${FE_MS}ms (slow)"
    [ "$OBS_STATUS" != "RED" ] && OBS_STATUS="YELLOW"
  else
    printf "│ 🟢 Frontend: %-50s │\n" "${FE_MS}ms"
  fi
else
  printf "│ 🔴 Frontend: %-50s │\n" "Unreachable"
  OBS_STATUS="RED"
fi

# API latency
API_TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:3010/health 2>/dev/null || echo "ERR")
if [ "$API_TIME" != "ERR" ]; then
  API_MS=$(echo "$API_TIME * 1000" | bc 2>/dev/null | cut -d'.' -f1 || echo "?")
  if [ "${API_MS:-999}" -gt 500 ] 2>/dev/null; then
    printf "│ 🟡 API: %-55s │\n" "${API_MS}ms (slow)"
    [ "$OBS_STATUS" != "RED" ] && OBS_STATUS="YELLOW"
  else
    printf "│ 🟢 API: %-55s │\n" "${API_MS}ms"
  fi
else
  printf "│ 🔴 API: %-55s │\n" "Unreachable"
  OBS_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Prometheus Error Rate (if available)                            │"
echo "├─────────────────────────────────────────────────────────────────┤"
# Try to get HTTP 5xx count from metrics
HTTP_5XX=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "http_requests_total.*status=\"5" | awk '{sum+=$2} END {print sum}')
if [ -n "$HTTP_5XX" ] && [ "${HTTP_5XX%.*}" -gt 0 ] 2>/dev/null; then
  printf "│ 🟡 HTTP 5xx errors: %-43s │\n" "$HTTP_5XX total"
  [ "$OBS_STATUS" != "RED" ] && OBS_STATUS="YELLOW"
else
  printf "│ 🟢 HTTP 5xx errors: %-43s │\n" "${HTTP_5XX:-0} (or metric N/A)"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ OBSERVABILITY STATUS: "
case "$OBS_STATUS" in
  GREEN)  printf "🟢 %-42s │\n" "HEALTHY" ;;
  YELLOW) printf "🟡 %-42s │\n" "REVIEW TRENDS" ;;
  RED)    printf "🔴 %-42s │\n" "ISSUES DETECTED" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 15: SESSION START SUMMARY

After completing ALL phases, provide this summary:

```
╔══════════════════════════════════════════════════════════════════╗
║                    SESSION START SUMMARY                         ║
╠══════════════════════════════════════════════════════════════════╣
║ Date: YYYY-MM-DD HH:MM                                           ║
║                                                                  ║
║ FILES LOADED:                                                    ║
║   ✓ Repository files: 10/10                                      ║
║   ✓ System files: 18/18                                          ║
║   ✓ Total context: 29 files                                      ║
║                                                                  ║
║ INFRASTRUCTURE:                                                  ║
║   • VPS: [🟢 Connected / 🔴 Disconnected]                        ║
║   • GitHub: [🟢 Authenticated / 🔴 Failed]                       ║
║   • Disk: [XX%] Memory: [XX%] Load: [X.XX]                       ║
║   • Containers: [X running / X restarting]                       ║
║                                                                  ║
║ SERVICES:                                                        ║
║   • Frontend (3000): [🟢/🔴]  API (3010): [🟢/🔴]                ║
║   • Webhooks (4101): [🟢/🔴]  World (3005): [🟢/🟡]              ║
║   • Redis: [🟢/🔴]  Prometheus: [🟢/🔴]  Grafana: [🟢/🔴]        ║
║                                                                  ║
║ DATA LAYER:                                                      ║
║   • Database: [🟢 HEALTHY / 🟡 DEGRADED / 🔴 CRITICAL]           ║
║   • Migrations: [Up to date / X pending / Drift detected]        ║
║   • Queue: [🟢 HEALTHY / 🟡 BACKLOG / 🔴 DOWN]                   ║
║   • Failed jobs: [X]                                             ║
║                                                                  ║
║ SMOKE TESTS:                                                     ║
║   • Auth guard: [🟢/🔴]  Webhooks: [🟢/🔴]  Frontend: [🟢/🔴]    ║
║                                                                  ║
║ SECURITY:                                                        ║
║   • Failed auth attempts: [X]                                    ║
║   • Env vars: [All present / X missing]                          ║
║   • Secrets in git: [None / X files flagged]                     ║
║                                                                  ║
║ OBSERVABILITY:                                                   ║
║   • Container restarts: [X total]                                ║
║   • Latency: Frontend [Xms] API [Xms]                            ║
║                                                                  ║
║ BACKUP:                                                          ║
║   • Last: [DATE] - [🟢 SUCCESS / 🔴 FAILED]                      ║
║   • Next: [Today/Tomorrow at 03:00 AM]                           ║
║                                                                  ║
║ GIT:                                                             ║
║   • Uncommitted: [X repos]  Unpushed: [X repos]                  ║
║   • Runtime/Git: [MATCH / MISMATCH / UNKNOWN]                    ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║                        GO / NO-GO GATE                           ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║ CRITERIA:                                                        ║
║   [✓/✗] SSH to VPS accessible                                    ║
║   [✓/✗] GitHub authentication working                            ║
║   [✓/✗] Database connectivity                                    ║
║   [✓/✗] API /health returns 200                                  ║
║   [✓/✗] Redis connectivity                                       ║
║   [✓/✗] No critical security issues                              ║
║                                                                  ║
║ ┌────────────────────────────────────────────────────────────┐   ║
║ │  DECISION: [🟢 GO] / [🟡 CAUTION] / [🔴 NO-GO]             │   ║
║ └────────────────────────────────────────────────────────────┘   ║
║                                                                  ║
║ [If CAUTION/NO-GO, list required actions before proceeding]      ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝

[GO] Session ready. What would you like to work on today?
[CAUTION] Session ready with warnings. Address highlighted issues.
[NO-GO] Critical issues must be resolved before development work.
```
```

---

## 2. Close Session Prompt

**Copy and paste before ending ANY session:**

```markdown
Hi Claude, I need to close this session. Execute the COMPLETE shutdown checklist. This is CRITICAL - every file must be updated.

## PHASE 1: PRE-CLOSE SYSTEM CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    PRE-CLOSE SYSTEM CHECK                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ SYSTEM RESOURCES                                                │"
echo "├─────────────────────────────────────────────────────────────────┤"
DISK_PCT=$(df -h / | tail -1 | awk '{print $5}')
MEM_PCT=$(free | grep Mem | awk '{printf("%.0f%%", $3/$2 * 100)}')
LOAD=$(uptime | awk -F'load average:' '{print $2}' | xargs | cut -d',' -f1)
printf "│ 💾 Disk: %-15s 🧠 Memory: %-15s ⚡ Load: %-8s │\n" "$DISK_PCT" "$MEM_PCT" "$LOAD"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ SERVICE STATUS                                                  │"
echo "├─────────────────────────────────────────────────────────────────┤"
FE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 2>/dev/null)
API=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3010/health 2>/dev/null)
WH=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:4101/health 2>/dev/null)
WORLD=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3005 2>/dev/null)
[ "$FE" = "200" ] && FE_S="🟢" || FE_S="🔴"
[ "$API" = "200" ] && API_S="🟢" || API_S="🔴"
[ "$WH" = "200" ] && WH_S="🟢" || WH_S="🔴"
[ "$WORLD" = "200" ] || [ "$WORLD" = "101" ] && WORLD_S="🟢" || WORLD_S="🟡"
printf "│ %s Frontend  %s API  %s Webhooks  %s World                      │\n" "$FE_S" "$API_S" "$WH_S" "$WORLD_S"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 2: UPDATE ALL REPOSITORY FILES (MANDATORY)

For EACH of the 10 repository files below, either:
- **UPDATE** with changes made during this session, OR
- **ADD** a review stamp if no changes: `<!-- Last Reviewed: YYYY-MM-DD - No updates needed -->`

### CLAUDE.md Files (5 files)
| File | Action |
|------|--------|
| `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md` | Update "Current Status" |
| `/srv/unofficial/prod/app/uc-api/CLAUDE.md` | Update if API changed |
| `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md` | Update if webhooks changed |
| `/srv/unofficial/prod/app/uc-world/CLAUDE.md` | Update if World changed |
| `/srv/unofficial/prod/app/uc-system-docs/CLAUDE.md` | Update if docs structure changed |

### README.md Files (5 files)
| File | Action |
|------|--------|
| `/srv/unofficial/prod/app/unofficial-communities/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-api/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-webhooks/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-world/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-system-docs/README.md` | Update if docs structure changed |

## PHASE 3: UPDATE ALL SYSTEM FILES (MANDATORY)

For EACH of the 18 system files, either:
- **UPDATE** the content if relevant changes were made, OR
- **ADD** a review stamp at the end: `<!-- Last Reviewed: YYYY-MM-DD - No updates needed -->`

Location: `/srv/unofficial/prod/app/uc-system-docs/`

| # | File | Update if... |
|---|------|--------------|
| 01 | `01_SYSTEM_CANONICAL_INDEX.md` | Document structure changed |
| 02 | `02_PRODUCT_BLUEPRINT.md` | Vision/principles changed |
| 03 | `03_TECHNICAL_ARCHITECTURE.md` | Architecture changed |
| 04 | `04_SYSTEM_ALGORITHM_AND_LIFECYCLE_SPEC.md` | Algorithms changed |
| 05 | `05_USER_STATE_MACHINE.md` | User flows changed |
| 06 | `06_DATA_ARCHITECTURE_AND_LIFECYCLE.md` | Data handling changed |
| 07 | `07_EVENT_AND_ANALYTICS_SPEC.md` | Events changed |
| 08 | `08_BACKGROUND_JOBS_AND_ASYNC_PROCESSING.md` | Jobs changed |
| 09 | `09_SECURITY_AND_AUTHORIZATION_SPEC.md` | Security changed |
| 10 | `10_DEPLOYMENT_AND_ROLLBACK_RUNBOOK.md` | Deployment changed |
| 11 | `11_ENVIRONMENT_AND_CONFIGURATION_REGISTRY.md` | Config changed |
| 12 | `12_COMPLIANCE_AND_DATA_PROTECTION.md` | Compliance changed |
| 13 | `13_RISK_REGISTER.md` | New risks identified |
| 14 | `14_ASSUMPTION_REGISTER.md` | Assumptions changed |
| 15 | `15_DECISION_LOG.md` | Decisions made |
| 16 | `16_SESSION_LOG.md` | **ALWAYS** add session entry |
| 17 | `17_CLAUDE_CODE_PROMPTS.md` | Procedures changed |
| 18 | `18_BEHAVIORAL_SCIENCE_FRAMEWORK.md` | Research updated |
| 19 | `19_EXECUTION_PLAN.md` | Execution steps changed |

### Session Log Entry Template (16_SESSION_LOG.md)

ALWAYS add:
```markdown
### Session: YYYY-MM-DD

**Duration:** ~X hours
**Focus:** [Main topic]

**Accomplished:**
1. [Task 1]
2. [Task 2]

**Files Modified:**
- [file1]
- [file2]

**Metrics at Close:**
- Disk: XX%
- Memory: XX%
- Services: X/7 healthy

**Next Priorities:**
1. [Priority 1]
2. [Priority 2]
```

## PHASE 4: VPS & GITHUB VERIFICATION

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  VPS & GITHUB VERIFICATION                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Connection Check                                                │"
echo "├─────────────────────────────────────────────────────────────────┤"
VPS=$(ssh -o ConnectTimeout=5 -o BatchMode=yes caue@72.61.134.35 "echo 'OK'" 2>/dev/null || echo "FAILED")
GH=$(ssh -T git@github.com 2>&1 | grep -q "successfully" && echo "OK" || echo "Check required")
printf "│ 🖥️  VPS SSH: %-51s │\n" "$VPS"
printf "│ 🐙 GitHub: %-52s │\n" "$GH"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 5: GIT STATUS & COMMIT

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      GIT STATUS CHECK                            ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
cd /srv/unofficial/prod/app
for repo in unofficial-communities uc-api uc-webhooks uc-world uc-system-docs; do
  echo "┌─────────────────────────────────────────────────────────────────┐"
  printf "│ 📁 %-60s │\n" "$repo"
  echo "├─────────────────────────────────────────────────────────────────┤"
  cd "$repo"
  CHANGES=$(git status --porcelain | wc -l)
  UNPUSHED=$(git log @{u}..HEAD --oneline 2>/dev/null | wc -l)
  if [ "$CHANGES" -gt 0 ]; then
    printf "│ ⚠️  Uncommitted: %-47s │\n" "$CHANGES files"
    git status --porcelain | head -5 | while read line; do
      printf "│    %s%-56s │\n" "" "$line"
    done
  else
    printf "│ ✓  Working tree: Clean                                        │\n"
  fi
  if [ "$UNPUSHED" -gt 0 ]; then
    printf "│ ⚠️  Unpushed: %-49s │\n" "$UNPUSHED commits"
  else
    printf "│ ✓  Remote: Synced                                             │\n"
  fi
  echo "└─────────────────────────────────────────────────────────────────┘"
  echo ""
  cd ..
done
```

### Commit & Push Each Repo with Changes
```bash
cd /srv/unofficial/prod/app/[REPO_NAME]
git add -A
git commit -m "[type]: [description]

Co-Authored-By: Claude <noreply@anthropic.com>"
git push origin main
```

Commit types: `feat`, `fix`, `docs`, `refactor`, `chore`

### Verify All Pushed
```bash
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ FINAL GIT SYNC STATUS                                           │"
echo "├─────────────────────────────────────────────────────────────────┤"
cd /srv/unofficial/prod/app
for repo in unofficial-communities uc-api uc-webhooks uc-world uc-system-docs; do
  AHEAD=$(git -C "$repo" rev-list @{u}..HEAD --count 2>/dev/null || echo "0")
  if [ "$AHEAD" -eq 0 ]; then
    printf "│ ✓  %-40s Synced              │\n" "$repo"
  else
    printf "│ ⚠️  %-40s %s unpushed          │\n" "$repo" "$AHEAD"
  fi
done
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 6: FINAL METRICS SNAPSHOT

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    FINAL METRICS SNAPSHOT                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Current Metrics (for session log)                               │"
echo "├─────────────────────────────────────────────────────────────────┤"
LOGINS=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_login_total" | grep -v "^#" | awk '{print $2}' | head -1)
SIGNUPS=$(curl -s http://localhost:3010/metrics 2>/dev/null | grep "uc_auth_signup_total" | grep -v "^#" | awk '{print $2}' | head -1)
EVENTS=$(curl -s http://localhost:4101/metrics 2>/dev/null | grep "social_events_enqueued_total" | grep -v "^#" | awk '{sum+=$2} END {print sum}')
printf "│ 🔐 Total Logins: %-46s │\n" "${LOGINS:-0}"
printf "│ 📝 Total Signups: %-45s │\n" "${SIGNUPS:-0}"
printf "│ 📨 Social Events: %-45s │\n" "${EVENTS:-0}"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""
CONTAINERS=$(docker ps --format "{{.Names}}" | wc -l)
echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ 🐳 Active Containers: %-41s │\n" "$CONTAINERS"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 7: BACKUP STATUS CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      BACKUP STATUS CHECK                         ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BACKUP VERIFICATION                                             │"
echo "├─────────────────────────────────────────────────────────────────┤"
LAST_BACKUP=$(sudo ls -td /root/uc-backups/2026-*/ 2>/dev/null | head -1)
if [ -n "$LAST_BACKUP" ]; then
  BACKUP_DATE=$(basename "$LAST_BACKUP")
  printf "│ 📅 Last backup: %-47s │\n" "$BACKUP_DATE"
  # Check success
  LAST_LOG=$(sudo tail -5 /root/uc-backups/logs/cron.log 2>/dev/null | grep -i "completed successfully")
  if [ -n "$LAST_LOG" ]; then
    printf "│ 🟢 Status: SUCCESSFUL                                          │\n"
  else
    printf "│ 🔴 Status: CHECK REQUIRED                                      │\n"
  fi
else
  printf "│ ⚠️  No backups found!                                           │\n"
fi
echo "├─────────────────────────────────────────────────────────────────┤"
CURRENT_HOUR=$(date +%H)
if [ "$CURRENT_HOUR" -lt 3 ]; then
  NEXT_BACKUP="Today at 03:00 AM"
else
  NEXT_BACKUP="Tomorrow at 03:00 AM"
fi
printf "│ ⏭️  Next backup: %-46s │\n" "$NEXT_BACKUP"
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 8: DATABASE HEALTH CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                   DATABASE HEALTH CHECK                          ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

CLOSE_DB_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ MySQL Status at Close                                           │"
echo "├─────────────────────────────────────────────────────────────────┤"
MYSQL_CONTAINER=$(docker ps --filter "name=mysql" --filter "status=running" --format "{{.Names}}" | head -1)
if [ -n "$MYSQL_CONTAINER" ]; then
  PING=$(docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null || echo "FAILED")
  if echo "$PING" | grep -q "alive"; then
    printf "│ 🟢 MySQL: %-53s │\n" "Running and responsive"
  else
    printf "│ 🔴 MySQL: %-53s │\n" "Not responding"
    CLOSE_DB_STATUS="RED"
  fi

  # Check active connections
  CONNS=$(docker exec "$MYSQL_CONTAINER" mysql -N -e "SHOW STATUS LIKE 'Threads_connected';" 2>/dev/null | awk '{print $2}' || echo "?")
  printf "│ 📊 Active connections: %-40s │\n" "${CONNS:-unknown}"
else
  printf "│ 🔴 MySQL container: %-43s │\n" "Not running"
  CLOSE_DB_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ DB STATUS AT CLOSE: "
case "$CLOSE_DB_STATUS" in
  GREEN) printf "🟢 %-43s │\n" "HEALTHY" ;;
  RED)   printf "🔴 %-43s │\n" "CRITICAL" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 9: QUEUE STATUS CHECK

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                   QUEUE STATUS AT CLOSE                          ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

CLOSE_QUEUE_STATUS="GREEN"
REDIS_PASS=$(cat /etc/uc/secrets/redis_password 2>/dev/null)

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ BullMQ Queue Health                                             │"
echo "├─────────────────────────────────────────────────────────────────┤"
REDIS_PING=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ping 2>/dev/null || echo "FAILED")
if [ "$REDIS_PING" = "PONG" ]; then
  WAITING=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" LLEN "bull:social-events-queue:wait" 2>/dev/null || echo "0")
  FAILED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:failed" 2>/dev/null || echo "0")

  printf "│ ⏳ Waiting jobs: %-46s │\n" "${WAITING:-0}"

  if [ "${FAILED:-0}" -gt 0 ]; then
    printf "│ 🔴 Failed jobs: %-47s │\n" "$FAILED (ATTENTION)"
    CLOSE_QUEUE_STATUS="RED"
  else
    printf "│ 🟢 Failed jobs: %-47s │\n" "0"
  fi

  # Warn if backlog at close
  if [ "${WAITING:-0}" -gt 50 ]; then
    printf "│ 🟡 WARNING: %-51s │\n" "Jobs pending at close"
    [ "$CLOSE_QUEUE_STATUS" != "RED" ] && CLOSE_QUEUE_STATUS="YELLOW"
  fi
else
  printf "│ 🔴 Redis: %-53s │\n" "Not reachable"
  CLOSE_QUEUE_STATUS="RED"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
printf "│ QUEUE STATUS AT CLOSE: "
case "$CLOSE_QUEUE_STATUS" in
  GREEN)  printf "🟢 %-40s │\n" "HEALTHY" ;;
  YELLOW) printf "🟡 %-40s │\n" "BACKLOG EXISTS" ;;
  RED)    printf "🔴 %-40s │\n" "ISSUES PRESENT" ;;
esac
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 10: SECURITY CHECK AT CLOSE

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                   SECURITY CHECK AT CLOSE                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Final Secrets Scan                                              │"
echo "├─────────────────────────────────────────────────────────────────┤"
cd /srv/unofficial/prod/app
SECRETS_RISK=0

for repo in unofficial-communities uc-api uc-webhooks uc-world uc-system-docs; do
  if [ -d "$repo/.git" ]; then
    STAGED=$(git -C "$repo" diff --cached --name-only 2>/dev/null)
    for file in $STAGED; do
      if [ -f "$repo/$file" ]; then
        if grep -qiE "(password|secret|api_key|private_key)\s*[:=]" "$repo/$file" 2>/dev/null; then
          printf "│ 🔴 STAGED SECRET: %-44s │\n" "$repo/$file"
          SECRETS_RISK=1
        fi
      fi
    done
  fi
done

if [ "$SECRETS_RISK" -eq 0 ]; then
  printf "│ 🟢 No secrets detected in staged files                        │\n"
fi
cd /srv/unofficial/prod/app
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 11: OBSERVABILITY CHECK AT CLOSE

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                 OBSERVABILITY CHECK AT CLOSE                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

CLOSE_OBS_STATUS="GREEN"

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Container Health                                                │"
echo "├─────────────────────────────────────────────────────────────────┤"

# Check for restarting containers
RESTARTING=$(docker ps --filter "status=restarting" --format "{{.Names}}" | wc -l)
if [ "$RESTARTING" -gt 0 ]; then
  printf "│ 🔴 Restarting containers: %-37s │\n" "$RESTARTING"
  docker ps --filter "status=restarting" --format "{{.Names}}" | while read name; do
    printf "│    - %-57s │\n" "$name"
  done
  CLOSE_OBS_STATUS="RED"
else
  printf "│ 🟢 No restarting containers                                   │\n"
fi

# Check total restarts
TOTAL_RESTARTS=0
for CONTAINER in $(docker ps --format "{{.Names}}"); do
  R=$(docker inspect "$CONTAINER" --format '{{.RestartCount}}' 2>/dev/null || echo "0")
  TOTAL_RESTARTS=$((TOTAL_RESTARTS + R))
done
printf "│ 📊 Total restart count: %-39s │\n" "$TOTAL_RESTARTS"
echo "└─────────────────────────────────────────────────────────────────┘"
echo ""

echo "┌─────────────────────────────────────────────────────────────────┐"
echo "│ Service Response Times at Close                                 │"
echo "├─────────────────────────────────────────────────────────────────┤"
FE_TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:3000 2>/dev/null || echo "ERR")
API_TIME=$(curl -s -o /dev/null -w "%{time_total}" http://localhost:3010/health 2>/dev/null || echo "ERR")
if [ "$FE_TIME" != "ERR" ]; then
  FE_MS=$(echo "$FE_TIME * 1000" | bc 2>/dev/null | cut -d'.' -f1 || echo "?")
  printf "│ 🌐 Frontend: %-50s │\n" "${FE_MS:-?}ms"
fi
if [ "$API_TIME" != "ERR" ]; then
  API_MS=$(echo "$API_TIME * 1000" | bc 2>/dev/null | cut -d'.' -f1 || echo "?")
  printf "│ 🔧 API: %-55s │\n" "${API_MS:-?}ms"
fi
echo "└─────────────────────────────────────────────────────────────────┘"
```

## PHASE 12: SESSION END SUMMARY

Provide this final summary:

```
╔══════════════════════════════════════════════════════════════════╗
║                     SESSION END SUMMARY                          ║
╠══════════════════════════════════════════════════════════════════╣
║ Date: YYYY-MM-DD                                                 ║
║ Duration: ~X hours                                               ║
║                                                                  ║
║ ACCOMPLISHED:                                                    ║
║   1. [Main accomplishment]                                       ║
║   2. [Secondary accomplishment]                                  ║
║   3. [Other work]                                                ║
║                                                                  ║
║ FILES UPDATED:                                                   ║
║   • Repository files: X/10                                       ║
║   • System files: X/18                                           ║
║   • Session log: ✓ Added                                         ║
║                                                                  ║
║ GIT STATUS:                                                      ║
║   • unofficial-communities: [Synced/X unpushed]                  ║
║   • uc-api: [Synced/X unpushed]                                  ║
║   • uc-webhooks: [Synced/X unpushed]                             ║
║   • uc-world: [Synced/X unpushed]                                ║
║   • uc-system-docs: [Synced/X unpushed]                          ║
║                                                                  ║
║ SYSTEM STATUS AT CLOSE:                                          ║
║   • Disk: XX%  Memory: XX%  Load: X.XX                           ║
║   • Containers: X running / X restarting                         ║
║   • Services: X/7 healthy                                        ║
║   • Database: [🟢/🔴]  Queue: [🟢/🟡/🔴]                         ║
║   • Failed jobs: X                                               ║
║                                                                  ║
║ BACKUP:                                                          ║
║   • Last: [DATE] - [🟢 SUCCESS / 🔴 FAILED]                      ║
║   • Next: [Today/Tomorrow at 03:00 AM]                           ║
║                                                                  ║
║ SECURITY:                                                        ║
║   • Staged secrets: [None / X files flagged - DO NOT PUSH]       ║
║                                                                  ║
║ NEXT SESSION PRIORITIES:                                         ║
║   1. [HIGH] [First priority]                                     ║
║   2. [MEDIUM] [Second priority]                                  ║
║   3. [LOW] [Third priority]                                      ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║                  SAFE TO LEAVE UNATTENDED?                       ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║ CRITERIA:                                                        ║
║   [✓/✗] No containers restarting                                 ║
║   [✓/✗] Failed jobs backlog < 10                                 ║
║   [✓/✗] Database connectivity OK                                 ║
║   [✓/✗] Last backup successful                                   ║
║   [✓/✗] All repos synced (no unpushed commits)                   ║
║   [✓/✗] No secrets staged for commit                             ║
║                                                                  ║
║ ┌────────────────────────────────────────────────────────────┐   ║
║ │  VERDICT: [🟢 YES - Safe] / [🔴 NO - Action Required]      │   ║
║ └────────────────────────────────────────────────────────────┘   ║
║                                                                  ║
║ [If NO, list required actions before leaving]                    ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝

Session closed. All 29 files reviewed/updated. Context preserved.
```
```

---

## Quick Reference

### Key Paths
| Item | Path |
|------|------|
| App Base | `/srv/unofficial/prod/app/` |
| System Docs | `/srv/unofficial/prod/app/uc-system-docs/` |
| Docker Compose | `/srv/unofficial/prod/app/docker-compose.prod.yml` |
| Secrets | `/etc/uc/secrets/` |
| Backups | `/root/uc-backups/` |
| VPS | `caue@72.61.134.35` |

### Service Ports
| Service | Port | Metrics |
|---------|------|---------|
| unofficial-communities | 3000 | - |
| uc-api | 3010 | `/metrics` |
| uc-webhooks | 4101 | `/metrics` |
| uc-world | 3005 | - |
| Prometheus | 9090 | - |
| Grafana | 3100 | - |

### Monitoring URLs
| Tool | URL |
|------|-----|
| Grafana | http://localhost:3100 |
| Prometheus | http://localhost:9090 |
| API Metrics | http://localhost:3010/metrics |
| Webhooks Metrics | http://localhost:4101/metrics |

### GO / NO-GO Criteria Reference

| Check | GO | CAUTION | NO-GO |
|-------|-----|---------|-------|
| VPS SSH | Connected | - | Disconnected |
| GitHub Auth | OK | - | Failed |
| Database | Connected | Pending migrations | Connection failed |
| API /health | 200 | Slow (>500ms) | Down |
| Redis | PONG | - | Connection failed |
| Queue backlog | <50 | 50-100 | >100 or DOWN |
| Failed jobs | 0 | 1-10 | >10 |
| Container restarts | 0 | 1-5 | >5 or restarting |
| Secrets in git | None | - | Detected |

### SAFE TO LEAVE Criteria Reference

| Check | SAFE | NOT SAFE |
|-------|------|----------|
| Containers restarting | 0 | >0 |
| Failed jobs | <10 | ≥10 |
| Database | Connected | Disconnected |
| Backup | Last successful | Last failed |
| Git repos | All synced | Unpushed commits |
| Staged secrets | None | Any detected |

---

*Version 6.0 - 2026-01-15 - Added comprehensive verification phases (DB, Queue, E2E, Security, Observability) and GO/NO-GO gates*

<!-- Last Reviewed: 2026-01-17 - No updates needed -->
