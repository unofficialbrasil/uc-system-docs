# UC System Docs

System-wide documentation for Unofficial Communities platform - the authoritative source of truth for architecture, specifications, and governance.

## Overview

This repository contains 18 numbered documents covering all aspects of the Unofficial Communities platform:

| # | Document | Purpose |
|---|----------|---------|
| 01 | System Canonical Index | Entry point, document registry |
| 02 | Product Blueprint | Vision, principles, behavioral science |
| 03 | Technical Architecture | System architecture and integrations |
| 04 | System Algorithm & Lifecycle Spec | Core behavior specification |
| 05 | User State Machine | User flows and state transitions |
| 06 | Data Architecture & Lifecycle | Data handling and lifecycle |
| 07 | Event & Analytics Spec | Events and analytics |
| 08 | Background Jobs & Async Processing | Async processing and queues |
| 09 | Security & Authorization Spec | Security specification |
| 10 | Deployment & Rollback Runbook | Deployment procedures |
| 11 | Environment & Configuration Registry | Configuration registry |
| 12 | Compliance & Data Protection | LGPD/GDPR compliance |
| 13 | Risk Register | Risk management |
| 14 | Assumption Register | Documented assumptions |
| 15 | Decision Log | Decision history |
| 16 | Session Log | Development session history |
| 17 | Claude Code Prompts | Session prompts for Claude Code |
| 18 | Behavioral Science Framework | Research reference |

## Documentation Structure

```
Unofficial Communities Documentation
├── uc-system-docs/           # Platform-wide (this repo)
│   └── 18 numbered documents
│
└── Per-repository:
    ├── CLAUDE.md             # Technical reference
    └── README.md             # GitHub overview
```

## Related Repositories

| Repository | Purpose |
|------------|---------|
| [unofficial-communities](https://github.com/unofficialbrasil/unofficial-communities) | Next.js frontend |
| [uc-api](https://github.com/unofficialbrasil/uc-api) | Fastify backend API |
| [uc-webhooks](https://github.com/unofficialbrasil/uc-webhooks) | Meta webhook processor |
| [uc-world](https://github.com/unofficialbrasil/uc-world) | 3D virtual world |

## Quick Start

Start with `01_SYSTEM_CANONICAL_INDEX.md` - it provides the reading order and document relationships.

## License

Proprietary - Unofficial Communities
