# UC Session Open

Execute the startup checklist for UC development session.

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
echo "║                      SYSTEM HEALTH CHECK                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# Disk
DISK_PCT=$(df -h / | tail -1 | awk '{print $5}' | tr -d '%')
DISK_INFO=$(df -h / | tail -1 | awk '{print $3"/"$2" ("$5")"}')
[ "$DISK_PCT" -gt 90 ] && echo "🔴 Disk: $DISK_INFO - CRITICAL" || { [ "$DISK_PCT" -gt 80 ] && echo "🟡 Disk: $DISK_INFO - WARNING" || echo "🟢 Disk: $DISK_INFO"; }

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
echo "║                    DOCKER SERVICES STATUS                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" 2>/dev/null

UNHEALTHY=$(docker ps --filter "status=restarting" --format "{{.Names}}" 2>/dev/null | wc -l)
[ "$UNHEALTHY" -gt 0 ] && echo "" && echo "⚠️  WARNING: $UNHEALTHY container(s) restarting"
```

## PHASE 4: SERVICE HEALTH PROBES

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    SERVICE HEALTH PROBES                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

for PORT in 3000 3005 3010 9090 3100; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 http://localhost:$PORT/health 2>/dev/null || curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 http://localhost:$PORT 2>/dev/null || echo "DOWN")
  [ "$STATUS" = "200" ] && ICON="🟢" || { [ "$STATUS" = "DOWN" ] && ICON="🔴" || ICON="🟡"; }
  echo "$ICON Port $PORT: HTTP $STATUS"
done

REDIS_PASS=$(docker exec app-uc-redis-1 cat /run/secrets/redis_password 2>/dev/null)
REDIS_STATUS=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ping 2>/dev/null || echo "DOWN")
[ "$REDIS_STATUS" = "PONG" ] && echo "🟢 Redis: $REDIS_STATUS" || echo "🔴 Redis: $REDIS_STATUS"

echo ""
echo "── Staging Environment ──"
STAGING_API=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 2 http://localhost:3011/health 2>/dev/null || echo "DOWN")
[ "$STAGING_API" = "200" ] && echo "🟢 Staging API (3011): HTTP $STAGING_API" || echo "🟡 Staging API (3011): HTTP $STAGING_API"
```

## PHASE 5: DATABASE STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      DATABASE STATUS                           ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

MYSQL_CONTAINER=$(docker ps --filter "name=api-db" --filter "status=running" --format "{{.Names}}" | grep -v staging | head -1)
if [ -n "$MYSQL_CONTAINER" ]; then
  DB_URL=$(docker exec "$MYSQL_CONTAINER" printenv DATABASE_URL 2>/dev/null)
  DB_USER=$(echo "$DB_URL" | sed -n 's|.*://\([^:]*\):.*|\1|p')
  DB_PASS=$(echo "$DB_URL" | sed -n 's|.*://[^:]*:\([^@]*\)@.*|\1|p')
  if [ -n "$DB_USER" ] && [ -n "$DB_PASS" ]; then
    PING=$(docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost -u "$DB_USER" -p"$DB_PASS" 2>/dev/null || echo "FAILED")
  else
    PING=$(docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null || echo "FAILED")
  fi
  echo "$PING" | grep -q "alive" && echo "🟢 MySQL: Running ($MYSQL_CONTAINER)" || echo "🔴 MySQL: Not responding"
else
  echo "🔴 MySQL: No container found"
fi

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
echo "║                       QUEUE STATUS                             ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

REDIS_PASS=$(docker exec app-uc-redis-1 cat /run/secrets/redis_password 2>/dev/null)
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
echo "║                        GIT STATUS                              ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
NON_MAIN_BRANCHES=""
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  UNPUSHED=$(git -C "$repo" log @{u}..HEAD --oneline 2>/dev/null | wc -l)
  BRANCH=$(git -C "$repo" branch --show-current 2>/dev/null)

  STATUS=""
  [ "$CHANGES" -gt 0 ] && STATUS="⚠️ $CHANGES uncommitted"
  [ "$UNPUSHED" -gt 0 ] && STATUS="$STATUS ⚠️ $UNPUSHED unpushed"
  [ -z "$STATUS" ] && STATUS="✓ Clean"

  # Warn if not on main branch
  BRANCH_WARN=""
  if [ "$BRANCH" != "main" ]; then
    BRANCH_WARN=" 🔀 NOT ON MAIN"
    NON_MAIN_BRANCHES="$NON_MAIN_BRANCHES $repo:$BRANCH"
  fi

  echo "📁 $repo ($BRANCH): $STATUS$BRANCH_WARN"
