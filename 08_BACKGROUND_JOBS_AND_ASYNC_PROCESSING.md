# Background Jobs and Async Processing

**System:** Unofficial Communities
**Last Updated:** 2026-01-17
**Version:** 1.2.0

---

## 1. Job Registry

### 1.1 Job Overview

All background jobs in the system are managed through BullMQ with Redis as the backing store.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JOB PROCESSING ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   API       │    │  Webhooks   │    │  Scheduler  │             │
│  │  Service    │    │   Service   │    │   (Cron)    │             │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘             │
│         │                  │                  │                     │
│         └──────────────────┼──────────────────┘                     │
│                            │                                         │
│                            ▼                                         │
│              ┌───────────────────────┐                              │
│              │        Redis          │                              │
│              │    (BullMQ Queues)    │                              │
│              └───────────┬───────────┘                              │
│                          │                                           │
│         ┌────────────────┼────────────────┐                         │
│         │                │                │                         │
│         ▼                ▼                ▼                         │
│   ┌───────────┐    ┌───────────┐    ┌───────────┐                  │
│   │  Worker   │    │  Worker   │    │  Worker   │                  │
│   │  Pool 1   │    │  Pool 2   │    │  Pool 3   │                  │
│   └───────────┘    └───────────┘    └───────────┘                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Job Catalog

| Job Name | Queue | Trigger | Priority | Concurrency |
|----------|-------|---------|----------|-------------|
| `daily-missions-assignment` | gamification | Cron (00:01 America/Sao_Paulo) | High | 1 |
| `streak-check` | gamification | Cron (00:30 America/Sao_Paulo) | High | 1 |
| `weekly-ranking-reset` | gamification | Cron (Sun 00:00 America/Sao_Paulo) | Medium | 1 |
| `daily-feature-aggregation` | analytics | Cron (01:00 America/Sao_Paulo) | Medium | 1 |
| `community-graph-build` | analytics | Cron (04:15 America/Sao_Paulo) | Medium | 1 |
| `session-cleanup` | maintenance | Cron (03:00 America/Sao_Paulo) | Low | 1 |
| `data-anonymization` | compliance | Cron (02:00 America/Sao_Paulo) | Medium | 1 |
| `backup-database` | maintenance | Cron (03:00 America/Sao_Paulo) | Critical | 1 |
| `webhook-dispatch` | webhooks | Event-driven | High | 5 |
| `notification-send` | notifications | Event-driven | Medium | 10 |
| `xp-calculation` | gamification | Event-driven | High | 3 |
| `world-state-sync` | world | Event-driven | High | 2 |
| `age-revalidation-prompt` | age | Cron (08:00 America/Sao_Paulo) | Medium | 1 |
| `minor-detection-scan` | age | Cron (04:00 America/Sao_Paulo) | Medium | 1 |
| `age-data-cleanup` | age | Cron (Sun 02:00 America/Sao_Paulo) | Low | 1 |
| `minor-case-review` | age | Event-driven | High | 2 |

### 1.3 Job Definitions

```typescript
// Job Definition Interface
interface JobDefinition {
  name: string;
  queue: string;
  description: string;
  schedule?: string;           // Cron expression (if scheduled)
  trigger?: string;            // Event name (if event-driven)
  priority: 'critical' | 'high' | 'medium' | 'low';
  timeout_ms: number;
  retry: RetryPolicy;
  concurrency: number;
  data_schema: Record<string, unknown>;
}

// Example: Daily Missions Assignment
const dailyMissionsJob: JobDefinition = {
  name: 'daily-missions-assignment',
  queue: 'gamification',
  description: 'Assign daily missions to all active members across all communities',
  schedule: '1 0 * * *',       // 00:01 UTC daily
  priority: 'high',
  timeout_ms: 300000,          // 5 minutes
  retry: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 60000 }
  },
  concurrency: 1,
  data_schema: {
    type: 'object',
    properties: {
      community_ids: { type: 'array', items: { type: 'number' } },
      force: { type: 'boolean', default: false }
    }
  }
};
```

---

## 2. Schedules

### 2.1 Cron Schedule Registry

> **Timezone:** All scheduled jobs run in **America/Sao_Paulo** timezone to align with Brazilian user activity patterns.

