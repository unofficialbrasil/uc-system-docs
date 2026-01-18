# Deployment and Rollback Runbook

**System:** Unofficial Communities
**Last Updated:** 2026-01-14
**Version:** 1.0.0

---

## 1. Deployment Overview

### 1.1 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PIPELINE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐     │
│  │  GitHub  │───▶│  Build   │───▶│  Test    │───▶│  Deploy  │     │
│  │  Push    │    │  Images  │    │  Suite   │    │  to VPS  │     │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘     │
│                                                                      │
│  Manual deployment via SSH to production VPS                         │
│  Server: 72.61.134.35                                               │
│  User: caue                                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Inventory

| Service | Container | Port | Build Context |
|---------|-----------|------|---------------|
| unofficial-communities | uc-frontend | 3000 | ./unofficial-communities |
| uc-api | uc-api | 3001 | ./uc-api |
| uc-webhooks | uc-webhooks | 3002 | ./uc-webhooks |
| uc-world | uc-world | 2567 | ./uc-world |
| MySQL | mysql | 3306 | Official image |
| Redis | redis | 6379 | Official image |

### 1.3 Deployment Frequency

| Environment | Frequency | Approval |
|-------------|-----------|----------|
| Development | On commit | Automatic |
| Staging | On merge to staging | Manual |
| Production | Scheduled/Manual | Manual (owner) |

---

## 2. Pre-Deployment Checklist

### 2.1 Before Every Deployment

```bash
# 1. Check current system status
ssh caue@72.61.134.35

# 2. Verify all containers are running
docker ps

# 3. Check disk space (must be < 80%)
df -h

# 4. Check memory (must be < 85%)
free -h

# 5. Check current git status in each repo
cd /srv/unofficial/prod/app
for repo in unofficial-communities uc-api uc-webhooks uc-world; do
  echo "=== $repo ==="
  cd $repo && git status && cd ..
done

# 6. Verify database backup exists (within last 24h)
ls -la /srv/unofficial/backups/

# 7. Check error logs for critical issues
docker logs uc-api --tail 100 2>&1 | grep -i error
```

### 2.2 Pre-Deployment Verification Matrix

| Check | Command | Expected Result | Abort If |
|-------|---------|-----------------|----------|
| Disk Space | `df -h /` | < 80% used | > 90% used |
| Memory | `free -h` | < 85% used | > 95% used |
| All Containers | `docker ps` | All running | Any down |
| Database Backup | `ls /srv/unofficial/backups/` | < 24h old | > 48h old |
| Error Rate | Log check | < 1% errors | > 5% errors |
| Load Average | `uptime` | < 2.0 | > 5.0 |

---

## 3. Deployment Procedure

### 3.1 Standard Deployment (Single Service)

```bash
#!/bin/bash
# deploy-service.sh <service-name>
# Example: ./deploy-service.sh uc-api

SERVICE=$1
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/srv/unofficial/logs/deploy_${SERVICE}_${TIMESTAMP}.log"

echo "Starting deployment of ${SERVICE} at ${TIMESTAMP}" | tee $LOG_FILE

# Step 1: Pull latest code
cd /srv/unofficial/prod/app/${SERVICE}
echo "Pulling latest code..." | tee -a $LOG_FILE
git pull origin main 2>&1 | tee -a $LOG_FILE

# Step 2: Build new image
echo "Building Docker image..." | tee -a $LOG_FILE
docker compose build ${SERVICE} 2>&1 | tee -a $LOG_FILE

# Step 3: Stop old container
echo "Stopping old container..." | tee -a $LOG_FILE
docker compose stop ${SERVICE} 2>&1 | tee -a $LOG_FILE

# Step 4: Start new container
echo "Starting new container..." | tee -a $LOG_FILE
docker compose up -d ${SERVICE} 2>&1 | tee -a $LOG_FILE

# Step 5: Wait for health check
echo "Waiting for health check..." | tee -a $LOG_FILE
sleep 10

# Step 6: Verify deployment
if docker ps | grep -q "${SERVICE}.*Up"; then
  echo "SUCCESS: ${SERVICE} deployed successfully" | tee -a $LOG_FILE
else
  echo "FAILURE: ${SERVICE} failed to start" | tee -a $LOG_FILE
  exit 1
fi
```

