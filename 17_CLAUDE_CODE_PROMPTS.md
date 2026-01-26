# Claude Code Session Prompts

**Last Updated:** 2026-01-25
**Version:** 2.0

These prompts are **self-maintaining** - they dynamically discover files, services, and configuration at runtime. The close prompt includes a self-update phase to ensure all command files and this reference document stay current.

## Authoritative Source

The session commands are defined as Claude Code skill files. **The command files are the authoritative source.** This document is a reference that describes their structure and criteria.

| Location | Purpose |
|----------|---------|
| `/home/caue/.claude/commands/uc-open.md` | Open session command (executes via `/uc-open`) |
| `/home/caue/.claude/commands/uc-close.md` | Close session command (executes via `/uc-close`) |
| `/srv/unofficial/prod/app/.claude/commands/uc-open.md` | Project-level copy (must be identical to home) |
| `/srv/unofficial/prod/app/.claude/commands/uc-close.md` | Project-level copy (must be identical to home) |
| This file (`17_CLAUDE_CODE_PROMPTS.md`) | Reference documentation |

All four command files must always be in sync. The uc-close command (Phase 2.4 and Phase 6) enforces this.

---

## 1. Open Session Command (`/uc-open`)

**10 Phases:**

| Phase | Name | Purpose |
|-------|------|---------|
| 1 | Discover and Read All Documentation | Find and read all CLAUDE.md, README.md, and system docs |
| 2 | System Health Check | Disk, memory, load |
| 3 | Docker Services Status | Container status, restart detection |
| 4 | Service Health Probes | HTTP health checks (prod + staging), Redis |
| 5 | Database Status | MySQL connectivity, Prisma migration status |
| 6 | Queue Status | BullMQ waiting/active/failed counts |
| 7 | Git Status | Uncommitted changes, unpushed commits, branch warnings |
| 8 | Backup Status | Last backup date, success verification |
| 9 | Documentation Consistency Audit | **Count validation, index sync, staleness, command sync** |
| 10 | Summary | GO / CAUTION / NO-GO decision |

### Phase 9: Documentation Consistency Audit (Key Phase)

This phase prevents documentation drift by checking:

1. **Count validation**: Files on disk vs canonical index vs CLAUDE.md vs README.md (text and table rows)
2. **Index completeness**: Every numbered file on disk must appear in the canonical index
3. **Staleness**: Documents with "Last Updated/Reviewed" older than 7 days are flagged
4. **Command sync**: Home-level and project-level command files must have matching hashes

**If any issues are found, they must be fixed before starting work.**

---

## 2. Close Session Command (`/uc-close`)

**7 Phases:**

| Phase | Name | Purpose |
|-------|------|---------|
| 1 | Pre-Close System Check | Quick health snapshot |
| 2 | Mandatory Documentation Updates | **Enforced doc updates for all changes** |
| 3 | Git Status & Commit | Review changes, commit, push |
| 4 | Database & Queue Check | MySQL (prod + staging), queue failed jobs |
| 5 | Security Scan | Secrets detection in staged/changed files |
| 6 | Documentation Consistency Validation | **Same audit as open Phase 9 - gate before leaving** |
| 7 | Final Verification | All repos synced check |

### Phase 2: Mandatory Documentation Updates (Key Phase)

This phase has 5 enforced sub-steps:

| Step | Name | Rule |
|------|------|------|
| 2.1 | Identify Changed Repos | List all repos with uncommitted, today's commits, or unpushed changes |
| 2.2 | Update Docs for Changed Repos | **MUST** update CLAUDE.md (Current Status + Last Reviewed) for every repo with code changes. Update README.md if structure/setup changed. |
| 2.3 | Verify Canonical Index | Count files on disk vs all index references. Fix mismatches in 01_, CLAUDE.md, and README.md |
| 2.4 | Self-Update Check | If infrastructure/services/workflow changed, update command files and sync all 4 locations + this doc |
| 2.5 | Session Log Entry | **ALWAYS** add entry to 16_SESSION_LOG.md with changes, docs updated, decisions, follow-ups |

### Phase 6: Documentation Consistency Validation (Gate)

Runs the same audit script as uc-open Phase 9. This is the final gate:
- All counts must match
- All files must be indexed
- Command files must be synced across locations
- **If any issue remains, go back and fix before proceeding to Phase 7**

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

# Command files
/home/caue/.claude/commands/uc-open.md
/home/caue/.claude/commands/uc-close.md
/srv/unofficial/prod/app/.claude/commands/uc-open.md
/srv/unofficial/prod/app/.claude/commands/uc-close.md
```

### Service Discovery
```bash
# Running services and ports
docker ps --format "table {{.Names}}\t{{.Ports}}"

# Health check any service
curl -s http://localhost:[PORT]/health
```

### GO / NO-GO Criteria (Open)

| Check | GO | CAUTION | NO-GO |
|-------|-----|---------|-------|
| Database | Connected | - | Connection failed |
| API health | Returns 200 | - | Down |
| Staging API | Returns 200 | Down (warning) | - |
| Redis | PONG | - | Connection failed |
| Queue failed jobs | 0 | 1-10 | >10 |
| Container restarts | 0 | - | Any restarting |
| Branches | All on main | Feature branches | - |
| Doc consistency | All match | Stale docs (>7d) | Count mismatches |
| Command sync | Hashes match | - | Files differ |

### SAFE TO LEAVE Criteria (Close)

| Check | SAFE | NOT SAFE |
|-------|------|----------|
| All repos synced | Committed + pushed | Uncommitted/unpushed changes |
| Changed repos documented | CLAUDE.md updated | Missing updates |
| Canonical index | Counts match | Mismatches |
| Session log | Entry added | Missing |
| Command files | Synced (4 locations) | Divergent |
| Failed jobs | <10 | >=10 |
| Database | Connected | Disconnected |
| Containers | All running | Any restarting |
| Secrets scan | None detected | Secrets in changed files |
| Self-update | Reviewed + updated if needed | New features uncaptured |

---

*The command files (`/home/caue/.claude/commands/`) are the authoritative source. This document is a reference. When commands are updated, this file must also be updated to match (enforced by uc-close Phase 2.4).*

<!-- Last Updated: 2026-01-25 - v2.0: Rewrote to match new 10-phase open and 7-phase close commands with documentation consistency enforcement -->
