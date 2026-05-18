# ATLAS Decision Protocol

> *A reproducible process for non-trivial decisions. Follow the steps. Log the outcome.*

---

## When to Use

Apply this protocol when a decision involves:
- Material resource allocation (>50€ or >1 hour)
- Irreversible or hard-to-reverse actions
- Multiple stakeholders or downstream dependencies
- Significant uncertainty

Skip this protocol for:
- Routine operational tasks
- Reversible experiments with negligible cost
- Decisions where the cost of process exceeds the cost of error

---

## Phase 1: Frame (HERMES + DELPHI)

### 1.1 Define the Decision

```
What is being decided?
Why now? (What changed?)
What happens if no decision is made? (Default path)
Who needs to approve?
```

### 1.2 Gather Options

DELPHI generates at least three distinct options with:
- Expected outcome (quantified)
- Key assumptions
- Resource requirements
- Timeline
- Failure modes

### 1.3 Build Comparison Table

| Criterion | Weight | Option A | Option B | Option C |
|-----------|--------|----------|----------|----------|
| Expected ROI | 40% | X€ | Y€ | Z€ |
| Risk level | 30% | Low | Medium | High |
| Implementation time | 20% | Fast | Medium | Slow |
| Strategic alignment | 10% | High | Medium | Low |

---

## Phase 2: Stress Test (AEGIS)

### 2.1 Assumption Audit

For the leading option(s), list and challenge every assumption:

```
Assumption: [X]
Challenge: What if X is wrong? What would that look like?
Mitigation: How would we detect and recover?
```

### 2.2 Pre-Mortem

"Assume we executed this decision and it failed. What caused the failure?"

### 2.3 Risk Score

```
Impact (1-5) × Probability (0-1) × Reversibility (0.2-1)
```

| Score | Action |
|-------|--------|
| <3 | Proceed |
| 3-6 | Proceed with caution, flag to HERMES |
| 6-12 | Mandatory human approval |
| >12 | Full quorum required |

---

## Phase 3: Validate (ATHENA)

### 3.1 Standards Check

- Does this decision violate any standing rules?
- Is it coherent with the current architecture and strategy?
- Are there undocumented side effects?

### 3.2 Precedent Check

- Have we made similar decisions before?
- What happened? (Check MNEMOSYNE)

---

## Phase 4: Decide (HERMES + Human)

### 4.1 Recommendation

HERMES presents:
1. Recommended option with rationale
2. Key trade-offs acknowledged
3. Confidence level (HIGH/MEDIUM/LOW)
4. Open questions that remain

### 4.2 Human Arbitration

- **Delegated authority**: System auto-executes and logs
- **Human approval required**: Human reviews and confirms
- **Escalation**: If modules disagree, human resolves

---

## Phase 5: Log (MNEMOSYNE)

### 5.1 Decision Record

```
Date: YYYY-MM-DD
Decision: [What was decided]
Options considered: [List]
Rationale: [Why this option]
Risk score: [X/25]
Assumptions: [Key assumptions to monitor]
Approved by: [Human / Autonomous under delegation]
Next review: [Date or trigger condition]
```

### 5.2 Monitoring Triggers

Define what would cause this decision to be revisited:
- "If APY drops below X%"
- "If cost exceeds Y€"
- "After Z months with no results"

---

## Quick Reference Card

```
FRAME → STRESS TEST → VALIDATE → DECIDE → LOG

Who does what:
  DELPHI: Options + comparison
  AEGIS:  Assumptions + risk score
  ATHENA: Rules + coherence
  HERMES: Recommendation + arbitration
  MNEMOSYNE: Permanent record
```