### 3.2 Full Stack Deployment

```bash
#!/bin/bash
# deploy-all.sh
# Full deployment of all services

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/srv/unofficial/logs/deploy_full_${TIMESTAMP}.log"

echo "=== FULL STACK DEPLOYMENT ===" | tee $LOG_FILE
echo "Started at: $(date)" | tee -a $LOG_FILE

# Pre-deployment backup
echo "Creating pre-deployment backup..." | tee -a $LOG_FILE
/srv/unofficial/scripts/backup.sh 2>&1 | tee -a $LOG_FILE

# Deploy order matters - dependencies first
SERVICES=("mysql" "redis" "uc-api" "uc-webhooks" "uc-world" "uc-frontend")

cd /srv/unofficial/prod/app

for service in "${SERVICES[@]}"; do
  echo "Deploying ${service}..." | tee -a $LOG_FILE

  # Pull if it's a custom service (not mysql/redis)
  if [[ "$service" != "mysql" && "$service" != "redis" ]]; then
    repo_name=${service/uc-/uc-}
    if [[ "$service" == "uc-frontend" ]]; then
      repo_name="unofficial-communities"
    fi
    cd ${repo_name}
    git pull origin main 2>&1 | tee -a $LOG_FILE
    cd ..
  fi

  docker compose build ${service} 2>&1 | tee -a $LOG_FILE
  docker compose up -d ${service} 2>&1 | tee -a $LOG_FILE

  # Wait between services
  sleep 5
done

# Verify all services
echo "Verifying deployment..." | tee -a $LOG_FILE
/srv/unofficial/scripts/health-check.sh 2>&1 | tee -a $LOG_FILE

echo "Completed at: $(date)" | tee -a $LOG_FILE
```

### 3.3 Zero-Downtime Deployment

```bash
#!/bin/bash
# deploy-zero-downtime.sh <service>
# Uses blue-green deployment pattern

SERVICE=$1
CONTAINER_NAME="${SERVICE}"
NEW_CONTAINER="${SERVICE}-new"

echo "Zero-downtime deployment for ${SERVICE}"

# Step 1: Build new image
docker compose build ${SERVICE}

# Step 2: Start new container on different port
docker run -d --name ${NEW_CONTAINER} \
  --network uc-network \
  -e PORT=3099 \
  ${SERVICE}:latest

# Step 3: Wait for new container to be healthy
echo "Waiting for new container health..."
for i in {1..30}; do
  if docker exec ${NEW_CONTAINER} curl -s http://localhost:3099/health > /dev/null; then
    echo "New container is healthy"
    break
  fi
  sleep 2
done

# Step 4: Switch traffic (update nginx/traefik)
# (Implementation depends on reverse proxy setup)

# Step 5: Stop old container
docker stop ${CONTAINER_NAME}
docker rm ${CONTAINER_NAME}

# Step 6: Rename new container
docker rename ${NEW_CONTAINER} ${CONTAINER_NAME}

echo "Deployment complete"
```

---

## 4. Rollback Procedure

### 4.1 Quick Rollback (Container)

```bash
#!/bin/bash
# rollback-quick.sh <service>
# Rolls back to previous Docker image

SERVICE=$1

echo "Quick rollback for ${SERVICE}"

# Get previous image
PREVIOUS_IMAGE=$(docker images ${SERVICE} --format "{{.ID}}" | sed -n '2p')

if [ -z "$PREVIOUS_IMAGE" ]; then
  echo "ERROR: No previous image found for ${SERVICE}"
  exit 1
fi

# Stop current container
docker compose stop ${SERVICE}

# Tag previous image as latest
docker tag ${PREVIOUS_IMAGE} ${SERVICE}:latest

# Start with previous image
docker compose up -d ${SERVICE}

echo "Rolled back to image: ${PREVIOUS_IMAGE}"
```

