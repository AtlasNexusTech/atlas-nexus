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

Current MNEMOSYNE implementation:
- **Storage backend**: Hermes persistent memory system
- **Retrieval**: Automatic injection at session start + `session_search` for on-demand recall
- **Budget**: ~2,200 chars personal memory + ~1,375 chars user profile
- **Curation**: Manual pruning of stale entries; skills absorb procedural knowledge
