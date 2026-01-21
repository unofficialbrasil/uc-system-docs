# Claude Code Session Prompts

**Last Updated:** 2026-01-21

These prompts are **self-maintaining** - they dynamically discover files, services, and configuration at runtime. No manual updates needed.

---

## 1. Open Session Prompt

**Copy and paste at the beginning of EVERY development session:**

```markdown
Hi Claude, let's start a development session. Execute the startup checklist below.

## PHASE 1: DISCOVER AND READ ALL DOCUMENTATION

Dynamically find and read all project documentation:

### 1.1 Find All CLAUDE.md Files
```bash
find /srv/unofficial/prod/app -name "CLAUDE.md" -type f 2>/dev/null | sort
```

### 1.2 Find All README.md Files
```bash
find /srv/unofficial/prod/app -maxdepth 2 -name "README.md" -type f 2>/dev/null | sort
```

### 1.3 Find All System Documentation
```bash
ls /srv/unofficial/prod/app/uc-system-docs/*.md 2>/dev/null | sort
```

**Read ALL files discovered above.** The total count will vary as the project evolves.

## PHASE 2: SYSTEM HEALTH CHECK

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      SYSTEM HEALTH CHECK                         ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Disk
DISK_PCT=$(df -h / | tail -1 | awk '{print $5}' | tr -d '%')
DISK_INFO=$(df -h / | tail -1 | awk '{print $3"/"$2" ("$5")"}')
[ "$DISK_PCT" -gt 90 ] && echo "🔴 Disk: $DISK_INFO - CRITICAL" || [ "$DISK_PCT" -gt 80 ] && echo "🟡 Disk: $DISK_INFO - WARNING" || echo "🟢 Disk: $DISK_INFO"

# Memory
MEM_PCT=$(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100)}')
MEM_INFO=$(free -h | grep Mem | awk '{print $3"/"$2}')
[ "$MEM_PCT" -gt 90 ] && echo "🔴 Memory: $MEM_INFO ($MEM_PCT%)" || echo "🟢 Memory: $MEM_INFO ($MEM_PCT%)"

# Load
LOAD=$(uptime | awk -F'load average:' '{print $2}' | xargs)
echo "⚡ Load: $LOAD"
echo "🕐 $(uptime -p)"
```

## PHASE 3: DOCKER SERVICES STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    DOCKER SERVICES STATUS                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# List all running containers with ports
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" 2>/dev/null

# Check for unhealthy/restarting containers
UNHEALTHY=$(docker ps --filter "status=restarting" --format "{{.Names}}" 2>/dev/null | wc -l)
[ "$UNHEALTHY" -gt 0 ] && echo "" && echo "⚠️  WARNING: $UNHEALTHY container(s) restarting"
```

## PHASE 4: SERVICE HEALTH PROBES

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    SERVICE HEALTH PROBES                         ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Probe common service ports (discovered from docker ps)
for PORT in 3000 3005 3010 9090 3100; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 http://localhost:$PORT/health 2>/dev/null || curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 http://localhost:$PORT 2>/dev/null || echo "DOWN")
  [ "$STATUS" = "200" ] && ICON="🟢" || [ "$STATUS" = "DOWN" ] && ICON="🔴" || ICON="🟡"
  echo "$ICON Port $PORT: HTTP $STATUS"
done

# Redis check
REDIS_PASS=$(cat /etc/uc/secrets/redis_password 2>/dev/null)
REDIS_STATUS=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ping 2>/dev/null || echo "DOWN")
[ "$REDIS_STATUS" = "PONG" ] && echo "🟢 Redis: $REDIS_STATUS" || echo "🔴 Redis: $REDIS_STATUS"
```

## PHASE 5: DATABASE STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      DATABASE STATUS                             ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Find MySQL container
MYSQL_CONTAINER=$(docker ps --filter "name=mysql" --filter "status=running" --format "{{.Names}}" | head -1)
if [ -n "$MYSQL_CONTAINER" ]; then
  PING=$(docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null || echo "FAILED")
  echo "$PING" | grep -q "alive" && echo "🟢 MySQL: Running ($MYSQL_CONTAINER)" || echo "🔴 MySQL: Not responding"
else
  echo "🔴 MySQL: No container found"
fi

# Check Prisma migrations (if in uc-api)
if [ -f "/srv/unofficial/prod/app/uc-api/prisma/schema.prisma" ]; then
  cd /srv/unofficial/prod/app/uc-api
  MIGRATION=$(npx prisma migrate status 2>&1 | grep -E "up to date|pending|drift" | head -1)
  echo "📊 Migrations: $MIGRATION"
  cd /srv/unofficial/prod/app
fi
```

## PHASE 6: QUEUE STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                       QUEUE STATUS                               ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

REDIS_PASS=$(cat /etc/uc/secrets/redis_password 2>/dev/null)
if docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ping 2>/dev/null | grep -q "PONG"; then
  WAITING=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" LLEN "bull:social-events-queue:wait" 2>/dev/null || echo "0")
  ACTIVE=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" LLEN "bull:social-events-queue:active" 2>/dev/null || echo "0")
  FAILED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:failed" 2>/dev/null || echo "0")
  echo "⏳ Waiting: $WAITING | ⚡ Active: $ACTIVE | ❌ Failed: $FAILED"
  [ "${FAILED:-0}" -gt 0 ] && echo "⚠️  WARNING: Failed jobs need attention"
  [ "${WAITING:-0}" -gt 100 ] && echo "⚠️  WARNING: Queue backlog detected"
else
  echo "🔴 Cannot check queue - Redis unavailable"
fi
```

## PHASE 7: GIT STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                        GIT STATUS                                ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  UNPUSHED=$(git -C "$repo" log @{u}..HEAD --oneline 2>/dev/null | wc -l)
  BRANCH=$(git -C "$repo" branch --show-current 2>/dev/null)

  STATUS=""
  [ "$CHANGES" -gt 0 ] && STATUS="⚠️ $CHANGES uncommitted"
  [ "$UNPUSHED" -gt 0 ] && STATUS="$STATUS ⚠️ $UNPUSHED unpushed"
  [ -z "$STATUS" ] && STATUS="✓ Clean"

  echo "📁 $repo ($BRANCH): $STATUS"
done
```

## PHASE 8: BACKUP STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                       BACKUP STATUS                              ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

LAST_BACKUP=$(sudo ls -td /root/uc-backups/2*/ 2>/dev/null | head -1)
if [ -n "$LAST_BACKUP" ]; then
  BACKUP_DATE=$(basename "$LAST_BACKUP")
  BACKUP_SIZE=$(sudo du -sh "$LAST_BACKUP" 2>/dev/null | awk '{print $1}')
  echo "📅 Last backup: $BACKUP_DATE ($BACKUP_SIZE)"

  LAST_LOG=$(sudo tail -5 /root/uc-backups/logs/cron.log 2>/dev/null | grep -i "completed successfully")
  [ -n "$LAST_LOG" ] && echo "🟢 Status: Successful" || echo "🟡 Status: Check logs"
