# UC Session Close

Execute the shutdown checklist for UC development session. **No phase can be skipped.**

## PHASE 1: PRE-CLOSE SYSTEM CHECK

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    PRE-CLOSE SYSTEM CHECK                      ║"
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

## PHASE 2: MANDATORY DOCUMENTATION UPDATES

This phase ensures NO documentation drift. Every change made during this session must be reflected in the docs. **Do not skip any sub-step.**

### 2.1 Identify Changed Repos

```bash
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                  IDENTIFY SESSION CHANGES                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  # Check uncommitted changes
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  # Check commits made this session (today)
  TODAY=$(date +%Y-%m-%d)
  TODAY_COMMITS=$(git -C "$repo" log --since="$TODAY" --oneline 2>/dev/null | wc -l)
  UNPUSHED=$(git -C "$repo" log @{u}..HEAD --oneline 2>/dev/null | wc -l)

  if [ "$CHANGES" -gt 0 ] || [ "$TODAY_COMMITS" -gt 0 ] || [ "$UNPUSHED" -gt 0 ]; then
    echo "📝 $repo: HAS CHANGES (uncommitted: $CHANGES, today's commits: $TODAY_COMMITS, unpushed: $UNPUSHED)"
    echo "   Changed files:"
    git -C "$repo" status --porcelain 2>/dev/null | head -10 | sed 's/^/   /'
    [ "$TODAY_COMMITS" -gt 0 ] && echo "   Today's commits:" && git -C "$repo" log --since="$TODAY" --oneline 2>/dev/null | sed 's/^/   /'
  else
    echo "   $repo: No changes"
  fi
  echo ""
done
```

### 2.2 Update Documentation for Changed Repos

For **EACH repo that had changes** during this session (code or config, not just docs), you **MUST**:

1. **Read its `CLAUDE.md`** and update:
   - "Current Status" section with what was done this session
   - Any structural changes (new files, moved files, new routes, new services)
   - The `<!-- Last Reviewed: YYYY-MM-DD -->` comment at the bottom

2. **Read its `README.md`** and update if changes affect:
   - Project overview or purpose
   - Setup/build commands
   - Project structure
   - Technology versions

3. **If only documentation changed** (no code): update the `<!-- Last Reviewed -->` stamp only.

### 2.3 Verify Canonical Index Consistency

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║              CANONICAL INDEX CONSISTENCY                       ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

ACTUAL=$(ls /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md 2>/dev/null | wc -l)
INDEX=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null)
CLAUDE_TEXT=$(grep -oP '\d+(?= numbered files)' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null | head -1)
CLAUDE_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null)
README_TEXT=$(grep -oP '\d+(?= numbered documents)' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null | head -1)
README_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null)

echo "Files on disk: $ACTUAL | Index: $INDEX | CLAUDE.md: $CLAUDE_TEXT (table: $CLAUDE_TABLE) | README.md: $README_TEXT (table: $README_TABLE)"

MISMATCH=false
for VAL in "$INDEX" "$CLAUDE_TEXT" "$CLAUDE_TABLE" "$README_TEXT" "$README_TABLE"; do
  [ "$VAL" != "$ACTUAL" ] && MISMATCH=true
done

if $MISMATCH; then
  echo "🔴 MISMATCH DETECTED - Must fix before committing"
else
  echo "🟢 All counts consistent ($ACTUAL documents)"
fi

# Check for unindexed files
for f in /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md; do
  BASENAME=$(basename "$f")
  if ! grep -q "$BASENAME" /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null; then
    echo "🔴 NOT INDEXED: $BASENAME - add to 01_SYSTEM_CANONICAL_INDEX.md, CLAUDE.md, and README.md"
  fi
done
```

**If any mismatch is found**, update ALL THREE files to match reality:
- `01_SYSTEM_CANONICAL_INDEX.md` - Add missing entries to Document Registry table, update count in heading
- `CLAUDE.md` - Add missing entries to Document Registry table, update count in text and Current Status
- `README.md` - Add missing entries to overview table, update count in text and structure diagram

### 2.4 Self-Update Check

Ask yourself these questions about what was done this session. **If YES to any**, update the command files and sync all locations:

1. **Did I add a new service or port?** → Add to health probes in uc-open Phase 4
2. **Did I add a new environment (staging, etc.)?** → Add environment check to uc-open Phase 4
3. **Did I add new infrastructure (database, cache, queue)?** → Add status check to uc-open
4. **Did I add new security concerns?** → Add to security scan in uc-close Phase 5
5. **Did I change git workflow (new branches, repos)?** → Update git checks in both commands
6. **Did I add new documentation files?** → Verify canonical index updated (Phase 2.3)
7. **Did I change monitoring or observability?** → Update relevant checks

If any command files are updated, **sync ALL four locations**:
- `/home/caue/.claude/commands/uc-open.md`
- `/home/caue/.claude/commands/uc-close.md`
- `/srv/unofficial/prod/app/.claude/commands/uc-open.md`
- `/srv/unofficial/prod/app/.claude/commands/uc-close.md`

If any skill files are created or updated, **sync between home and project**:
- `/home/caue/.claude/skills/<name>/SKILL.md`
- `/srv/unofficial/prod/app/.claude/skills/<name>/SKILL.md`

Also update `17_CLAUDE_CODE_PROMPTS.md` to reflect any structural changes.

### 2.5 Session Log Entry

**ALWAYS** add an entry to `16_SESSION_LOG.md`. This is mandatory regardless of what was done:

```markdown
## SESS-YYYY-MM-DD-N

