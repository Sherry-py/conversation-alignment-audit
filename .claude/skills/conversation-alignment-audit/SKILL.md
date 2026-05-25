---
name: gatefix-audit
version: 1.0.0
description: |
  GateFix AI Agent Governance Audit. Analyzes an AI agent system to identify
  high-risk commit points, evaluate current governance state against the 4D-CQ
  framework (Relevance, Coverage, Ordering, Robustness), and produce a
  risk-ranked governance report with ICAG measurement setup.
  
  Invoke when: user wants to audit an AI agent deployment, identify where AI
  actions are irreversible, assess governance gaps before production deployment,
  or prepare a GateFix pilot engagement.
allowed-tools:
  - Bash
  - Read
  - Glob
  - WebSearch
triggers:
  - gatefix audit
  - audit my agent
  - find commit points
  - governance audit
  - where are the risks in my agent
---

## GateFix Audit Skill

**Theory**: Context quality is not a property of context — it is a property of the
(context, action) pair. Governance effort should concentrate at commit points:
the moments when AI planning becomes irreversible action.

**4D-CQ Framework**:
- Relevance [CRITICAL] — Does the AI recommendation match the user's actual goal domain?
- Coverage [CRITICAL] — Are all necessary preconditions verified before proceeding?
- Ordering [NON-CRITICAL] — Is sequencing and prerequisite logic enforced?
- Robustness [NON-CRITICAL] — Does the recommendation hold across reasonable variations?

**Decision rule φ**:
- Any CRITICAL dimension unaddressed → HIGH risk (equivalent to REFUSE zone)
- Any NON-CRITICAL dimension unaddressed → MEDIUM risk (equivalent to CLARIFY zone)
- All dimensions addressed → governance-ready

---

## Step 0 — Establish input mode

Ask the user (or infer from context) which input they're providing:

**A) Codebase** — they're in an agent repo (check for .py, .ts files, MCP configs)
**B) Description** — they describe what their agent does in natural language  
**C) Tool schema** — they provide function definitions, MCP server schemas, or API specs

If codebase is present in current directory, default to A without asking.
State which mode you're using.

---

## Step 1 — Action inventory

**Goal**: List every action the agent can perform that has an external effect.

### If mode A (codebase):
Search for:
```bash
# Find external action patterns
grep -rn "def \(submit\|send\|create\|delete\|update\|pay\|enroll\|post\|execute\|write\|book\|schedule\|approve\|reject\|assign\)" . --include="*.py" -l
grep -rn "requests\.\(post\|put\|delete\|patch\)" . --include="*.py" -l
grep -rn "tool_call\|function_call\|mcp\." . --include="*.py" -l
```
Read the relevant files. Extract function names and what external system each touches.

### If mode B (description):
Extract every verb-object pair that implies an external effect:
"sends email" / "books appointment" / "places order" / "updates record" / "submits form"

### If mode C (tool schema):
Read the schema. List every tool with its name, description, and parameters.
Flag tools with write semantics (create, update, delete, send, execute).

**Output**: Numbered list of all agent actions with brief description of external effect.

---

## Step 2 — Commit point classification

For each action from Step 1, apply the reversibility test:

| Question | Yes → | No → |
|----------|-------|------|
| Can this be fully undone within 24h at near-zero cost? | lower risk | commit candidate |
| Does this trigger a financial transaction? | commit candidate | — |
| Does this create a legal obligation or regulatory record? | commit candidate | — |
| Does this change state in an external system you don't control? | commit candidate | — |

**Irreversibility tiers**:
- **HARD commit**: financial transaction, legal obligation, external system mutation (no rollback)
- **SOFT commit**: internal state change, reversible within same session, notification only

**Output**: Table of commit point candidates, ranked by irreversibility tier, then estimated frequency.

```
COMMIT POINT CANDIDATES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Rank  Action              Tier   Frequency  External System
1.    execute_payment()   HARD   high       payment gateway
2.    send_offer()        HARD   medium     email + HR system  
3.    enroll_course()     HARD   medium     university system
4.    schedule_meeting()  SOFT   high       calendar API
```

---

## Step 3 — 4D-CQ governance gap assessment

For each HARD commit point, evaluate current governance state across all four dimensions.

For each dimension, ask: **does the system currently check this before the commit executes?**