else
  echo "⚠️ No backups found"
fi
```

## PHASE 9: SUMMARY

After all checks, provide:

```
SESSION START SUMMARY
=====================
Date: [current date/time]
Files loaded: [count from Phase 1]
Services: [healthy/total from Phase 4]
Database: [status from Phase 5]
Queue: [status from Phase 6]
Git: [repos with uncommitted/unpushed changes]

DECISION: [GO / CAUTION / NO-GO]
- GO: All systems healthy
- CAUTION: Minor issues, can proceed with awareness
- NO-GO: Critical issues must be resolved first

What would you like to work on?
```
```

---

## 2. Close Session Prompt

**Copy and paste before ending ANY session:**

```markdown
Hi Claude, I need to close this session. Execute the shutdown checklist.

## PHASE 1: PRE-CLOSE SYSTEM CHECK

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    PRE-CLOSE SYSTEM CHECK                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Quick health snapshot
DISK=$(df -h / | tail -1 | awk '{print $5}')
MEM=$(free | grep Mem | awk '{printf("%.0f%%", $3/$2 * 100)}')
LOAD=$(uptime | awk -F'load average:' '{print $2}' | xargs | cut -d',' -f1)
echo "💾 Disk: $DISK | 🧠 Memory: $MEM | ⚡ Load: $LOAD"

# Service status
CONTAINERS=$(docker ps --format "{{.Names}}" 2>/dev/null | wc -l)
UNHEALTHY=$(docker ps --filter "status=restarting" 2>/dev/null | wc -l)
echo "🐳 Containers: $CONTAINERS running, $UNHEALTHY unhealthy"
```

## PHASE 2: DOCUMENTATION UPDATES

For each repository and system doc, either:
- **UPDATE** if changes were made during this session
- **SKIP** if no relevant changes (no review stamp needed)

Key files to consider:
```bash
# Find all documentation files
echo "Repository CLAUDE.md files:"
find /srv/unofficial/prod/app -name "CLAUDE.md" -type f 2>/dev/null

echo ""
echo "System documentation:"
ls /srv/unofficial/prod/app/uc-system-docs/*.md 2>/dev/null
```

**ALWAYS update 16_SESSION_LOG.md** with a session entry:

```markdown
### Session: YYYY-MM-DD

**Focus:** [Main topic]

**Accomplished:**
1. [Task 1]
2. [Task 2]

**Files Modified:**
- [file1]
- [file2]

**Next Priorities:**
1. [Priority 1]
2. [Priority 2]
```