**Date:** YYYY-MM-DD
**Focus Area:** [Primary focus of the session]

### Summary
[Brief description of what was accomplished]

### Changes Made
- [repo/file]: [what changed]

### Documentation Updated
- [list each doc file updated and why]

### Decisions Made
- [key decisions with rationale]

### Follow-up Items
- [ ] [tasks for future sessions]
```

## PHASE 3: GIT STATUS & COMMIT

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                      GIT STATUS CHECK                          ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  echo "📁 $repo:"
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  UNPUSHED=$(git -C "$repo" log @{u}..HEAD --oneline 2>/dev/null | wc -l)

  if [ "$CHANGES" -gt 0 ]; then
    echo "   ⚠️ $CHANGES uncommitted changes:"
    git -C "$repo" status --porcelain 2>/dev/null | head -10 | sed 's/^/   /'
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
echo "║                  DATABASE & QUEUE AT CLOSE                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

# MySQL check (prod)
MYSQL_CONTAINER=$(docker ps --filter "name=uc-api-db" --filter "status=running" --format "{{.Names}}" | head -1)
[ -n "$MYSQL_CONTAINER" ] && docker exec "$MYSQL_CONTAINER" mysqladmin ping -h localhost 2>/dev/null | grep -q "alive" && echo "🟢 MySQL (prod): Running" || echo "🔴 MySQL (prod): Check required"

# Staging MySQL check
STAGING_MYSQL=$(docker ps --filter "name=staging.*db" --filter "status=running" --format "{{.Names}}" | head -1)
[ -n "$STAGING_MYSQL" ] && echo "🟢 Staging DB: Running ($STAGING_MYSQL)" || echo "🟡 Staging DB: Not found"

# Queue check
REDIS_PASS=$(grep -r "REDIS_PASSWORD" /etc/uc/secrets/ 2>/dev/null | head -1 | cut -d= -f2)
FAILED=$(docker exec app-uc-redis-1 redis-cli -a "$REDIS_PASS" ZCARD "bull:social-events-queue:failed" 2>/dev/null || echo "?")
echo "📊 Queue failed jobs: $FAILED"
[ "${FAILED:-0}" -gt 10 ] && echo "⚠️ WARNING: High number of failed jobs"
```

## PHASE 5: SECURITY SCAN

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    SECURITY SCAN AT CLOSE                      ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
SECRETS_FOUND=0

for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||'); do
  # Check staged files
  STAGED=$(git -C "$repo" diff --cached --name-only 2>/dev/null)
  # Also check uncommitted changes
  CHANGED=$(git -C "$repo" diff --name-only 2>/dev/null)
  for file in $STAGED $CHANGED; do
    if [ -f "$repo/$file" ] && grep -qiE "(password|secret|api_key|private_key)\s*[:=]" "$repo/$file" 2>/dev/null; then
      echo "🔴 POTENTIAL SECRET: $repo/$file"
      SECRETS_FOUND=1
    fi
  done
done