### 4.2 Git Rollback (Code)

```bash
#!/bin/bash
# rollback-git.sh <service> <commit-hash>
# Rolls back to specific git commit

SERVICE=$1
COMMIT=$2

echo "Git rollback for ${SERVICE} to ${COMMIT}"

cd /srv/unofficial/prod/app/${SERVICE}

# Store current HEAD for potential re-rollback
CURRENT_HEAD=$(git rev-parse HEAD)
echo "Current HEAD: ${CURRENT_HEAD}"

# Checkout specific commit
git checkout ${COMMIT}

# Rebuild and deploy
docker compose build ${SERVICE}
docker compose up -d ${SERVICE}

echo "Rolled back to commit: ${COMMIT}"
echo "Previous HEAD was: ${CURRENT_HEAD}"
```

### 4.3 Database Rollback

```bash
#!/bin/bash
# rollback-database.sh <backup-file>
# Restores database from backup

BACKUP_FILE=$1

if [ ! -f "$BACKUP_FILE" ]; then
  echo "ERROR: Backup file not found: ${BACKUP_FILE}"
  exit 1
fi

echo "WARNING: This will replace all database data!"
read -p "Are you sure? (type 'yes' to confirm): " CONFIRM

if [ "$CONFIRM" != "yes" ]; then
  echo "Aborted"
  exit 1
fi

# Stop API to prevent writes
docker compose stop uc-api uc-webhooks uc-world

# Restore backup
echo "Restoring database from ${BACKUP_FILE}..."
docker exec -i mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} unofficial < ${BACKUP_FILE}

# Restart services
docker compose up -d uc-api uc-webhooks uc-world

echo "Database restored from: ${BACKUP_FILE}"
```

### 4.4 Full System Rollback

```bash
#!/bin/bash
# rollback-full.sh <timestamp>
# Rolls back entire system to specific point in time

TIMESTAMP=$1
BACKUP_DIR="/srv/unofficial/backups/${TIMESTAMP}"

if [ ! -d "$BACKUP_DIR" ]; then
  echo "ERROR: Backup directory not found: ${BACKUP_DIR}"
  echo "Available backups:"
  ls /srv/unofficial/backups/
  exit 1
fi

echo "=== FULL SYSTEM ROLLBACK TO ${TIMESTAMP} ==="
echo "This will:"
echo "  1. Stop all services"
echo "  2. Restore database"
echo "  3. Restore code to backup state"
echo "  4. Restart all services"
echo ""
read -p "Are you sure? (type 'ROLLBACK' to confirm): " CONFIRM

if [ "$CONFIRM" != "ROLLBACK" ]; then
  echo "Aborted"
  exit 1
fi

# Step 1: Stop all services
echo "Stopping all services..."
docker compose down

# Step 2: Restore database
echo "Restoring database..."
mysql -u root -p${MYSQL_ROOT_PASSWORD} unofficial < ${BACKUP_DIR}/database.sql

# Step 3: Restore code (if version controlled)
for service in unofficial-communities uc-api uc-webhooks uc-world; do
  if [ -f "${BACKUP_DIR}/${service}_commit.txt" ]; then
    COMMIT=$(cat ${BACKUP_DIR}/${service}_commit.txt)
    cd /srv/unofficial/prod/app/${service}
    git checkout ${COMMIT}
  fi
done

# Step 4: Rebuild and restart
cd /srv/unofficial/prod/app
docker compose build
docker compose up -d

echo "Full rollback complete"
```

---

## 5. Health Checks

### 5.1 Service Health Endpoints