| Schedule | Cron Expression | Timezone | Jobs |
|----------|-----------------|----------|------|
| Midnight Daily | `0 0 * * *` | America/Sao_Paulo | streak-check |
| Early Morning | `1 0 * * *` | America/Sao_Paulo | daily-missions-assignment |
| Post-Midnight | `0 1 * * *` | America/Sao_Paulo | daily-feature-aggregation |
| Pre-Dawn | `0 2 * * *` | America/Sao_Paulo | data-anonymization |
| Backup Window | `0 3 * * *` | America/Sao_Paulo | backup-database, session-cleanup |
| Graph Build | `15 4 * * *` | America/Sao_Paulo | community-graph-build |
| Weekly Reset | `0 0 * * 0` | America/Sao_Paulo | weekly-ranking-reset |
| Hourly Health | `0 * * * *` | America/Sao_Paulo | health-check |
| Every 5 Minutes | `*/5 * * * *` | America/Sao_Paulo | queue-metrics |

### 2.2 Scheduler Configuration

```typescript
// Scheduler Setup (BullMQ)
import { Queue, QueueScheduler } from 'bullmq';

const scheduledJobs = [
  {
    name: 'daily-missions-assignment',
    pattern: '1 0 * * *',
    data: {},
    opts: {
      jobId: 'daily-missions-singleton',  // Prevents duplicates
      removeOnComplete: { count: 10 },
      removeOnFail: { count: 50 }
    }
  },
  {
    name: 'streak-check',
    pattern: '30 0 * * *',
    data: {},
    opts: {
      jobId: 'streak-check-singleton',
      removeOnComplete: { count: 10 },
      removeOnFail: { count: 50 }
    }
  },
  {
    name: 'weekly-ranking-reset',
    pattern: '0 0 * * 0',
    data: {},
    opts: {
      jobId: 'ranking-reset-singleton',
      removeOnComplete: { count: 5 },
      removeOnFail: { count: 10 }
    }
  },
  {
    name: 'session-cleanup',
    pattern: '0 3 * * *',
    data: {},
    opts: {
      jobId: 'session-cleanup-singleton',
      removeOnComplete: { count: 10 },
      removeOnFail: { count: 50 }
    }
  },
  {
    name: 'backup-database',
    pattern: '0 3 * * *',
    data: {},
    opts: {
      jobId: 'backup-singleton',
      priority: 1,  // Highest priority
      removeOnComplete: { count: 7 },
      removeOnFail: { count: 30 }
    }
  }
];

async function initializeScheduler(queue: Queue): Promise<void> {
  // Clear existing repeatable jobs
  const existing = await queue.getRepeatableJobs();
  for (const job of existing) {
    await queue.removeRepeatableByKey(job.key);
  }

  // Register scheduled jobs
  for (const job of scheduledJobs) {
    await queue.add(job.name, job.data, {
      ...job.opts,
      repeat: { pattern: job.pattern, tz: 'UTC' }
    });
    console.log(`Scheduled job: ${job.name} at ${job.pattern}`);
  }
}
```

### 2.3 Schedule Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│              DAILY JOB SEQUENCE (America/Sao_Paulo)                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  00:00 ──┬── Day boundary (São Paulo time)                          │
│          │                                                           │
│  00:01 ──┼── daily-missions-assignment                              │
│          │   └── Must complete before user activity                 │
│          │                                                           │
│  00:30 ──┼── streak-check                                           │
│          │   └── Depends on: day boundary passed                    │
│          │   └── Updates: streaks, milestones                       │
│          │                                                           │
│  01:00 ──┼── daily-feature-aggregation                              │
│          │   └── Processes: previous day events                     │
│          │   └── Outputs: community_day_features                    │
│          │                                                           │
│  02:00 ──┼── data-anonymization                                     │
│          │   └── Depends on: aggregation complete                   │
│          │   └── Anonymizes: events > 90 days                       │
│          │                                                           │
│  03:00 ──┼── backup-database + session-cleanup                      │
│          │   └── Runs in low-traffic window                         │
│          │                                                           │
│  04:15 ──┴── community-graph-build                                  │
│              └── Depends on: daily-feature-aggregation complete     │
│              └── Outputs: community_graph_edges,                    │
│                           community_portal_assignments               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Retry Policies

