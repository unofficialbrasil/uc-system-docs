# Claude Code Session Prompts

**Last Updated:** 2026-01-14
**Version:** 5.0

---

## Files Registry (26 Files Total)

### Repository Files (8 files)

| Repo | CLAUDE.md | README.md |
|------|-----------|-----------|
| unofficial-communities | `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md` | `/srv/unofficial/prod/app/unofficial-communities/README.md` |
| uc-api | `/srv/unofficial/prod/app/uc-api/CLAUDE.md` | `/srv/unofficial/prod/app/uc-api/README.md` |
| uc-webhooks | `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md` | `/srv/unofficial/prod/app/uc-webhooks/README.md` |
| uc-world | `/srv/unofficial/prod/app/uc-world/CLAUDE.md` | `/srv/unofficial/prod/app/uc-world/README.md` |

### System Documentation (18 files)

Location: `/srv/unofficial/prod/app/system/`

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

---

## 1. Open Session Prompt

**Copy and paste at the beginning of EVERY development session:**

```markdown
Hi Claude, let's start a development session. Execute the COMPLETE startup checklist below. Do NOT skip any step.

## PHASE 1: READ ALL FILES (MANDATORY - NO EXCEPTIONS)

You MUST read ALL 26 files to have full context. No exceptions.

### 1.1 Repository CLAUDE.md Files (4 files)
Read each file completely:
- `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-api/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md`
- `/srv/unofficial/prod/app/uc-world/CLAUDE.md`

### 1.2 Repository README.md Files (4 files)
Read each file completely:
- `/srv/unofficial/prod/app/unofficial-communities/README.md`
- `/srv/unofficial/prod/app/uc-api/README.md`
- `/srv/unofficial/prod/app/uc-webhooks/README.md`
- `/srv/unofficial/prod/app/uc-world/README.md`

### 1.3 System Documentation (18 files)
Read ALL files in `/srv/unofficial/prod/app/system/`:
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
for repo in unofficial-communities uc-api uc-webhooks uc-world; do
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
# Webhooks
WH_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:4101/health 2>/dev/null || echo "DOWN")
[ "$WH_STATUS" = "200" ] && WH_ICON="🟢" || WH_ICON="🔴"
printf "│ %-25s │ %-5s │ %s %-23s │\n" "uc-webhooks" "4101" "$WH_ICON" "$WH_STATUS"
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
# Social events from webhooks
EVENTS=$(curl -s http://localhost:4101/metrics 2>/dev/null | grep "social_events_enqueued_total" | grep -v "^#" | awk '{sum+=$2} END {print sum}')
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

## PHASE 9: SESSION START SUMMARY

After completing ALL phases, provide this summary:

```
╔══════════════════════════════════════════════════════════════════╗
║                    SESSION START SUMMARY                         ║
╠══════════════════════════════════════════════════════════════════╣
║ Date: YYYY-MM-DD HH:MM                                           ║
║                                                                  ║
║ FILES LOADED:                                                    ║
║   ✓ Repository files: 8/8                                        ║
║   ✓ System files: 18/18                                          ║
║   ✓ Total context: 26 files                                      ║
║                                                                  ║
║ SYSTEM STATUS:                                                   ║
║   • VPS: [Connected/Disconnected]                                ║
║   • GitHub: [Authenticated/Failed]                               ║
║   • Disk: [XX%]                                                  ║
║   • Memory: [XX%]                                                ║
║   • Containers: [X running]                                      ║
║   • Services: [X/7 healthy]                                      ║
║                                                                  ║
║ BACKUP STATUS:                                                   ║
║   • Last backup: [DATE] - [SUCCESS/FAILED]                       ║
║   • Next backup: [Today/Tomorrow at 03:00 AM]                    ║
║                                                                  ║
║ GIT STATUS:                                                      ║
║   • Uncommitted changes: [X repos]                               ║
║   • Unpushed commits: [X repos]                                  ║
║                                                                  ║
║ METRICS SINCE LAST SESSION:                                      ║
║   • Logins: [X]                                                  ║
║   • Signups: [X]                                                 ║
║   • Social events: [X]                                           ║
╚══════════════════════════════════════════════════════════════════╝

Session ready. What would you like to work on today?
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

For EACH of the 8 repository files below, either:
- **UPDATE** with changes made during this session, OR
- **ADD** a review stamp if no changes: `<!-- Last Reviewed: YYYY-MM-DD - No updates needed -->`

### CLAUDE.md Files (4 files)
| File | Action |
|------|--------|
| `/srv/unofficial/prod/app/unofficial-communities/CLAUDE.md` | Update "Current Status" |
| `/srv/unofficial/prod/app/uc-api/CLAUDE.md` | Update if API changed |
| `/srv/unofficial/prod/app/uc-webhooks/CLAUDE.md` | Update if webhooks changed |
| `/srv/unofficial/prod/app/uc-world/CLAUDE.md` | Update if World changed |

### README.md Files (4 files)
| File | Action |
|------|--------|
| `/srv/unofficial/prod/app/unofficial-communities/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-api/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-webhooks/README.md` | Update if significant changes |
| `/srv/unofficial/prod/app/uc-world/README.md` | Update if significant changes |

## PHASE 3: UPDATE ALL SYSTEM FILES (MANDATORY)

For EACH of the 18 system files, either:
- **UPDATE** the content if relevant changes were made, OR
- **ADD** a review stamp at the end: `<!-- Last Reviewed: YYYY-MM-DD - No updates needed -->`

Location: `/srv/unofficial/prod/app/system/`

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
for repo in unofficial-communities uc-api uc-webhooks uc-world; do
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
for repo in unofficial-communities uc-api uc-webhooks uc-world; do
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

## PHASE 8: SESSION END SUMMARY

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
║   • Repository files: X/8                                        ║
║   • System files: X/18                                           ║
║   • Session log: ✓ Added                                         ║
║                                                                  ║
║ GIT STATUS:                                                      ║
║   • unofficial-communities: [Synced/X unpushed]                  ║
║   • uc-api: [Synced/X unpushed]                                  ║
║   • uc-webhooks: [Synced/X unpushed]                             ║
║   • uc-world: [Synced/X unpushed]                                ║
║                                                                  ║
║ SYSTEM STATUS AT CLOSE:                                          ║
║   • Disk: XX%                                                    ║
║   • Memory: XX%                                                  ║
║   • Load: X.XX                                                   ║
║   • Containers: X running                                        ║
║   • Services: X/7 healthy                                        ║
║                                                                  ║
║ BACKUP STATUS:                                                   ║
║   • Last backup: [DATE] - [SUCCESS/FAILED]                       ║
║   • Next backup: [Today/Tomorrow at 03:00 AM]                    ║
║                                                                  ║
║ NEXT SESSION PRIORITIES:                                         ║
║   1. [HIGH] [First priority]                                     ║
║   2. [MEDIUM] [Second priority]                                  ║
║   3. [LOW] [Third priority]                                      ║
║                                                                  ║
║ WARNINGS/CONCERNS:                                               ║
║   [Any issues to address or "None"]                              ║
╚══════════════════════════════════════════════════════════════════╝

Session closed. All 26 files reviewed/updated. Context preserved.
```
```

---

## Quick Reference

### Key Paths
| Item | Path |
|------|------|
| App Base | `/srv/unofficial/prod/app/` |
| System Docs | `/srv/unofficial/prod/app/system/` |
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

---

*Version 5.1 - 2026-01-14 - Added comprehensive backup checks*