| Service | Endpoint | Expected Response | Timeout |
|---------|----------|-------------------|---------|
| uc-frontend | `GET /` | 200 OK | 5s |
| uc-api | `GET /health` | `{"status":"ok"}` | 3s |
| uc-webhooks | `GET /health` | `{"status":"ok"}` | 3s |
| uc-world | `GET /health` | `{"status":"ok"}` | 3s |
| MySQL | TCP 3306 | Connection success | 5s |
| Redis | `PING` | `PONG` | 2s |

### 5.2 Health Check Script

```bash
#!/bin/bash
# health-check.sh
# Comprehensive health check for all services

echo "=== SYSTEM HEALTH CHECK ==="
echo "Timestamp: $(date)"
echo ""

ERRORS=0

# Frontend
echo -n "Frontend (3000): "
if curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 | grep -q "200"; then
  echo "OK"
else
  echo "FAIL"
  ((ERRORS++))
fi

# API
echo -n "API (3001): "
API_HEALTH=$(curl -s http://localhost:3001/health)
if echo "$API_HEALTH" | grep -q '"status":"ok"'; then
  echo "OK"
else
  echo "FAIL - $API_HEALTH"
  ((ERRORS++))
fi

# Webhooks
echo -n "Webhooks (3002): "
WEBHOOK_HEALTH=$(curl -s http://localhost:3002/health)
if echo "$WEBHOOK_HEALTH" | grep -q '"status":"ok"'; then
  echo "OK"
else
  echo "FAIL - $WEBHOOK_HEALTH"
  ((ERRORS++))
fi

# World Server
echo -n "World (2567): "
WORLD_HEALTH=$(curl -s http://localhost:2567/health)
if echo "$WORLD_HEALTH" | grep -q '"status":"ok"'; then
  echo "OK"
else
  echo "FAIL - $WORLD_HEALTH"
  ((ERRORS++))
fi

# MySQL
echo -n "MySQL (3306): "
if docker exec mysql mysqladmin ping -h localhost -u root -p${MYSQL_ROOT_PASSWORD} 2>/dev/null | grep -q "alive"; then
  echo "OK"
else
  echo "FAIL"
  ((ERRORS++))
fi

# Redis
echo -n "Redis (6379): "
if docker exec redis redis-cli ping 2>/dev/null | grep -q "PONG"; then
  echo "OK"
else
  echo "FAIL"
  ((ERRORS++))
fi

echo ""
echo "=== SUMMARY ==="
if [ $ERRORS -eq 0 ]; then
  echo "All services healthy"
  exit 0
else
  echo "ERRORS: $ERRORS services unhealthy"
  exit 1
fi
```

### 5.3 Deep Health Check

```bash
#!/bin/bash
# health-check-deep.sh
# Extended health check including dependencies

echo "=== DEEP HEALTH CHECK ==="

# Basic checks
./health-check.sh
BASIC_STATUS=$?

# Database connectivity from API
echo ""
echo "=== DATABASE CONNECTIVITY ==="
echo -n "API -> MySQL: "
if docker exec uc-api curl -s http://localhost:3001/health/db | grep -q "ok"; then
  echo "OK"
else
  echo "FAIL"
fi

# Redis connectivity
echo ""
echo "=== REDIS CONNECTIVITY ==="
echo -n "API -> Redis: "
if docker exec uc-api curl -s http://localhost:3001/health/redis | grep -q "ok"; then
  echo "OK"
else
  echo "FAIL"
fi

# Queue health
echo ""
echo "=== QUEUE HEALTH ==="
echo -n "BullMQ Queues: "
QUEUE_STATUS=$(docker exec uc-api curl -s http://localhost:3001/health/queues)
if echo "$QUEUE_STATUS" | grep -q '"healthy":true'; then
  echo "OK"
else
  echo "WARN - $QUEUE_STATUS"
fi

# Disk space
echo ""
echo "=== RESOURCE CHECK ==="
echo -n "Disk Space: "
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ $DISK_USAGE -lt 80 ]; then
  echo "OK (${DISK_USAGE}%)"
else
  echo "WARN (${DISK_USAGE}%)"
fi

# Memory
echo -n "Memory: "
MEM_USAGE=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')
if [ $MEM_USAGE -lt 85 ]; then
  echo "OK (${MEM_USAGE}%)"
else
  echo "WARN (${MEM_USAGE}%)"
fi

# Container health
echo ""
echo "=== CONTAINER STATUS ==="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

## 6. Incident Runbook

### 6.1 Incident Classification

| Severity | Description | Response Time | Escalation |
|----------|-------------|---------------|------------|
| P1 - Critical | System down, data loss risk | Immediate | Owner + All |
| P2 - High | Major feature unavailable | 30 minutes | Owner |
| P3 - Medium | Degraded performance | 4 hours | Team |
| P4 - Low | Minor issues | 24 hours | Team |

### 6.2 Common Incidents

#### 6.2.1 API Unresponsive

```bash
# Symptoms: 502/503 errors, API health check failing