### 3.1 Retry Strategy Types

| Strategy | Use Case | Configuration |
|----------|----------|---------------|
| Exponential Backoff | External API calls | delay: 1s, factor: 2, max: 5 retries |
| Fixed Delay | Database operations | delay: 5s, max: 3 retries |
| Immediate | Idempotent operations | delay: 0, max: 2 retries |
| None | Critical one-shot jobs | max: 0 (no retry) |

### 3.2 Retry Configuration

```typescript
interface RetryPolicy {
  attempts: number;
  backoff: {
    type: 'exponential' | 'fixed';
    delay: number;        // Initial delay in ms
    factor?: number;      // For exponential (default: 2)
    maxDelay?: number;    // Cap for exponential
  };
  retryOn?: string[];     // Error types to retry
  noRetryOn?: string[];   // Error types to not retry
}

// Default Retry Policies
const RETRY_POLICIES: Record<string, RetryPolicy> = {
  // For external API calls (WhatsApp, etc.)
  external_api: {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 1000,
      factor: 2,
      maxDelay: 60000
    },
    retryOn: ['ECONNRESET', 'ETIMEDOUT', 'RATE_LIMITED'],
    noRetryOn: ['INVALID_TOKEN', 'NOT_FOUND']
  },

  // For database operations
  database: {
    attempts: 3,
    backoff: {
      type: 'fixed',
      delay: 5000
    },
    retryOn: ['LOCK_TIMEOUT', 'DEADLOCK'],
    noRetryOn: ['CONSTRAINT_VIOLATION']
  },

  // For webhook dispatch
  webhook: {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 30000,
      factor: 2,
      maxDelay: 3600000  // Max 1 hour
    }
  },

  // For gamification calculations
  gamification: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
      factor: 2
    }
  },

  // For critical jobs (no retry, handle manually)
  critical: {
    attempts: 1,
    backoff: { type: 'fixed', delay: 0 }
  }
};
```

### 3.3 Retry Implementation

```typescript
import { Worker, Job } from 'bullmq';

// Worker with retry handling
const worker = new Worker('gamification', async (job: Job) => {
  try {
    await processJob(job);
  } catch (error) {
    // Determine if error is retryable
    const policy = RETRY_POLICIES[job.name] || RETRY_POLICIES.database;

    if (policy.noRetryOn?.includes(error.code)) {
      // Don't retry, move to dead letter
      await moveToDeadLetter(job, error);
      throw error;
    }

    if (job.attemptsMade < policy.attempts) {
      // Calculate backoff delay
      const delay = calculateBackoff(policy.backoff, job.attemptsMade);
      throw new Error(`Retrying in ${delay}ms: ${error.message}`);
    }

    // Max retries exceeded
    await moveToDeadLetter(job, error);
    throw error;
  }
}, {
  connection: redisConnection,
  concurrency: 3
});

function calculateBackoff(
  config: RetryPolicy['backoff'],
  attempt: number
): number {
  if (config.type === 'fixed') {
    return config.delay;
  }

  // Exponential backoff with jitter
  const factor = config.factor || 2;
  const delay = config.delay * Math.pow(factor, attempt);
  const jitter = Math.random() * 0.1 * delay; // 10% jitter
  const finalDelay = Math.min(delay + jitter, config.maxDelay || Infinity);

  return Math.floor(finalDelay);
}
```

---

## 4. Failure Handling

### 4.1 Failure Classification

| Failure Type | Handling | Alert Level | Auto-Retry |
|--------------|----------|-------------|------------|
| Transient | Retry with backoff | None | Yes |
| Rate Limit | Retry after cooldown | Warning | Yes |
| Invalid Input | Dead letter | Error | No |
| Resource Unavailable | Retry with escalation | Critical | Yes |
| Logic Error | Dead letter + alert | Error | No |
| System Failure | Dead letter + page | Critical | No |

### 4.2 Dead Letter Queue