## PHASE 3: GIT STATUS & COMMIT

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      GIT STATUS CHECK                            ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  echo "📁 $repo:"
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  UNPUSHED=$(git -C "$repo" log @{u}..HEAD --oneline 2>/dev/null | wc -l)

  if [ "$CHANGES" -gt 0 ]; then
    echo "   ⚠️ $CHANGES uncommitted changes:"
    git -C "$repo" status --porcelain 2>/dev/null | head -5 | sed 's/^/   /'
  fi

  if [ "$UNPUSHED" -gt 0 ]; then
    echo "   ⚠️ $UNPUSHED unpushed commits"
  fi

  [ "$CHANGES" -eq 0 ] && [ "$UNPUSHED" -eq 0 ] && echo "   ✓ Clean and synced"
  echo ""
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

## PHASE 4: DATABASE & QUEUE CHECK

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  DATABASE & QUEUE AT CLOSE                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# MySQL check
MYSQL_CONTAINER=$(docker ps --filter "name=mysql" --filter "status=running" --format "{{.Names}}" | head -1)
[ -n "$MYSQL_CONTAINER" ] && docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null | grep -q "alive" && echo "🟢 MySQL: Running" || echo "🔴 MySQL: Check required"

# Queue check
REDIS_PASS=$(cat /etc/uc/secrets/redis_password 2>/dev/null)
FAILED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:failed" 2>/dev/null || echo "?")
echo "📊 Queue failed jobs: $FAILED"
[ "${FAILED:-0}" -gt 10 ] && echo "⚠️ WARNING: High number of failed jobs"
```

## PHASE 5: SECURITY SCAN

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    SECURITY SCAN AT CLOSE                        ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
SECRETS_FOUND=0

for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||'); do
  STAGED=$(git -C "$repo" diff --cached --name-only 2>/dev/null)
  for file in $STAGED; do
    if [ -f "$repo/$file" ] && grep -qiE "(password|secret|api_key|private_key)\s*[:=]" "$repo/$file" 2>/dev/null; then
      echo "🔴 POTENTIAL SECRET: $repo/$file"
      SECRETS_FOUND=1
    fi
  done
done

[ "$SECRETS_FOUND" -eq 0 ] && echo "🟢 No secrets detected in staged files"
```

## PHASE 6: FINAL VERIFICATION

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    FINAL SYNC VERIFICATION                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
ALL_SYNCED=true
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  AHEAD=$(git -C "$repo" rev-list @{u}..HEAD --count 2>/dev/null || echo "0")
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  if [ "$AHEAD" -eq 0 ] && [ "$CHANGES" -eq 0 ]; then
    echo "✓ $repo: Synced"
  else
    echo "⚠️ $repo: NOT synced (uncommitted: $CHANGES, unpushed: $AHEAD)"
    ALL_SYNCED=false
  fi
done

echo ""
$ALL_SYNCED && echo "🟢 SAFE TO LEAVE: All repos synced" || echo "🔴 NOT SAFE: Uncommitted/unpushed changes exist"
```

## CLOSE SUMMARY

```
SESSION CLOSE SUMMARY
=====================
Date: [current date/time]
Session focus: [what was worked on]
Files modified: [list]
Commits pushed: [count]

System status:
- Services: [running/healthy]
- Database: [status]
- Queue failed jobs: [count]
- Secrets scan: [pass/fail]
- Git sync: [all synced / pending changes]

SAFE TO LEAVE: [YES / NO - list pending items]
```
```

---

## Quick Reference

### Key Paths (discover dynamically)
```bash
# Application root
/srv/unofficial/prod/app/

# Find all repos
find /srv/unofficial/prod/app -maxdepth 2 -name ".git" -type d | sed 's|/\.git||'

# Find docker compose
ls /srv/unofficial/prod/app/docker-compose*.yml

# Find secrets
ls /etc/uc/secrets/

# Find backups
ls /root/uc-backups/
```

### Service Discovery
```bash
# Running services and ports
docker ps --format "table {{.Names}}\t{{.Ports}}"

# Health check any service
curl -s http://localhost:[PORT]/health
```

### GO / NO-GO Criteria

| Check | GO | NO-GO |
|-------|-----|-------|
| Database | Connected | Connection failed |
| API health | Returns 200 | Down |
| Redis | PONG | Connection failed |
| Queue failed jobs | <10 | >10 |
| Container restarts | 0 | >5 |
| Secrets in staged files | None | Any detected |

### SAFE TO LEAVE Criteria

| Check | SAFE | NOT SAFE |
|-------|------|----------|
| All repos | Synced | Uncommitted/unpushed |
| Failed jobs | <10 | >=10 |
| Database | Connected | Disconnected |
| Containers | All running | Any restarting |

---

*This file is self-maintaining. Service ports, file counts, and configurations are discovered at runtime.*