### Relevance [CRITICAL]
Does the system verify the AI recommendation falls within the user's stated goal domain?
- Look for: goal extraction, scope validation, intent matching before commit
- Red flag: agent infers goals from conversation without explicit confirmation

### Coverage [CRITICAL]  
Does the system verify all preconditions are met before proceeding?
- Look for: prerequisite checks, eligibility validation, rule oracle calls
- Red flag: agent proceeds on partial information, missing data treated as "probably fine"

### Ordering [NON-CRITICAL]
Does the system enforce correct sequencing?
- Look for: prerequisite DAG, dependency checks, temporal ordering logic
- Red flag: agent recommends step N before step N-1 is confirmed complete

### Robustness [NON-CRITICAL]
Does the recommendation hold under reasonable variation?
- Look for: edge case handling, alternative scenario testing, confidence thresholds
- Red flag: single-path reasoning, no consideration of user profile variations

**Output per commit point**:
```
COMMIT POINT: execute_payment()
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Relevance  [CRITICAL]     ❌ UNADDRESSED — agent infers payment intent without confirmation
Coverage   [CRITICAL]     ❌ UNADDRESSED — no balance/limit check before execution  
Ordering   [NON-CRITICAL] ✅ addressed — cart validation runs first
Robustness [NON-CRITICAL] ⚠️  partial — no handling for concurrent transaction edge cases

Risk: HIGH (2 CRITICAL dimensions unaddressed)
ICAG exposure: unknown — no C2 oracle defined
```

---

## Step 4 — ICAG setup recommendation

For each HIGH-risk commit point, propose the oracle structure needed to measure
the Inter-Commit Authorization Gap (ICAG).

**ICAG = Auth_Accuracy(C1) − Auth_Accuracy(C2)**

For single-commit systems: propose C1 oracle only.
For multi-commit systems: propose C1 + C2 oracles and flag inter-commit misalignment risk.

**Oracle template**:
```python
def [commit_name]_oracle(plan, context) -> bool:
    """
    Deterministic authorization check for [commit_name].
    Based on: [public rule source — handbook, regulation, published policy]
    NOT a live system call — mirrors published rules only.
    """
    # Relevance check
    # Coverage check  
    # Ordering check
    # Robustness check
    return all_checks_pass
```

State clearly:
- What public data source grounds this oracle
- Which 4D dimensions it covers
- What would make it HARD to implement (missing public rules → flag as research question)

---

## Step 5 — Governance report

Produce the final report in this structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GATEFIX GOVERNANCE AUDIT REPORT
System: [agent name / description]
Date: [today]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

EXECUTIVE SUMMARY
[2-3 sentences: what the agent does, how many commit points found, overall risk level]

COMMIT POINTS IDENTIFIED: [N]
  HIGH risk:   [N] — requires governance before production deployment
  MEDIUM risk: [N] — recommended governance
  LOW risk:    [N] — monitor only

━━━ HIGH RISK COMMIT POINTS ━━━━━━━━━━━━━━━━━━

[For each HIGH risk commit point:]

[N]. [action_name()]
  What it does: [one sentence]
  Why it's a hard commit: [irreversibility reason]
  Governance gaps:
    ❌ Relevance — [specific gap description]
    ❌ Coverage  — [specific gap description]
  Recommended oracle: [brief spec]
  ICAG exposure: [can it be measured? what data needed?]
  Priority: [deploy blocker / high / medium]

━━━ MEDIUM RISK COMMIT POINTS ━━━━━━━━━━━━━━━

[same structure, briefer]

━━━ GOVERNANCE RECOMMENDATIONS ━━━━━━━━━━━━━━

Immediate (before any production deployment):
1. [specific action]
2. [specific action]

Near-term (within 30 days):
1. [specific action]

ICAG MEASUREMENT PLAN
To establish baseline authorization accuracy:
- C1 oracle: [what to build, based on what public source]
- C2 oracle: [if applicable]
- Measurement: run [N] synthetic plans through both arms (governed vs ungoverned)
  and record verdict distribution

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Completion

After delivering the report, offer:
- "Want me to draft the oracle code for any of these commit points?"
- "Want me to generate synthetic test plans to estimate ICAG?"
- "Want a one-page summary for a non-technical stakeholder?"

**DONE** when report is delivered and user's follow-up questions are addressed.