```typescript
// Dead Letter Queue Configuration
interface DeadLetterConfig {
  queue_name: string;
  max_retention_days: number;
  alert_threshold: number;      // Alert if DLQ size exceeds
  auto_purge: boolean;
}

const DLQ_CONFIG: DeadLetterConfig = {
  queue_name: 'dead-letter',
  max_retention_days: 30,
  alert_threshold: 100,
  auto_purge: true
};

// Move failed job to DLQ
async function moveToDeadLetter(job: Job, error: Error): Promise<void> {
  const dlqData = {
    original_job: {
      id: job.id,
      name: job.name,
      queue: job.queueName,
      data: job.data,
      opts: job.opts
    },
    failure: {
      error_message: error.message,
      error_stack: error.stack,
      error_code: (error as any).code,
      attempts_made: job.attemptsMade,
      failed_at: new Date().toISOString()
    },
    metadata: {
      first_processed_at: job.processedOn,
      total_processing_time: Date.now() - (job.processedOn || Date.now())
    }
  };

  // Add to DLQ
  await deadLetterQueue.add(
    `dlq:${job.name}`,
    dlqData,
    {
      removeOnComplete: false,
      removeOnFail: false
    }
  );

  // Log failure
  await logJobFailure(job, error);

  // Check DLQ threshold
  const dlqSize = await deadLetterQueue.count();
  if (dlqSize >= DLQ_CONFIG.alert_threshold) {
    await alertOps('DLQ threshold exceeded', { size: dlqSize });
  }
}
```

### 4.3 Error Recovery Procedures

```typescript
// Manual job recovery
async function recoverDeadLetterJob(dlqJobId: string): Promise<void> {
  const dlqJob = await deadLetterQueue.getJob(dlqJobId);
  if (!dlqJob) throw new Error('DLQ job not found');

  const originalJob = dlqJob.data.original_job;

  // Re-queue to original queue
  const targetQueue = new Queue(originalJob.queue);
  await targetQueue.add(
    originalJob.name,
    originalJob.data,
    {
      ...originalJob.opts,
      attempts: 0,  // Reset attempts
      jobId: `recovered:${originalJob.id}`
    }
  );

  // Mark DLQ job as recovered
  await dlqJob.updateData({
    ...dlqJob.data,
    recovered_at: new Date().toISOString()
  });

  await dlqJob.moveToCompleted('recovered', true);
}

// Bulk recovery for specific job type
async function bulkRecoverJobs(
  jobName: string,
  filter?: (job: Job) => boolean
): Promise<number> {
  const jobs = await deadLetterQueue.getJobs(['waiting', 'delayed']);
  let recovered = 0;

  for (const job of jobs) {
    if (job.data.original_job.name !== jobName) continue;
    if (filter && !filter(job)) continue;

    await recoverDeadLetterJob(job.id!);
    recovered++;
  }

  return recovered;
}
```

### 4.4 Circuit Breaker

```typescript
// Circuit breaker for external services
class CircuitBreaker {
  private failures: number = 0;
  private lastFailure: Date | null = null;
  private state: 'closed' | 'open' | 'half-open' = 'closed';

  constructor(
    private threshold: number = 5,
    private resetTimeout: number = 60000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailure!.getTime() > this.resetTimeout) {
        this.state = 'half-open';
      } else {
        throw new Error('Circuit breaker is open');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailure = new Date();

    if (this.failures >= this.threshold) {
      this.state = 'open';
    }
  }
}

// Usage in job processor
const whatsappBreaker = new CircuitBreaker(5, 60000);

async function sendNotification(data: NotificationData): Promise<void> {
  await whatsappBreaker.execute(async () => {
    await whatsappApi.sendMessage(data);
  });
}
```

---

## 5. BullMQ Queues

### 5.1 Queue Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                       QUEUE STRUCTURE                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │                       REDIS                              │        │
│  │                                                          │        │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │        │
│  │  │ gamification │  │   webhooks   │  │  maintenance │  │        │
│  │  │    queue     │  │    queue     │  │    queue     │  │        │
│  │  │              │  │              │  │              │  │        │
│  │  │ ├─ waiting   │  │ ├─ waiting   │  │ ├─ waiting   │  │        │
│  │  │ ├─ active    │  │ ├─ active    │  │ ├─ active    │  │        │
│  │  │ ├─ delayed   │  │ ├─ delayed   │  │ ├─ delayed   │  │        │
│  │  │ ├─ completed │  │ ├─ completed │  │ ├─ completed │  │        │
│  │  │ └─ failed    │  │ └─ failed    │  │ └─ failed    │  │        │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │        │
│  │                                                          │        │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │        │
│  │  │  analytics   │  │ notifications│  │  dead-letter │  │        │
│  │  │    queue     │  │    queue     │  │    queue     │  │        │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │        │
│  │                                                          │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 Queue Configuration

