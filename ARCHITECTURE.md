# ATLAS Architecture

## Overview

ATLAS is a distributed cognition framework. It treats decision-making as a process that can be decomposed, parallelized, verified, and logged — just like software.

The architecture has three layers:

```
┌─────────────────────────────────────────────────────────┐
│                    DECISION LAYER                        │
│  HERMES ←→ ATHENA ←→ AEGIS ←→ DELPHI                   │
│  (coordinate, govern, validate, synthesize)             │
├─────────────────────────────────────────────────────────┤
│                  INFRASTRUCTURE LAYER                    │
│  HEPHAESTUS                                             │
│  (deploy, monitor, recover, automate)                   │
├─────────────────────────────────────────────────────────┤
│                    MEMORY LAYER                          │
│  MNEMOSYNE                                              │
│  (persist, index, retrieve, learn)                      │
└─────────────────────────────────────────────────────────┘
```

## Decision Layer

The four modules in this layer collaborate to transform inputs into verified decisions:

### Flow

1. **HERMES** receives the task, assesses priority, routes to the appropriate module(s)
2. **DELPHI** synthesizes relevant information, builds models, generates options
3. **AEGIS** stress-tests the options — counter-arguments, edge cases, assumptions check
4. **ATHENA** validates against governance rules, ensures architectural coherence
5. **HERMES** presents the final recommendation to the human for arbitration (or auto-executes if within delegated authority)

### Key Properties

- **Parallelizable**: DELPHI and AEGIS can run concurrently on different sub-problems
- **Re-entrant**: Any module can escalate back to HERMES if it hits its knowledge boundary
- **Traceable**: Every step is logged to MNEMOSYNE

## Infrastructure Layer

HEPHAESTUS handles everything that makes the system run:

- **Agent lifecycle**: Spawn, monitor, terminate agent instances
- **Tool provisioning**: Which tools each module has access to
- **Reliability**: Retry logic, fallback models, circuit breakers
- **Automation**: Scheduled jobs (cron-equivalent), event-driven triggers
- **Cost tracking**: Token usage, API costs, compute budget

## Memory Layer

MNEMOSYNE is the persistent knowledge store:

### What Gets Stored

- **Decisions**: What was decided, by whom, with what rationale, under what assumptions
- **Incidents**: Things that went wrong, root causes, remediation steps
- **Configurations**: System state snapshots, environment variables (secrets excluded)
- **Learnings**: Patterns, heuristics, corrections that emerged over time

### Retrieval Patterns

- **Session injection**: Current session gets relevant prior context automatically
- **Keyword search**: Fast recall of specific past decisions
- **Pattern matching**: Detecting recurring problem structures

## Design Constraints

1. **Session-resilient**: Architecture must survive context window resets. Memory layer is the foundation.
2. **Model-agnostic**: Any capable LLM can fill any module role. No vendor lock-in.
3. **Human-in-the-loop by default**: Full autonomy requires explicit delegation. The default is recommendation, not execution.
4. **Minimal state surface**: Complex state = fragile state. Each module maintains only what it strictly needs.
5. **Observable**: Every action produces a log entry. No black boxes.

## Current Implementation

ATLAS is currently implemented via Hermes Agent as the runtime, with DeepSeek V4 Pro as the primary model. Each module is instantiated as a skill + prompt configuration.

See [`modules/`](modules/) for the per-module specifications.
