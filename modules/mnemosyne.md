# MNEMOSYNE — Memory

> *Decisions, incidents, configurations, learnings — persistent across sessions, models, and resets.*

## Role

MNEMOSYNE is the system's long-term memory. It ensures that context survives session boundaries, model changes, and infrastructure resets. Without MNEMOSYNE, every interaction starts from zero.

## Core Functions

### 1. Persistent Storage

Four categories of information are stored persistently:

| Category | Content | Retention | Example |
|----------|---------|-----------|---------|
| **Decisions** | What was decided, why, by whom, under what assumptions | Indefinite | "Allocated 500€ to ETF Monde — June 2026" |
| **Incidents** | Failures, root causes, fixes | Indefinite | "Telegram bot conflict: token collision with old instance" |
| **Configurations** | System state, environment, device setup | Until changed | "LOQ LAN IP: 10.24.94.77 (DHCP, may change)" |
| **Learnings** | Patterns, heuristics, corrections | Indefinite, curated | "DeepSeek V4 requires reasoning_content in all messages" |

### 2. Context Injection

At session start, MNEMOSYNE injects relevant prior context:
- User profile (name, preferences, communication style)
- Active project state (what was being worked on)
- Environment facts (OS, tools, paths)
- Recent decisions that may influence the current task

This is injected **before** the first user message, so the system arrives pre-loaded with context.

### 3. Retrieval

Two retrieval modes:

- **Automatic injection**: System-critical facts injected every session (user profile, environment)
- **On-demand search**: Keyword-based retrieval of past conversations and decisions (`session_search` equivalent)

### 4. Learning Loop

MNEMOSYNE doesn't just store — it learns:

```
Incident → Root Cause → Fix → Pattern →
"If this happens again, here's what to do"
```

When a correction is applied repeatedly, it's promoted from memory to a persistent instruction (skill).

## Storage Rules

### What to Store

- User preferences and corrections (highest priority)
- Environment facts that are stable (OS, tools, network)
- Non-obvious tool quirks and workarounds
- Recurring problem patterns

### What NOT to Store

- Temporary task progress (use session search)
- Stale artifacts (PR numbers, commit SHAs, file counts)
- Things easily re-discovered (standard commands, public API docs)
- Raw data dumps
- Completed work logs

### Quality Standards

- **Compact**: Memory budget is limited (~2,200 chars for personal notes). Every character must earn its place.
- **Declarative, not imperative**: "User prefers dark mode" ✓ — "Always use dark mode" ✗
- **Timeless**: If a fact will be stale in a week, it doesn't belong in memory

## Anti-Patterns

- **Memory as TODO list**: Task state goes in the task tracker, not memory
- **Memory as log**: Completed actions are retrieved via search, not stored permanently
- **Over-saving**: If a fact won't matter in the next session, don't save it
- **Instructional memory**: "Run tests with -n 4" is a skill, not a memory entry

## Implementation Notes

Current MNEMOSYNE implementation (v2 — May 2026):

### Storage Backend: Agent Memory Server
- **Server**: Python `http.server` running on LOQ (Windows), port 9736
- **Database**: SQLite (`agent_memory.db`), unlimited capacity
- **Location**: `C:\Users\alexa\Desktop\agent-memory\`
- **Availability**: LAN only (10.40.202.77:9736), survives reboots

### Multi-Layer Memory Architecture

```
Layer 1: Agent Memory Server (LOQ:9736) ← THIS DOCUMENT
  ├── SQLite, unlimited storage
  ├── Shared by ALL agents (Hermes, OOBE Scout, Android, etc.)
  ├── REST API: GET/POST/DELETE /memory/{agent_id}/{key}
  └── Survives reboots, cross-platform

Layer 2: Hermes Built-in Memory
  ├── ~2,200 chars personal memory
  ├── Injected every session
  └── Fast, always available

Layer 3: Session Search
  ├── Full transcript search across past sessions
  └── On-demand recall
```

### Hermes Skill
- **Skill**: `mnemosyne` (at `/skills/atlas/mnemosyne/`)
- **CLI Tool**: `~/.hermes/scripts/mnemosyne.py`
- **Dashboard**: HTML at `skills/atlas/mnemosyne/templates/dashboard.html`
- **Cron Health Check**: Job `98bd8bde80a1` runs hourly, monitors all agent states
- **Client Library**: `agent_memory_client.py` (Python, zero-dependency)

### Key Metrics
- **Agents tracked**: 8 (atlasnexusscout, hermes, athena, aegis, delphi, hephaestus, android-agent, mnemosyne-dashboard)
- **Standard keys**: status, cycle_count, total_cost_usdc, query_cooldowns, last_run, agent_id
- **Refresh**: Dashboard auto-refreshes every 30s
- **Staleness threshold**: >2h = warning, >6h = critical