```typescript
import { Queue, QueueOptions } from 'bullmq';

const QUEUE_CONFIG: Record<string, QueueOptions> = {
  gamification: {
    defaultJobOptions: {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
      removeOnComplete: { count: 100 },
      removeOnFail: { count: 500 }
    }
  },
  webhooks: {
    defaultJobOptions: {
      attempts: 5,
      backoff: { type: 'exponential', delay: 30000 },
      removeOnComplete: { count: 1000 },
      removeOnFail: { count: 5000 }
    }
  },
  notifications: {
    defaultJobOptions: {
      attempts: 3,
      backoff: { type: 'fixed', delay: 5000 },
      removeOnComplete: { count: 500 },
      removeOnFail: { count: 1000 }
    }
  },
  maintenance: {
    defaultJobOptions: {
      attempts: 1,
      removeOnComplete: { count: 50 },
      removeOnFail: { count: 100 }
    }
  },
  analytics: {
    defaultJobOptions: {
      attempts: 3,
      backoff: { type: 'fixed', delay: 60000 },
      removeOnComplete: { count: 30 },
      removeOnFail: { count: 100 }
    }
  },
  'dead-letter': {
    defaultJobOptions: {
      removeOnComplete: false,
      removeOnFail: false
    }
  }
};

// Queue initialization
const queues: Map<string, Queue> = new Map();

async function initializeQueues(): Promise<void> {
  for (const [name, config] of Object.entries(QUEUE_CONFIG)) {
    const queue = new Queue(name, {
      connection: redisConnection,
      ...config
    });

    queues.set(name, queue);
    console.log(`Queue initialized: ${name}`);
  }
}
```

### 5.3 Worker Configuration

```typescript
import { Worker, WorkerOptions } from 'bullmq';

interface WorkerConfig {
  queue: string;
  concurrency: number;
  processor: string;  // Path to processor module
  limiter?: {
    max: number;
    duration: number;
  };
}

const WORKER_CONFIG: WorkerConfig[] = [
  {
    queue: 'gamification',
    concurrency: 3,
    processor: './workers/gamification.processor.js',
    limiter: { max: 100, duration: 1000 }
  },
  {
    queue: 'webhooks',
    concurrency: 5,
    processor: './workers/webhooks.processor.js',
    limiter: { max: 50, duration: 1000 }
  },
  {
    queue: 'notifications',
    concurrency: 10,
    processor: './workers/notifications.processor.js',
    limiter: { max: 30, duration: 1000 }  // WhatsApp rate limit
  },
  {
    queue: 'maintenance',
    concurrency: 1,
    processor: './workers/maintenance.processor.js'
  },
  {
    queue: 'analytics',
    concurrency: 2,
    processor: './workers/analytics.processor.js'
  }
];

// Worker initialization
const workers: Map<string, Worker> = new Map();

async function initializeWorkers(): Promise<void> {
  for (const config of WORKER_CONFIG) {
    const workerOpts: WorkerOptions = {
      connection: redisConnection,
      concurrency: config.concurrency,
      limiter: config.limiter
    };

    const worker = new Worker(
      config.queue,
      config.processor,
      workerOpts
    );

    // Event handlers
    worker.on('completed', (job) => {
      logJobCompleted(job);
    });

    worker.on('failed', (job, error) => {
      logJobFailed(job, error);
    });

    workers.set(config.queue, worker);
    console.log(`Worker started: ${config.queue}`);
  }
}
```

### 5.4 Job Priority