done

if [ -n "$NON_MAIN_BRANCHES" ]; then
  echo ""
  echo "⚠️  WARNING: Some repos not on main branch:$NON_MAIN_BRANCHES"
fi
```

## PHASE 8: BACKUP STATUS

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                       BACKUP STATUS                            ║"
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

## PHASE 9: DOCUMENTATION CONSISTENCY AUDIT

This phase catches documentation drift before work begins.

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║              DOCUMENTATION CONSISTENCY AUDIT                   ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

ISSUES=0

# Count actual numbered docs on disk
ACTUAL_COUNT=$(ls /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md 2>/dev/null | wc -l)

# Count entries in canonical index registry table
INDEX_COUNT=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null)

# Check CLAUDE.md document count (text and table)
CLAUDE_TEXT=$(grep -oP '\d+(?= numbered files)' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null | head -1)
CLAUDE_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null)

# Check README.md document count (text and table)
README_TEXT=$(grep -oP '\d+(?= numbered documents)' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null | head -1)
README_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null)

echo "📊 Files on disk:        $ACTUAL_COUNT"
echo "📊 Canonical index:      $INDEX_COUNT entries"
echo "📊 CLAUDE.md states:     $CLAUDE_TEXT files ($CLAUDE_TABLE table rows)"
echo "📊 README.md states:     $README_TEXT documents ($README_TABLE table rows)"

# Check all counts match
for LABEL_VAL in "Canonical_index:$INDEX_COUNT" "CLAUDE.md_text:$CLAUDE_TEXT" "CLAUDE.md_table:$CLAUDE_TABLE" "README.md_text:$README_TEXT" "README.md_table:$README_TABLE"; do
  LABEL=$(echo "$LABEL_VAL" | cut -d: -f1)
  VAL=$(echo "$LABEL_VAL" | cut -d: -f2)
  if [ "$VAL" != "$ACTUAL_COUNT" ]; then
    echo "🔴 MISMATCH: $LABEL says $VAL but $ACTUAL_COUNT files exist on disk"
    ISSUES=$((ISSUES + 1))
  fi
done

# Check for files on disk not in canonical index
echo ""
echo "── Missing from Index ──"
MISSING=0
for f in /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md; do
  BASENAME=$(basename "$f")
  if ! grep -q "$BASENAME" /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null; then
    echo "🔴 NOT INDEXED: $BASENAME"
    MISSING=$((MISSING + 1))
    ISSUES=$((ISSUES + 1))
  fi
done
[ "$MISSING" -eq 0 ] && echo "🟢 All files registered in index"

# Check for stale docs (Last Updated/Reviewed older than 7 days)
echo ""
echo "── Staleness Check (>7 days since last review) ──"
STALE=0
SEVEN_DAYS_AGO=$(date -d '7 days ago' +%Y-%m-%d)
for f in /srv/unofficial/prod/app/uc-system-docs/*.md; do
  LAST_DATE=$(grep -oP '(?:Last Updated|Last Reviewed): \K\d{4}-\d{2}-\d{2}' "$f" 2>/dev/null | tail -1)
  if [ -n "$LAST_DATE" ] && [[ "$LAST_DATE" < "$SEVEN_DAYS_AGO" ]]; then
    echo "🟡 STALE: $(basename $f) - last updated $LAST_DATE"
    STALE=$((STALE + 1))
  fi
done
[ "$STALE" -eq 0 ] && echo "🟢 All documents reviewed within 7 days"

# Check command file sync (home vs project)
echo ""
echo "── Command File Sync ──"
DESYNC=0
for CMD in uc-open.md uc-close.md; do
  HOME_HASH=$(md5sum /home/caue/.claude/commands/$CMD 2>/dev/null | awk '{print $1}')
  PROJ_HASH=$(md5sum /srv/unofficial/prod/app/.claude/commands/$CMD 2>/dev/null | awk '{print $1}')
  if [ "$HOME_HASH" != "$PROJ_HASH" ]; then
    echo "🔴 DESYNC: $CMD differs between home and project"
    DESYNC=$((DESYNC + 1))
    ISSUES=$((ISSUES + 1))
  fi
done
[ "$DESYNC" -eq 0 ] && echo "🟢 Command files in sync"

# Check skill file sync (home vs project)
echo ""
echo "── Skill File Sync ──"
SKILL_DESYNC=0
for SKILL_DIR in /home/caue/.claude/skills/*/; do
  [ ! -d "$SKILL_DIR" ] && continue
  SKILL_NAME=$(basename "$SKILL_DIR")
  HOME_SKILL="$SKILL_DIR/SKILL.md"
  PROJ_SKILL="/srv/unofficial/prod/app/.claude/skills/$SKILL_NAME/SKILL.md"
  if [ -f "$HOME_SKILL" ]; then
    if [ -f "$PROJ_SKILL" ]; then
      HOME_HASH=$(md5sum "$HOME_SKILL" 2>/dev/null | awk '{print $1}')
      PROJ_HASH=$(md5sum "$PROJ_SKILL" 2>/dev/null | awk '{print $1}')
      if [ "$HOME_HASH" != "$PROJ_HASH" ]; then
        echo "🔴 DESYNC: skill/$SKILL_NAME differs between home and project"
        SKILL_DESYNC=$((SKILL_DESYNC + 1))
        ISSUES=$((ISSUES + 1))
      fi
    else
      echo "🔴 MISSING: skill/$SKILL_NAME exists in home but not in project"
      SKILL_DESYNC=$((SKILL_DESYNC + 1))
      ISSUES=$((ISSUES + 1))
    fi
  fi
done
for SKILL_DIR in /srv/unofficial/prod/app/.claude/skills/*/; do
  [ ! -d "$SKILL_DIR" ] && continue
  SKILL_NAME=$(basename "$SKILL_DIR")
  PROJ_SKILL="$SKILL_DIR/SKILL.md"
  HOME_SKILL="/home/caue/.claude/skills/$SKILL_NAME/SKILL.md"
  if [ -f "$PROJ_SKILL" ] && [ ! -f "$HOME_SKILL" ]; then
    echo "🔴 MISSING: skill/$SKILL_NAME exists in project but not in home"
    SKILL_DESYNC=$((SKILL_DESYNC + 1))
    ISSUES=$((ISSUES + 1))
  fi
done
[ "$SKILL_DESYNC" -eq 0 ] && echo "🟢 Skill files in sync"

echo ""
[ "$ISSUES" -eq 0 ] && echo "🟢 DOCUMENTATION CONSISTENCY: ALL CHECKS PASSED" || echo "🔴 DOCUMENTATION CONSISTENCY: $ISSUES ISSUE(S) FOUND - fix before proceeding"
```