# Step 1: Check container status
docker ps | grep uc-api
docker logs uc-api --tail 50

# Step 2: Check if container is OOM killed
docker inspect uc-api | grep -A5 "State"

# Step 3: Restart container
docker compose restart uc-api

# Step 4: If restart fails, check port conflicts
netstat -tlnp | grep 3001

# Step 5: Full rebuild if necessary
docker compose build uc-api
docker compose up -d uc-api
```

#### 6.2.2 Database Connection Errors

```bash
# Symptoms: "Connection refused" or "Too many connections"

# Step 1: Check MySQL container
docker ps | grep mysql
docker logs mysql --tail 50

# Step 2: Check connection count
docker exec mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "SHOW STATUS LIKE 'Threads_connected';"

# Step 3: Kill idle connections if needed
docker exec mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "SHOW PROCESSLIST;" | grep Sleep

# Step 4: Restart MySQL if necessary
docker compose restart mysql
# Wait for startup
sleep 30
# Restart dependent services
docker compose restart uc-api uc-webhooks uc-world
```

#### 6.2.3 Redis Memory Issues

```bash
# Symptoms: Redis OOM, slow responses

# Step 1: Check Redis memory
docker exec redis redis-cli INFO memory | grep used_memory_human

# Step 2: Check for large keys
docker exec redis redis-cli --bigkeys

# Step 3: Clear expired sessions if needed
docker exec redis redis-cli KEYS "session:*" | head -100

# Step 4: If critical, flush non-essential data
# CAUTION: This clears sessions
docker exec redis redis-cli FLUSHDB

# Step 5: Restart Redis
docker compose restart redis
```

#### 6.2.4 High CPU/Load

```bash
# Symptoms: Slow responses, high load average

# Step 1: Identify resource hogs
docker stats --no-stream

# Step 2: Check system processes
top -bn1 | head -20

# Step 3: Identify problematic queries
docker exec mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "SHOW FULL PROCESSLIST;"

# Step 4: If UC World causing issues
docker compose restart uc-world

# Step 5: If persistent, scale down non-essential services
docker compose stop uc-world  # Temporary measure
```

#### 6.2.5 Disk Space Critical

```bash
# Symptoms: Disk > 90% full