```typescript
// Priority levels (lower = higher priority)
enum JobPriority {
  CRITICAL = 1,
  HIGH = 2,
  MEDIUM = 3,
  LOW = 4
}

// Priority assignment by job type
const JOB_PRIORITIES: Record<string, JobPriority> = {
  'backup-database': JobPriority.CRITICAL,
  'daily-missions-assignment': JobPriority.HIGH,
  'streak-check': JobPriority.HIGH,
  'webhook-dispatch': JobPriority.HIGH,
  'xp-calculation': JobPriority.HIGH,
  'notification-send': JobPriority.MEDIUM,
  'weekly-ranking-reset': JobPriority.MEDIUM,
  'event-aggregation': JobPriority.LOW,
  'session-cleanup': JobPriority.LOW,
  'data-anonymization': JobPriority.LOW
};

// Add job with priority
async function addJob(
  queueName: string,
  jobName: string,
  data: Record<string, unknown>,
  opts?: Partial<JobsOptions>
): Promise<Job> {
  const queue = queues.get(queueName);
  if (!queue) throw new Error(`Queue not found: ${queueName}`);

  const priority = JOB_PRIORITIES[jobName] || JobPriority.MEDIUM;

  return queue.add(jobName, data, {
    priority,
    ...opts
  });
}
```

---

## 6. Observability

### 6.1 Metrics Collection

```typescript
// Metrics to collect
interface QueueMetrics {
  queue_name: string;
  timestamp: Date;
  waiting: number;
  active: number;
  completed: number;
  failed: number;
  delayed: number;
  paused: boolean;
  throughput_per_minute: number;
  avg_processing_time_ms: number;
  error_rate: number;
}

// Metrics collector
async function collectQueueMetrics(): Promise<QueueMetrics[]> {
  const metrics: QueueMetrics[] = [];

  for (const [name, queue] of queues) {
    const counts = await queue.getJobCounts();
    const isPaused = await queue.isPaused();

    // Calculate throughput from completed jobs
    const completed = await queue.getCompleted(0, 100);
    const recentCompleted = completed.filter(
      j => j.finishedOn && Date.now() - j.finishedOn < 60000
    );

    // Calculate average processing time
    const processingTimes = recentCompleted
      .filter(j => j.processedOn && j.finishedOn)
      .map(j => j.finishedOn! - j.processedOn!);

    const avgTime = processingTimes.length > 0
      ? processingTimes.reduce((a, b) => a + b, 0) / processingTimes.length
      : 0;

    // Calculate error rate
    const failed = await queue.getFailed(0, 100);
    const recentFailed = failed.filter(
      j => j.finishedOn && Date.now() - j.finishedOn < 60000
    );
    const errorRate = recentCompleted.length + recentFailed.length > 0
      ? recentFailed.length / (recentCompleted.length + recentFailed.length)
      : 0;

    metrics.push({
      queue_name: name,
      timestamp: new Date(),
      waiting: counts.waiting,
      active: counts.active,
      completed: counts.completed,
      failed: counts.failed,
      delayed: counts.delayed,
      paused: isPaused,
      throughput_per_minute: recentCompleted.length,
      avg_processing_time_ms: avgTime,
      error_rate: errorRate
    });
  }

  return metrics;
}
```

### 6.2 Dashboard Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| Queue Depth | Jobs waiting in queue | > 1000 |
| Processing Time | Avg time to process job | > 30s |
| Error Rate | % of failed jobs | > 5% |
| Throughput | Jobs/minute | Varies by queue |
| DLQ Size | Dead letter queue count | > 100 |
| Worker Health | Workers responding | Any down |
| Stalled Jobs | Jobs stuck in active | > 0 for 5min |

### 6.3 Logging Standards

```typescript
// Job lifecycle logging
interface JobLog {
  level: 'info' | 'warn' | 'error';
  event: 'added' | 'started' | 'progress' | 'completed' | 'failed' | 'retrying';
  queue: string;
  job_id: string;
  job_name: string;
  timestamp: string;
  duration_ms?: number;
  attempt?: number;
  error?: string;
  metadata?: Record<string, unknown>;
}

function logJobEvent(log: JobLog): void {
  const message = `[${log.queue}] ${log.job_name}:${log.job_id} - ${log.event}`;

  const logData = {
    ...log,
    message,
    service: 'job-worker'
  };

  if (log.level === 'error') {
    console.error(JSON.stringify(logData));
  } else {
    console.log(JSON.stringify(logData));
  }
}

// Worker event handlers with logging
worker.on('active', (job) => {
  logJobEvent({
    level: 'info',
    event: 'started',
    queue: job.queueName,
    job_id: job.id!,
    job_name: job.name,
    timestamp: new Date().toISOString(),
    attempt: job.attemptsMade + 1
  });
});

worker.on('completed', (job) => {
  logJobEvent({
    level: 'info',
    event: 'completed',
    queue: job.queueName,
    job_id: job.id!,
    job_name: job.name,
    timestamp: new Date().toISOString(),
    duration_ms: job.finishedOn! - job.processedOn!
  });
});

worker.on('failed', (job, error) => {
  logJobEvent({
    level: 'error',
    event: 'failed',
    queue: job.queueName,
    job_id: job.id!,
    job_name: job.name,
    timestamp: new Date().toISOString(),
    attempt: job.attemptsMade,
    error: error.message
  });
});
```