**If any mismatches are found, fix them before starting work.** This prevents documentation drift from accumulating across sessions.

## PHASE 10: CROSS-DOCUMENT DEPENDENCY & CONTENT AUDIT

This phase catches **content-level drift** that structural checks miss: documents that require updates to other documents, stale status fields, and `Last Updated` headers that don't match actual file modifications.

### 10.1 Pending Cross-Document Update Requirements

Scan all numbered docs for sections that declare required updates to other documents (e.g., "System Doc Updates Required", "Required Updates to Canonical Files"). Surface any unchecked items.

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║          CROSS-DOCUMENT DEPENDENCY AUDIT                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

DOCS_DIR="/srv/unofficial/prod/app/uc-system-docs"
DEP_ISSUES=0

echo "── Pending Update Requirements ──"
for f in "$DOCS_DIR"/[0-9]*.md; do
  BASENAME=$(basename "$f")
  DOC_NUM=$(echo "$BASENAME" | grep -oP '^\d+')

  # Find sections that declare required updates to other docs
  # Look for unchecked items (- [ ]) under headings mentioning "update", "required", or referencing other doc numbers
  PENDING=$(grep -n '^\- \[ \]' "$f" 2>/dev/null | grep -iE 'update|add|register|integrate|07_|08_|06_|09_|12_|04_|11_|13_|19_|20_|22_|CLAUDE|README' | head -10)
  if [ -n "$PENDING" ]; then
    echo "🔴 $BASENAME has pending cross-doc updates:"
    echo "$PENDING" | sed 's/^/   /'
    DEP_ISSUES=$((DEP_ISSUES + 1))
  fi