# Step 1: Check disk usage
df -h
du -sh /srv/unofficial/*

# Step 2: Clean Docker artifacts
docker system prune -f
docker volume prune -f

# Step 3: Remove old logs
find /srv/unofficial/logs -name "*.log" -mtime +7 -delete

# Step 4: Remove old backups (keep last 7)
cd /srv/unofficial/backups
ls -t | tail -n +8 | xargs rm -rf

# Step 5: Clean MySQL binlogs if enabled
docker exec mysql mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "PURGE BINARY LOGS BEFORE DATE(NOW() - INTERVAL 7 DAY);"
```

### 6.3 Escalation Procedures

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ESCALATION MATRIX                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Incident Detected                                                   │
│        │                                                             │
│        ▼                                                             │
│  ┌──────────────┐                                                   │
│  │ Initial Triage│ (5 minutes)                                      │
│  │ - Check logs  │                                                   │
│  │ - Run health  │                                                   │
│  └──────┬───────┘                                                   │
│         │                                                            │
│         ├─── Can fix immediately? ──── Yes ───▶ Fix & Document      │
│         │                                                            │
│         No                                                           │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                   │
│  │ Classify     │                                                   │
│  │ Severity     │                                                   │
│  └──────┬───────┘                                                   │
│         │                                                            │
│    P1/P2? ────────────────────────────────────────┐                 │
│         │                                          │                 │
│        Yes                                         No (P3/P4)       │
│         │                                          │                 │
│         ▼                                          ▼                 │
│  ┌──────────────┐                          ┌──────────────┐        │
│  │ Alert Owner  │                          │ Create Ticket │        │
│  │ Immediately  │                          │ Schedule Fix  │        │
│  └──────┬───────┘                          └──────────────┘        │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                   │
│  │ War Room     │ (if P1)                                           │
│  │ - All hands  │                                                   │
│  │ - Status page│                                                   │
│  └──────────────┘                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.4 Post-Incident Checklist

```markdown
## Post-Incident Report Template

**Incident ID:** INC-YYYY-MM-DD-XXX
**Severity:** P1/P2/P3/P4
**Duration:** HH:MM start to HH:MM end (total: X hours)
**Services Affected:** [list]

### Timeline
- HH:MM - Incident detected by [how]
- HH:MM - Initial response by [who]
- HH:MM - Root cause identified
- HH:MM - Fix deployed
- HH:MM - Service restored
- HH:MM - Incident closed

### Root Cause
[Detailed explanation]

### Impact
- Users affected: [number/percentage]
- Features unavailable: [list]
- Data impact: [none/partial/full]

### Resolution
[What was done to fix it]

### Action Items
- [ ] Immediate fix applied
- [ ] Root cause addressed
- [ ] Monitoring improved
- [ ] Documentation updated
- [ ] Team debriefed

### Lessons Learned
[What we learned and how to prevent recurrence]
```

---

## 7. Monitoring and Alerts

### 7.1 Key Metrics to Monitor

| Metric | Warning | Critical | Check Interval |
|--------|---------|----------|----------------|
| API Response Time | > 500ms | > 2s | 1 min |
| Error Rate | > 1% | > 5% | 1 min |
| CPU Usage | > 70% | > 90% | 1 min |
| Memory Usage | > 80% | > 95% | 1 min |
| Disk Usage | > 70% | > 90% | 5 min |
| Queue Depth | > 1000 | > 5000 | 1 min |
| Active Connections | > 80% max | > 95% max | 1 min |

### 7.2 Alerting Configuration

```yaml
# alerts.yml
alerts:
  - name: api_down
    condition: http_status != 200
    for: 2m
    severity: critical
    message: "API is not responding"

  - name: high_error_rate
    condition: error_rate > 5%
    for: 5m
    severity: critical
    message: "Error rate above 5%: {{ $value }}%"

  - name: disk_space_critical
    condition: disk_usage > 90%
    for: 5m
    severity: critical
    message: "Disk usage critical: {{ $value }}%"

  - name: memory_high
    condition: memory_usage > 85%
    for: 10m
    severity: warning
    message: "Memory usage high: {{ $value }}%"

  - name: database_connections_high
    condition: db_connections > 80%
    for: 5m
    severity: warning
    message: "Database connections at {{ $value }}%"
```

---

*This runbook must be followed for all deployments. Update after each incident.*

<!-- Last Reviewed: 2026-01-18 - No updates needed -->