### 6.4 Alerting Rules

```yaml
# Alert configuration
alerts:
  queue_depth_high:
    condition: waiting > 1000
    severity: warning
    channels: [slack]
    message: "Queue {{queue}} depth high: {{waiting}} jobs waiting"

  error_rate_high:
    condition: error_rate > 0.05
    severity: critical
    channels: [slack, pagerduty]
    message: "Queue {{queue}} error rate: {{error_rate}}%"

  dlq_threshold:
    condition: dlq_size > 100
    severity: warning
    channels: [slack]
    message: "Dead letter queue size: {{dlq_size}}"

  worker_down:
    condition: worker_count < expected_workers
    severity: critical
    channels: [slack, pagerduty]
    message: "Worker down: {{queue}} ({{worker_count}}/{{expected_workers}})"

  stalled_jobs:
    condition: stalled_count > 0 for 5m
    severity: warning
    channels: [slack]
    message: "Stalled jobs detected in {{queue}}: {{stalled_count}}"

  job_timeout:
    condition: active_duration > timeout_threshold
    severity: warning
    channels: [slack]
    message: "Job {{job_name}} running for {{duration}}s (timeout: {{timeout}}s)"
```

### 6.5 Health Check Endpoint

```typescript
// Job system health check
interface JobSystemHealth {
  status: 'healthy' | 'degraded' | 'unhealthy';
  queues: Record<string, QueueHealth>;
  workers: Record<string, WorkerHealth>;
  dlq_size: number;
  last_check: string;
}

interface QueueHealth {
  status: 'ok' | 'backlogged' | 'stalled';
  waiting: number;
  active: number;
  error_rate: number;
}

interface WorkerHealth {
  status: 'running' | 'paused' | 'down';
  concurrency: number;
  processing: number;
}

async function getJobSystemHealth(): Promise<JobSystemHealth> {
  const queueHealth: Record<string, QueueHealth> = {};
  const workerHealth: Record<string, WorkerHealth> = {};
  let overallStatus: 'healthy' | 'degraded' | 'unhealthy' = 'healthy';

  for (const [name, queue] of queues) {
    const counts = await queue.getJobCounts();
    const metrics = await getRecentMetrics(name);

    let status: QueueHealth['status'] = 'ok';
    if (counts.waiting > 1000) status = 'backlogged';
    if (counts.active > 0 && counts.waiting > 5000) status = 'stalled';

    queueHealth[name] = {
      status,
      waiting: counts.waiting,
      active: counts.active,
      error_rate: metrics.error_rate
    };

    if (status !== 'ok') overallStatus = 'degraded';
    if (metrics.error_rate > 0.1) overallStatus = 'unhealthy';
  }

  for (const [name, worker] of workers) {
    const isRunning = await worker.isRunning();
    const isPaused = await worker.isPaused();

    workerHealth[name] = {
      status: isPaused ? 'paused' : (isRunning ? 'running' : 'down'),
      concurrency: worker.opts.concurrency || 1,
      processing: 0  // Would need to track this
    };

    if (!isRunning) overallStatus = 'unhealthy';
  }

  const dlq = queues.get('dead-letter');
  const dlqSize = dlq ? await dlq.count() : 0;

  return {
    status: overallStatus,
    queues: queueHealth,
    workers: workerHealth,
    dlq_size: dlqSize,
    last_check: new Date().toISOString()
  };
}
```

---

*This document defines all background processing in the system. New jobs must be added to the registry before implementation.*

<!-- Last Updated: 2026-01-18 - Added age queue jobs (Section 1.2) -->
