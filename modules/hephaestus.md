# HEPHAESTUS — Infrastructure

> *The forge where agents are built, deployed, and kept running.*

## Role

HEPHAESTUS handles everything operational: agent lifecycle, automation pipelines, monitoring, and reliability engineering. If it runs, HEPHAESTUS keeps it running.

## Core Functions

### 1. Agent Lifecycle Management

```
Spawn → Configure → Monitor → Scale → Retire

Each agent instance:
- Has a defined role (which ATLAS module it instantiates)
- Gets provisioned with appropriate tools
- Has resource limits (token budget, API call cap)
- Is monitored for health (response time, error rate)
```

### 2. Automation Infrastructure

- **Scheduled jobs**: Cron-equivalent for recurring tasks (health checks, data refreshes, yield scans)
- **Event-driven triggers**: Webhooks, file watchers, API callbacks
- **Pipeline orchestration**: Multi-step workflows with dependencies and error handling

### 3. Reliability Engineering

| Concern | Mechanism |
|---------|-----------|
| Model failure | Fallback to alternative provider/model |
| Rate limiting | Exponential backoff, request queuing |
| Context overflow | Automatic summarization and memory offload |
| Cost overrun | Budget alerts, per-task token caps |
| Silent failures | Health checks, heartbeat monitoring |

### 4. Multi-Environment Management

ATLAS runs across environments:

- **Development**: Local WSL2, full access, experimental
- **Production**: Gateway + cron pipeline, restricted tools
- **Testing**: Sandboxed instances for validation before deploy

HEPHAESTUS ensures consistent configuration across environments while respecting per-environment constraints (e.g., no file system access in production).

### 5. Gateway Operations

The Gateway is the external interface:
- Telegram bot for async interaction
- HTTP API for programmatic access
- Session management and routing
- Message delivery across platforms

## Infrastructure Principles

1. **Fail gracefully.** A dead model shouldn't crash the system — fall back, retry, alert.
2. **Observable by default.** Every action produces a log. No silent failures.
3. **Cost-aware.** Every API call has a price tag. Track it.
4. **Reproducible.** Infrastructure-as-code. Configs, not manual setup.
5. **Minimal privileged surface.** Agents get the minimum tools they need for their role.

## Current Stack

| Component | Implementation |
|-----------|---------------|
| Runtime | Hermes Agent v0.13.0 |
| Primary Model | DeepSeek V4 Pro |
| Delegation | Claude Sonnet 4.6 (max 4 concurrent, depth 2) |
| Gateway | Hermes Gateway (Telegram + HTTP) |
| Host | WSL2 on Lenovo LOQ 83FQ (8 CPU, 17 GB RAM, 1 TB SSD) |
| Scheduling | Hermes cron (8 jobs) + GitHub Actions (market data) |
| Memory | Hermes persistent memory + session search |