[ "$SECRETS_FOUND" -eq 0 ] && echo "🟢 No secrets detected in changed files"
```

## PHASE 6: CROSS-DOCUMENT DEPENDENCY & CONTENT AUDIT

This phase catches **content-level drift** that structural checks miss: documents that require updates to other documents, stale status fields, and `Last Updated` headers that don't match actual file modifications.

### 6.1 Pending Cross-Document Update Requirements

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

### 6.2 Last Updated Header vs Git Modification

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

### 6.3 Stale Status Fields

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

**If cross-document pending items were found, verify they were addressed during this session or document them as follow-up items in the session log.**

## PHASE 7: DOCUMENTATION CONSISTENCY VALIDATION

Run the same audit as uc-open Phase 9 to confirm everything is consistent **AFTER** documentation updates but **BEFORE** final commit and **AFTER** cross-document audit (Phase 6). This is the gate that prevents leaving with stale docs.

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║         DOCUMENTATION CONSISTENCY VALIDATION                   ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

ISSUES=0

# Count actual numbered docs on disk
ACTUAL_COUNT=$(ls /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md 2>/dev/null | wc -l)
INDEX_COUNT=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null)
CLAUDE_TEXT=$(grep -oP '\d+(?= numbered files)' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null | head -1)
CLAUDE_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/CLAUDE.md 2>/dev/null)
README_TEXT=$(grep -oP '\d+(?= numbered documents)' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null | head -1)
README_TABLE=$(grep -cP '^\| \d+' /srv/unofficial/prod/app/uc-system-docs/README.md 2>/dev/null)

# Check all counts match disk
for LABEL_VAL in "Canonical_index:$INDEX_COUNT" "CLAUDE.md_text:$CLAUDE_TEXT" "CLAUDE.md_table:$CLAUDE_TABLE" "README.md_text:$README_TEXT" "README.md_table:$README_TABLE"; do
  LABEL=$(echo "$LABEL_VAL" | cut -d: -f1)
  VAL=$(echo "$LABEL_VAL" | cut -d: -f2)
  if [ "$VAL" != "$ACTUAL_COUNT" ]; then
    echo "🔴 MISMATCH: $LABEL says $VAL but $ACTUAL_COUNT files exist"
    ISSUES=$((ISSUES + 1))
  fi
done

# Check for unindexed files
for f in /srv/unofficial/prod/app/uc-system-docs/[0-9]*.md; do
  BASENAME=$(basename "$f")
  if ! grep -q "$BASENAME" /srv/unofficial/prod/app/uc-system-docs/01_SYSTEM_CANONICAL_INDEX.md 2>/dev/null; then
    echo "🔴 NOT INDEXED: $BASENAME"
    ISSUES=$((ISSUES + 1))
  fi
done

# Check command file sync
for CMD in uc-open.md uc-close.md; do
  HOME_HASH=$(md5sum /home/caue/.claude/commands/$CMD 2>/dev/null | awk '{print $1}')
  PROJ_HASH=$(md5sum /srv/unofficial/prod/app/.claude/commands/$CMD 2>/dev/null | awk '{print $1}')
  if [ "$HOME_HASH" != "$PROJ_HASH" ]; then
    echo "🔴 DESYNC: $CMD differs between home and project"
    ISSUES=$((ISSUES + 1))
  fi
done

# Check skill file sync (home vs project)
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
[ "$ISSUES" -eq 0 ] && echo "🟢 DOCUMENTATION VALIDATION PASSED" || echo "🔴 $ISSUES ISSUE(S) REMAIN - fix before leaving"
```

**If any issues remain, go back and fix them. Do NOT proceed to final verification with documentation issues.**

## PHASE 8: FINAL VERIFICATION

```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║                    FINAL SYNC VERIFICATION                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""

cd /srv/unofficial/prod/app
ALL_SYNCED=true
for repo in $(find . -maxdepth 2 -name ".git" -type d 2>/dev/null | sed 's|/\.git||' | sed 's|^\./||' | sort); do
  AHEAD=$(git -C "$repo" rev-list @{u}..HEAD --count 2>/dev/null || echo "0")
  CHANGES=$(git -C "$repo" status --porcelain 2>/dev/null | wc -l)
  BRANCH=$(git -C "$repo" branch --show-current 2>/dev/null)
  if [ "$AHEAD" -eq 0 ] && [ "$CHANGES" -eq 0 ]; then
    echo "✓ $repo ($BRANCH): Synced"
  else
    echo "⚠️ $repo ($BRANCH): NOT synced (uncommitted: $CHANGES, unpushed: $AHEAD)"
    ALL_SYNCED=false
  fi
done

echo ""
$ALL_SYNCED && echo "🟢 ALL REPOS SYNCED" || echo "🔴 NOT SYNCED: Uncommitted/unpushed changes exist"
```

## CLOSE SUMMARY

```
SESSION CLOSE SUMMARY
=====================
Date: [current date/time]
Session focus: [what was worked on]
Files modified: [list]
Commits pushed: [count per repo]

System status:
- Services: [running/healthy]
- Staging: [status]
- Database: [status]
- Queue failed jobs: [count]
- Secrets scan: [pass/fail]
- Git sync: [all synced / pending changes]
- Branches: [all on main / feature branches active]

Documentation status:
- Repos with changes documented: [yes/no - list each]
- Canonical index consistent: [yes/no]
- Session log updated: [yes/no]
- Self-update check: [no changes needed / updated + synced]
- Command files synced: [yes/no]
- Skill files synced: [yes/no]
- Cross-doc dependencies: [clean / N pending items]
- Consistency validation: [passed / N issues]

SAFE TO LEAVE: [YES / NO - list pending items]

Required for YES:
✓ All repos committed and pushed
✓ All documentation consistent (Phase 6 passed)
✓ CLAUDE.md updated for every changed repo
✓ Session log entry added
✓ Command files synced across all locations
✓ Skill files synced across all locations
✓ No secrets in changed files
✓ Database and queue healthy
```