done
[ "$DEP_ISSUES" -eq 0 ] && echo "🟢 No pending cross-document update requirements found"
```

### 10.2 Last Updated Header vs Git Modification

Detect when a file was modified by git more recently than its `Last Updated` header claims.

```bash
echo ""
echo "── Last Updated vs Git Modification ──"
HEADER_ISSUES=0
cd "$DOCS_DIR"
for f in [0-9]*.md CLAUDE.md README.md; do
  [ ! -f "$f" ] && continue
  HEADER_DATE=$(grep -oP '(?:Last Updated|Last Reviewed): \K\d{4}-\d{2}-\d{2}' "$f" 2>/dev/null | tail -1)
  GIT_DATE=$(git log -1 --format='%ai' -- "$f" 2>/dev/null | cut -d' ' -f1)

  if [ -n "$HEADER_DATE" ] && [ -n "$GIT_DATE" ]; then
    if [[ "$GIT_DATE" > "$HEADER_DATE" ]]; then
      echo "🟡 $f: header says $HEADER_DATE but git shows modified $GIT_DATE"
      HEADER_ISSUES=$((HEADER_ISSUES + 1))
    fi
  elif [ -z "$HEADER_DATE" ]; then
    echo "🟡 $f: no Last Updated/Reviewed header found"
    HEADER_ISSUES=$((HEADER_ISSUES + 1))
  fi
done
[ "$HEADER_ISSUES" -eq 0 ] && echo "🟢 All headers match git history"
```

### 10.3 Stale Status Fields

Check documents with `Status:` fields to flag any that still say "Implementation Pending" or have completion percentages that may need review.

```bash
echo ""
echo "── Status Field Review ──"
STATUS_FLAGS=0
for f in "$DOCS_DIR"/[0-9]*.md; do
  BASENAME=$(basename "$f")
  # Extract status from header (first 10 lines)
  DOC_STATUS=$(head -10 "$f" | grep -oP '(?<=\*\*Status:\*\* ).*' 2>/dev/null | head -1)
  if [ -n "$DOC_STATUS" ]; then
    # Flag docs with action-pending statuses for awareness
    if echo "$DOC_STATUS" | grep -qiE 'pending|draft|in progress|active execution'; then
      echo "📋 $BASENAME: Status = \"$DOC_STATUS\""
      STATUS_FLAGS=$((STATUS_FLAGS + 1))
    fi
  fi
done
[ "$STATUS_FLAGS" -eq 0 ] && echo "🟢 No action-pending status fields" || echo "ℹ️  $STATUS_FLAGS document(s) have action-pending status — review if still accurate"
```

**Review the output.** Cross-document pending items are blockers that must be resolved before implementing the features they gate. Status fields and header mismatches are warnings to review during the session.

## PHASE 11: SUMMARY

After all checks, provide:

```
SESSION START SUMMARY
=====================
Date: [current date/time]
Files loaded: [count from Phase 1]
Services: [healthy/total from Phase 4]
Staging: [status from Phase 4]
Database: [status from Phase 5]
Queue: [status from Phase 6]
Git: [repos with uncommitted/unpushed changes]
Branches: [any repos not on main]
Documentation: [consistent / N issues found]
Cross-doc dependencies: [clean / N pending items]

DECISION: [GO / CAUTION / NO-GO]
- GO: All systems healthy, all repos on main, docs consistent
- CAUTION: Minor issues, feature branches active, or stale docs
- NO-GO: Critical issues or doc mismatches must be resolved first

What would you like to work on?
```
