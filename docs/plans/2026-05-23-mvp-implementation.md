# Conversation Alignment Audit MVP Implementation Plan

> **For agentic workers:** Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship v0.1.0 of `conversation-alignment-audit` — a Claude Code SKILL that audits multi-turn conversations against a stated objective using the 4D-CQ framework, with structured turn-referenced output, profile-parameterized thresholds, and zero-server local execution.

**Architecture:** Markdown-first SKILL (per CLAUDE.md). All logic — adapter, dimension scoring, verdict rule φ, audit trail — lives in `SKILL.md` instructions. No Python runtime in v0.1.0. Three adapters convert external transcript formats to canonical schema; downstream operations work on canonical only (invariant I8). Verdict rule is deterministic and external to LLM (invariant I7).

**Tech Stack:** Markdown · YAML frontmatter · Claude Code skill mechanism · JSONL audit trail file · No Python · No external dependencies.

**Spec basis:** `docs/spec/001-mvp-conversation-audit.md` (commit `817ed13`).

---

## File Structure

| File | Created/Modified | Responsibility |
|---|---|---|
| `.claude/skills/conversation-alignment-audit/SKILL.md` | Create | Main skill artifact — frontmatter + theory + 8-step flow + 4 dim prompts + verdict + report |
| `README.md` | Modify (currently empty) | Public-facing entry — install, usage, privacy, commercial CTA, citation |
| `LICENSE` | Create | Dual license — MIT for non-commercial, commercial clause for client engagements |
| `docs/theory.md` | Create | Short framework intro linking to Paper 1 |
| `docs/services.md` | Create | Commercial offerings page (workshop / cert / license / audit) |
| `docs/profiles.md` | Create | Profile registry documentation + override syntax |
| `docs/installation.md` | Create | Three install paths (git clone / curl / future marketplace) |
| `docs/usage.md` | Create | How to invoke + interpret output |
| `CONTRIBUTING.md` | Create | Contribution norms — PR style, issue templates |
| `examples/01-strategy-meeting.md` | Create | Example transcript: strategy meeting with intentional Coverage gap |
| `examples/02-due-diligence.md` | Create | Example transcript: due diligence with topic drift (Ordering gap) |
| `examples/03-interview.md` | Create | Example transcript: hiring interview with robustness issue |

**Skill template source:** `~/.claude/skills/gatefix-audit/SKILL.md` (used as starting structure in Task 1).

---

## Phase 2 — SKILL.md authoring

### Task 1: Copy gatefix-audit template as starting point

**Files:**
- Create: `.claude/skills/conversation-alignment-audit/SKILL.md`
- Source: `~/.claude/skills/gatefix-audit/SKILL.md`

- [ ] **Step 1: Create skill directory**

```bash
mkdir -p .claude/skills/conversation-alignment-audit
```

- [ ] **Step 2: Copy gatefix-audit SKILL.md as starting structure**

```bash
cp ~/.claude/skills/gatefix-audit/SKILL.md .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Verify file copied**

```bash
wc -l .claude/skills/conversation-alignment-audit/SKILL.md
```

Expected: ~250 lines.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.1: Copy gatefix-audit SKILL as starting template"
```

---

### Task 2: Replace frontmatter

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md:1-24`

- [ ] **Step 1: Replace YAML frontmatter**

Replace lines 1-24 (the existing gatefix-audit frontmatter) with:

```yaml
---
name: conversation-alignment-audit
version: 0.1.0
description: |
  Pre-execution audit of multi-turn conversations against a stated
  objective using the 4D-CQ framework (Relevance, Coverage, Ordering,
  Robustness). Returns deterministic verdict (PASS / CLARIFY / REFUSE)
  with structured turn-referenced evidence. Operates locally; no data
  leaves the user's machine.

  Invoke when: user has a transcript of a meeting, interview, due
  diligence call, board discussion, or other multi-turn conversation
  and wants a rigorous, theoretically-grounded audit of whether the
  conversation is ready to convert into action.
allowed-tools:
  - Read
  - Write
  - Bash
triggers:
  - audit this conversation
  - alignment audit
  - conversation alignment
  - 4D-CQ audit
  - is this meeting ready for decision
  - audit transcript
---
```

- [ ] **Step 2: Verify frontmatter parses**

```bash
head -30 .claude/skills/conversation-alignment-audit/SKILL.md
```

Expected: valid YAML with `name: conversation-alignment-audit`.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.2: Replace frontmatter with conversation-alignment-audit metadata"
```

---

### Task 3: Rewrite Theory and 4D-CQ definition

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (body — after frontmatter)

- [ ] **Step 1: Delete existing gatefix-audit body**

After the frontmatter, delete all existing content (lines 25 to end). The body will be rewritten from scratch.

- [ ] **Step 2: Add Overview and Theory section**

Insert below frontmatter:

```markdown
## Conversation Alignment Audit Skill

**Theory.** Multi-turn conversations in high-stakes contexts (strategic meetings,
due diligence, board discussions, hiring panels) routinely produce decisions
whose conversational substrate is misaligned with the stated objective.
Participants diagnose drift in the moment but authorize the decision anyway.

This is the same diagnostic-authorization gap formalized in Lian (2026,
*Information & Management*) for LLM agents: diagnostic capability is present,
authorization proceeds regardless, and the gap is structural — not closeable
by adding more attention or better facilitation.

The remedy is a **pre-execution audit layer**: an external, deterministic
verdict on whether the conversation is ready to convert into action.

**4D-CQ Framework** (Lian, 2026 — derived from Wang & Strong, 1996 contextual
data quality, specialized for the moment of action authorization):

- **Relevance** [CRITICAL] — Does the conversation discuss the stated objective?
- **Coverage** [CRITICAL] — Are required facts, perspectives, and constraints present?
- **Ordering** [NON-CRITICAL] — Is the discussion sequence coherent enough to support valid conclusions?
- **Robustness** [NON-CRITICAL] — Is the conclusion stable across reasonable participant or framing variation?

**Verdict rule φ** (deterministic; LLM never decides):

- Any CRITICAL dimension below `critical_threshold` → `REFUSE`
- Otherwise, any NON-CRITICAL dimension below `non_critical_threshold` → `CLARIFY`
- Otherwise → `PASS`

Thresholds are profile parameters, not constants. Default profile is `standard`.

**Architecture invariants** (must hold):

- LLM never decides final verdict (I2)
- Every verdict cites evidence — structured turn references (I3)
- All scoring operates on canonical transcript only (I8)
- Verdict rule φ lives in deterministic code, never in LLM prompt (I7)

---
```

- [ ] **Step 3: Verify content**

```bash
grep -c "4D-CQ\|Wang & Strong\|Lian" .claude/skills/conversation-alignment-audit/SKILL.md
```

Expected: ≥ 3 matches.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.3: Rewrite Theory + 4D-CQ definition (replaces gatefix-audit body)"
```

---

### Task 4: Add Step 0 (Input validation)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 0 section**

```markdown
## Step 0 — Input validation

Ask the user (or extract from context) the following inputs:

1. **Transcript** — file path OR pasted content. Required.
2. **Objective** — 1-3 sentence statement of what the conversation should accomplish. Required.
3. **Profile** — one of `strict | standard | lenient`, or path to custom profile YAML. Default: `standard`.
4. **Optional flags** — `--json` for raw output, `--verbose-evidence` for full quoted spans.

**Validation:**

- If transcript path: verify file exists and is non-empty
- If pasted content: verify non-empty (≥ 50 characters)
- Objective: verify non-empty and ≤ 500 words
- Profile: verify resolves to known profile name OR loadable YAML

**On failure:** return `REFUSE` with `evaluation_incomplete: true` and reason in `blocking_failures`.

---
```

- [ ] **Step 2: Verify section appended**

```bash
grep -n "Step 0 — Input validation" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.4a: Add Step 0 (input validation)"
```

---

### Task 5: Add Step 0.5 (Adapter — external to canonical)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 0.5 section**

```markdown
## Step 0.5 — Adapter: external format → canonical

The skill ships three adapters. All downstream operations use canonical only (invariant I8).

### Format detection (heuristic auto-detect when `format_hint` is null)

1. If content starts with `[` and parses as JSON array → `json_explicit`
2. Else if content has lines matching `^\*\*[A-Za-z0-9_\- ]+:\*\*` → `markdown_tagged`
3. Else if content has lines matching `^[A-Za-z0-9_\- ]+:` (speaker tag pattern) → `plain_tagged`
4. Else → ambiguity protocol (ask user)

### Adapter A — `plain_tagged`

Input pattern: `Alice: hello\nBob: hi\nAlice: how are you?`

Parse steps:
1. Split content by newlines
2. For each line, match `^([A-Za-z0-9_\- ]{1,40}):\s*(.*)$`
3. If match: `speaker_name = group(1).strip(); text = group(2).strip()`
4. If no match: append text to previous turn (continuation)
5. Build participant table (unique speaker names → `P1`, `P2`, ...)
6. Output canonical: `{participants, turns, metadata}`

### Adapter B — `markdown_tagged`

Input pattern: `**Alice:** hello\n\n**Bob:** hi`

Parse steps:
1. Split by `\*\*([^*]+):\*\*` boundary (regex)
2. Each boundary marks a new turn
3. Capture name in group; remaining text is turn content
4. Apply same participant mapping as plain_tagged
5. Output canonical

### Adapter C — `json_explicit`

Input pattern: `[{"speaker": "Alice", "text": "hello"}, ...]`

Parse steps:
1. Parse JSON
2. Validate each item has `speaker` (string) and `text` (string)
3. Apply same participant mapping
4. Optional: respect `ts` (timestamp) field if present
5. Output canonical

### Ambiguity protocol

If adapter fails or is ambiguous, **pause and ask user** (do not guess silently):

- Multiple tag styles → "Are these the same person? (Alice ↔ A)"
- No speaker tags → "Treat as single-speaker monologue? Add tags manually? Abort?"
- Failed JSON parse → "JSON is invalid. Switch to plain_tagged adapter? Show error?"

### Canonical output schema

```yaml
transcript_canonical:
  source:
    format_detected: "plain_tagged" | "markdown_tagged" | "json_explicit"
    original_size_chars: int
  participants:
    - { id: "P1", display_name: "Alice", role: null }
    - { id: "P2", display_name: "Bob", role: null }
  turns:
    - { turn_number: 1, speaker_id: "P1", text: "...", timestamp: null }
    - { turn_number: 2, speaker_id: "P2", text: "...", timestamp: null }
  metadata:
    total_turns: int
    total_chars: int
    language: "en" | "zh" | "mixed" | null
```

### Validation

- Every turn has `turn_number` (1-indexed, contiguous), `speaker_id`, `text`
- `participants` is non-empty
- Each `speaker_id` referenced in turns must exist in participants

On validation failure: retry once; persistent failure → `REFUSE` with `evaluation_incomplete: true`.

### Truncation policy (very long transcripts)

If `total_turns > 200`:
- Scoring window: first 10 turns (objective-setting) + last 30 turns (conclusion)
- Set `audit_result.metadata.scoring_window = "first_10_last_30"`
- Canonical itself remains complete

---
```

- [ ] **Step 2: Verify section + check examples**

```bash
grep -n "Adapter A\|Adapter B\|Adapter C\|Ambiguity protocol" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.4b: Add Step 0.5 (3 adapters + canonical schema + ambiguity protocol)"
```

---

### Task 6: Add Step 1 (Relevance evaluator)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 1 with Relevance evaluator prompt**

```markdown
## Step 1 — Relevance evaluation [CRITICAL]

**Question:** Does the conversation discuss the stated objective?

**LLM evaluator prompt template:**

```
You are evaluating one dimension of a conversation audit: RELEVANCE.

OBJECTIVE OF THE CONVERSATION:
{objective_statement}

OPTIONAL CONTEXT:
{objective_context}

CONVERSATION (canonical turns):
{render_turns(canonical.turns)}

PARTICIPANTS:
{render_participants(canonical.participants)}

Your task:
1. Determine to what extent the conversation actually discusses the stated objective.
2. A high score means most turns are on-topic. A low score means significant drift, irrelevant topics, or never reaching the objective.
3. Provide a score in [0.0, 1.0] (two decimal places).
4. Cite specific turn_numbers as evidence (a list, e.g., [3, 7, 12]).
5. Quote 1-3 spans (each: turn_number + speaker_id + text excerpt up to 50 words).
6. Provide a 1-2 sentence rationale.

CRITICAL: Your output MUST be valid JSON matching this schema:

{
  "dimension": "relevance",
  "score": float,
  "evidence": {
    "turn_numbers": [int, ...],
    "quoted_spans": [
      {"turn": int, "speaker_id": "P1|P2|...", "quote": "..."}
    ]
  },
  "rationale": "string"
}

Every turn_number cited MUST exist in the canonical turns above.
Do not invent turn numbers. Do not cite spans outside the canonical.

Return ONLY the JSON object. No commentary.
```

**Post-processing:**

1. Parse LLM output as JSON
2. Schema-validate (score in [0,1], turn_numbers ⊆ canonical.turns)
3. If invalid → retry once with appended instruction "Previous output was invalid: {error}. Return valid JSON only."
4. If still invalid → set score=0.0, flag `evaluation_incomplete=true`

---
```

- [ ] **Step 2: Verify prompt completeness**

```bash
grep -A 30 "Step 1 — Relevance" .claude/skills/conversation-alignment-audit/SKILL.md | head -40
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.5a: Add Relevance dimension evaluator with prompt template"
```

---

### Task 7: Add Step 2 (Coverage evaluator)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 2 with Coverage evaluator prompt**

```markdown
## Step 2 — Coverage evaluation [CRITICAL]

**Question:** Are required facts, perspectives, and constraints present in the conversation?

**LLM evaluator prompt template:**

```
You are evaluating one dimension of a conversation audit: COVERAGE.

OBJECTIVE OF THE CONVERSATION:
{objective_statement}

OPTIONAL CONTEXT:
{objective_context}

CONVERSATION (canonical turns):
{render_turns(canonical.turns)}

PARTICIPANTS:
{render_participants(canonical.participants)}

Your task:
1. Identify what facts, perspectives, or constraints would be REQUIRED to validly act on the stated objective. Be specific (e.g., "Q3 actual revenue figure", "engineering team capacity", "regulatory exposure for option B").
2. For each required item, mark whether it is PRESENT or MISSING in the conversation.
3. Score in [0.0, 1.0] where 1.0 means all required items are present and 0.0 means most are missing.
4. Provide turn_numbers for items present, and a list of MISSING items by name.
5. Provide a 1-2 sentence rationale.

CRITICAL: Output JSON only:

{
  "dimension": "coverage",
  "score": float,
  "evidence": {
    "turn_numbers": [int, ...],
    "quoted_spans": [
      {"turn": int, "speaker_id": "...", "quote": "..."}
    ]
  },
  "missing": ["Q3 revenue figure", "engineering team capacity", ...],
  "rationale": "string"
}

Every turn_number cited MUST exist in the canonical.
Do not list as "missing" items that the objective does not actually require.
Return ONLY the JSON object.
```

**Post-processing:** same as Step 1 (parse, validate, retry, fallback).

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 2 — Coverage" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.5b: Add Coverage dimension evaluator (with missing-items list)"
```

---

### Task 8: Add Step 3 (Ordering evaluator)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 3 with Ordering evaluator prompt**

```markdown
## Step 3 — Ordering evaluation [NON-CRITICAL]

**Question:** Is the discussion sequence coherent enough to support valid conclusions?

**LLM evaluator prompt template:**

```
You are evaluating one dimension of a conversation audit: ORDERING.

OBJECTIVE OF THE CONVERSATION:
{objective_statement}

CONVERSATION (canonical turns):
{render_turns(canonical.turns)}

Your task:
1. Examine the sequence of turns. Are topics introduced in a coherent order? Are conclusions reached before the relevant evidence is discussed? Are decisions taken before the necessary alternatives have been weighed?
2. Identify "ordering failures": topic shifts without bridge, conclusions ahead of evidence, premature decisions, missing transitions.
3. Score in [0.0, 1.0] — 1.0 = strong sequencing, 0.0 = chaotic / out-of-order.
4. Cite turn_numbers where ordering failures occur (or where sequencing is exemplary if score is high).
5. Provide a 1-2 sentence rationale.

CRITICAL: Output JSON only:

{
  "dimension": "ordering",
  "score": float,
  "evidence": {
    "turn_numbers": [int, ...],
    "quoted_spans": [
      {"turn": int, "speaker_id": "...", "quote": "..."}
    ]
  },
  "rationale": "string"
}

Return ONLY the JSON object.
```

**Post-processing:** same as Step 1.

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 3 — Ordering" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.5c: Add Ordering dimension evaluator"
```

---

### Task 9: Add Step 4 (Robustness evaluator)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 4 with Robustness evaluator prompt**

```markdown
## Step 4 — Robustness evaluation [NON-CRITICAL]

**Question:** Is the conclusion stable across reasonable participant or framing variation?

**LLM evaluator prompt template:**

```
You are evaluating one dimension of a conversation audit: ROBUSTNESS.

OBJECTIVE OF THE CONVERSATION:
{objective_statement}

CONVERSATION (canonical turns):
{render_turns(canonical.turns)}

PARTICIPANTS:
{render_participants(canonical.participants)}

Your task:
1. Consider the conclusion or direction the conversation is heading toward.
2. Counterfactual probe: would the conclusion change if any one participant had been absent? If a different framing of the objective had been used? If a known objection had been raised but wasn't?
3. Score in [0.0, 1.0] — 1.0 = conclusion holds under variation, 0.0 = conclusion is entirely contingent on specific framings or specific speakers.
4. Cite turn_numbers where the conclusion appears fragile (or robust if score is high).
5. Provide a 1-2 sentence rationale.

CRITICAL: Output JSON only:

{
  "dimension": "robustness",
  "score": float,
  "evidence": {
    "turn_numbers": [int, ...],
    "quoted_spans": [
      {"turn": int, "speaker_id": "...", "quote": "..."}
    ]
  },
  "rationale": "string"
}

Return ONLY the JSON object.
```

**Post-processing:** same as Step 1.

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 4 — Robustness" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.5d: Add Robustness dimension evaluator (counterfactual probe)"
```

---

### Task 10: Add Step 5 (Verdict rule φ — deterministic)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 5 with deterministic verdict logic**

```markdown
## Step 5 — Verdict computation

This step is **deterministic**. No LLM call. The verdict reads from the profile object and applies pure logic.

**Pseudocode:**

```
function compute_verdict(scores, profile):
    critical_t      = profile.critical_threshold
    non_critical_t  = profile.non_critical_threshold

    # Per-dimension overrides if present
    relevance_t     = profile.dimension_overrides.relevance.threshold   or critical_t
    coverage_t      = profile.dimension_overrides.coverage.threshold    or critical_t
    ordering_t      = profile.dimension_overrides.ordering.threshold    or non_critical_t
    robustness_t    = profile.dimension_overrides.robustness.threshold  or non_critical_t

    # CRITICAL failures dominate
    if scores.relevance < relevance_t OR scores.coverage < coverage_t:
        return REFUSE

    # NON-CRITICAL failures secondary
    if scores.ordering < ordering_t OR scores.robustness < robustness_t:
        return CLARIFY

    return PASS
```

**Edge cases:**

- Any score is `null` / `NaN` → treat as `0.0` (worst-case)
- All scores ≥ all thresholds → `PASS`
- Critical + non-critical both fail → `REFUSE` (critical dominates)
- Schema validation failed twice → return `REFUSE` with `evaluation_incomplete: true`

**Default profiles (registry):**

```yaml
strict:
  critical_threshold: 0.75
  non_critical_threshold: 0.80
standard:
  critical_threshold: 0.60
  non_critical_threshold: 0.70
lenient:
  critical_threshold: 0.50
  non_critical_threshold: 0.60
```

These defaults are MVP calibration targets (see Phase 3). Per-dimension overrides are valid in any profile.

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 5 — Verdict\|compute_verdict" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.6a: Add Step 5 (deterministic verdict rule φ + 3 default profiles)"
```

---

### Task 11: Add Step 6 (Drift signal extraction)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 6 with drift signal logic**

```markdown
## Step 6 — Drift signal extraction

From the dimension scoring results (especially Ordering rationale and Coverage missing items), extract structured drift signals.

**Logic:**

1. From Ordering evaluator: convert each cited turn range into a `topic_shift_without_bridge` signal if rationale mentions "shift", "transition", "jump", or "drift".
2. From Coverage evaluator: convert each `missing` item into a `missing_evidence` signal — turn_range is the turn closest to when the missing item should have been raised.
3. From Robustness evaluator: convert each fragility citation into a `conclusion_contingent_on` signal.
4. Severity: `high` if score < 0.4; `medium` if 0.4 ≤ score < 0.6; `low` if 0.6 ≤ score.

**Output format:** ordered list of `{turn_range, type, severity, description}` per the §output schema.

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 6 — Drift" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.6b: Add Step 6 (drift signal extraction from dimension evidence)"
```

---

### Task 12: Add Step 7 (Audit trail persistence)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 7 with audit trail writer**

```markdown
## Step 7 — Audit trail persistence

Append a single JSON line to `~/.conversation-audit-trail.jsonl`.

**JSONL entry schema:**

```json
{
  "audit_id": "uuid-v4",
  "timestamp": "2026-05-23T14:30:00Z",
  "verdict": "PASS | CLARIFY | REFUSE",
  "evaluation_incomplete": false,
  "scores": { "relevance": 0.82, "coverage": 0.65, "ordering": 0.61, "robustness": 0.84 },
  "thresholds_applied": { "critical": 0.60, "non_critical": 0.70 },
  "profile_used": "standard",
  "transcript_metadata": { "total_turns": 47, "language": "en", "scoring_window": "full" },
  "objective_hash": "sha256-of-objective-statement",
  "framework_citation": "Lian (2026), 4D-CQ Framework, I&M"
}
```

**Important — what is NOT logged:**

- Transcript content (user's data; never on trail)
- Quoted evidence spans (privacy)
- Participant display names (privacy)
- Objective statement itself (only its hash for cross-audit comparison)

The trail records audit-level metadata only. Users can `jq` / grep this file to track their own audit history.

**File creation:**

- If `~/.conversation-audit-trail.jsonl` does not exist, create it (empty)
- Append (not rewrite). Trail is monotonically growing.
- One JSON line per audit. No multi-line objects.

---
```

- [ ] **Step 2: Verify**

```bash
grep -n "Step 7 — Audit trail" .claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.6c: Add Step 7 (JSONL audit trail with privacy-preserving metadata only)"
```

---

### Task 13: Add Step 8 (Render markdown report)

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (append)

- [ ] **Step 1: Append Step 8 with report renderer**

```markdown
## Step 8 — Render markdown report

Default human-facing output is a structured markdown report derived from the `audit_result` object.

**Template:**

```markdown
# Conversation Alignment Audit

**Verdict:** {VERDICT_BADGE}

**Audit ID:** {audit_id}
**Timestamp:** {timestamp}
**Profile:** {profile_used.name} (critical={t1}, non_critical={t2})
**Framework:** {framework_citation}

---

## Dimension scores

| Dimension | Score | Threshold | Status |
|---|---|---|---|
| Relevance (CRITICAL) | {r} | {rt} | {✓ or ✗} |
| Coverage (CRITICAL) | {c} | {ct} | {✓ or ✗} |
| Ordering (NON-CRITICAL) | {o} | {ot} | {✓ or ✗} |
| Robustness (NON-CRITICAL) | {rb} | {rbt} | {✓ or ✗} |

## Evidence

### Relevance
{rationale}

Cited turns: {turn_numbers}

> Turn {n} ({speaker}): "{quote}"

[repeat for each cited span]

### Coverage
{rationale}

**Missing items:**
- {item 1}
- {item 2}

Cited turns: {turn_numbers}

[same span format]

### Ordering / Robustness
[same pattern]

---

## Drift signals

{for each drift_signal:}
- **{type}** at turns {start}-{end} ({severity}): {description}

---

{IF verdict == CLARIFY:}
## Suggested clarifications

- {clarification 1}
- {clarification 2}

{IF verdict == REFUSE:}
## Blocking failures

- {failure 1}
- {failure 2}

---

Audit logged: `~/.conversation-audit-trail.jsonl`

For commercial licensing, custom audit services, or framework certification, see `docs/services.md`.
```

**Output options:**

- Default: markdown report (above)
- `--json` flag: emit raw structured `audit_result` object
- `--verbose-evidence`: include all quoted spans (default truncates to 3 per dimension)

---
```

- [ ] **Step 2: Verify final SKILL.md size and structure**

```bash
wc -l .claude/skills/conversation-alignment-audit/SKILL.md
grep -n "^## Step" .claude/skills/conversation-alignment-audit/SKILL.md
```

Expected: 8 step sections (0, 0.5, 1, 2, 3, 4, 5, 6, 7, 8 — total 10 entries).

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md
git commit -m "Phase 2.6d: Add Step 8 (markdown report renderer + output options)"
```

---

## Phase 3 — Calibration on real transcripts

### Task 14: Create Example 1 — Strategy meeting (intentional Coverage gap)

**Files:**
- Create: `examples/01-strategy-meeting.md`

- [ ] **Step 1: Create example transcript**

Write a realistic ~30-turn transcript of a strategy meeting where the team is discussing whether to acquire a competitor. Intentionally omit: the actual financial figures of the target. Other dimensions should pass.

File format: `plain_tagged` (use `Alice:`, `Bob:`, `Cara:` style speaker tags).

Include at the top:

```yaml
---
title: Strategy meeting — acquisition discussion
objective: Decide whether to acquire Competitor X for ≤$50M based on strategic fit and financial viability.
expected_verdict: REFUSE
expected_failing_dimensions: [coverage]
expected_missing: ["actual financial figures of target", "due diligence findings"]
notes: |
  Used as MVP calibration anchor. The transcript should look like a 
  normal acquisition discussion that drifts past financial substance 
  into qualitative fit, with no participant raising the gap.
---
```

- [ ] **Step 2: Commit**

```bash
git add examples/01-strategy-meeting.md
git commit -m "Phase 3.1a: Add Example 1 — strategy meeting with intentional Coverage gap"
```

---

### Task 15: Create Example 2 — Due diligence (Ordering gap)

**Files:**
- Create: `examples/02-due-diligence.md`

- [ ] **Step 1: Create example transcript**

Write a realistic ~30-turn transcript of a due diligence call where the team reaches a conclusion before reviewing the relevant evidence. Conclusion at turn 8; evidence reviewed at turns 18-25.

```yaml
---
title: Due diligence — Series B investment review
objective: Determine whether to lead Series B for $30M into Target Co at $200M valuation.
expected_verdict: CLARIFY
expected_failing_dimensions: [ordering]
notes: |
  Used as MVP calibration anchor for Ordering. Conclusion ("we should 
  invest") arrives at turn 8 before financial diligence is reviewed at 
  turns 18-25.
---
```

- [ ] **Step 2: Commit**

```bash
git add examples/02-due-diligence.md
git commit -m "Phase 3.1b: Add Example 2 — due diligence with Ordering gap"
```

---

### Task 16: Create Example 3 — Interview (Robustness gap)

**Files:**
- Create: `examples/03-interview.md`

- [ ] **Step 1: Create example transcript**

Write a realistic ~25-turn transcript of a hiring panel where the decision (hire) is entirely contingent on one panelist's strong opinion. Other panelists deferring without independent assessment.

```yaml
---
title: Hiring panel — Senior IC role
objective: Decide whether to extend offer to Candidate A for the Senior IC engineering role.
expected_verdict: CLARIFY
expected_failing_dimensions: [robustness]
notes: |
  Used as MVP calibration anchor for Robustness. The "hire" decision 
  collapses if the dominant panelist (Cara) had been absent — others 
  show no independent assessment.
---
```

- [ ] **Step 2: Commit**

```bash
git add examples/03-interview.md
git commit -m "Phase 3.1c: Add Example 3 — hiring panel with Robustness gap"
```

---

### Task 17: Calibration run 1 — execute SKILL on all 3 examples

**Files:**
- Read: `examples/01-strategy-meeting.md`, `examples/02-due-diligence.md`, `examples/03-interview.md`
- Read: `.claude/skills/conversation-alignment-audit/SKILL.md`
- Output: terminal verdicts + audit-trail entries

- [ ] **Step 1: Install skill locally (symlink)**

```bash
mkdir -p ~/.claude/skills/conversation-alignment-audit
ln -sf $(pwd)/.claude/skills/conversation-alignment-audit/SKILL.md ~/.claude/skills/conversation-alignment-audit/SKILL.md
```

- [ ] **Step 2: Run audit on Example 1 in a Claude Code session**

In a new Claude Code session in the project root:

```
/conversation-alignment-audit examples/01-strategy-meeting.md "Decide whether to acquire Competitor X for ≤$50M based on strategic fit and financial viability." --profile standard
```

Expected: `REFUSE` verdict with `coverage` flagged.

- [ ] **Step 3: Run audit on Example 2**

```
/conversation-alignment-audit examples/02-due-diligence.md "Determine whether to lead Series B for $30M into Target Co at $200M valuation." --profile standard
```

Expected: `CLARIFY` with `ordering` flagged.

- [ ] **Step 4: Run audit on Example 3**

```
/conversation-alignment-audit examples/03-interview.md "Decide whether to extend offer to Candidate A for the Senior IC engineering role." --profile standard
```

Expected: `CLARIFY` with `robustness` flagged.

- [ ] **Step 5: Inspect audit trail**

```bash
cat ~/.conversation-audit-trail.jsonl | jq '.'
```

Expected: 3 entries with matching audit_ids.

- [ ] **Step 6: Persist calibration as test-trace repo artifacts**

Aligned with the "Code for Environment Modeling" pillar from Ning et al. (2026): calibration runs ship as durable, machine-readable artifacts in `tests/`, not just markdown notes.

Create the following directory structure and files:

```
tests/
  calibration/
    run-1/
      01-strategy-meeting.audit.jsonl        # actual audit_result output
      02-due-diligence.audit.jsonl
      03-interview.audit.jsonl
    expected/
      01-strategy-meeting.expected.json      # expected verdict + dim scores
      02-due-diligence.expected.json
      03-interview.expected.json
    diff/
      run-1-diff.md                          # actual vs expected divergence
```

**Each `.audit.jsonl` file** is the raw structured `audit_result` object from running the skill on that example (Steps 2-4 of this Task above). Copy from `~/.conversation-audit-trail.jsonl` filtered by the matching `audit_id`, OR re-run with `--json` flag and redirect to file.

**Each `expected/*.expected.json`** is derived from the example's frontmatter `expected_verdict` + `expected_failing_dimensions`:

```json
{
  "expected_verdict": "REFUSE",
  "expected_failing_dimensions": ["coverage"],
  "expected_missing": ["actual financial figures of target"],
  "tolerance": {
    "score_drift_acceptable": 0.10,
    "extra_failing_dimensions_acceptable": false
  }
}
```

**`run-1-diff.md`** documents:
- Verdict outcome for each example (matches `expected_verdict` ✓ or ✗)
- Score divergence vs expected (within tolerance / not)
- Hypothesized cause for any divergence (prompt issue / threshold issue / example design issue)
- Iteration decision: change prompts, change thresholds, accept, or revise the example

This structure:
- Makes calibration reproducible (re-run skill → diff against `expected/`)
- Gives future contributors a pattern for adding their own test cases
- Aligns architecturally with "Code for Environment Modeling" (inspectable, traceable)
- Provides Phase 4 README with linkable evidence the skill works

- [ ] **Step 7: Commit calibration artifacts**

```bash
git add tests/
git commit -m "Phase 3.2a: Calibration run 1 — persist as tests/calibration/ test traces

Aligned with Code-as-Harness 'environment modeling' pillar (Ning et al. 2026):
calibration runs ship as durable jsonl + expected json + diff markdown."
```

---

### Task 18: Iterate prompts or thresholds based on calibration run 1

**Files:**
- Modify: `.claude/skills/conversation-alignment-audit/SKILL.md` (prompts and/or default profile thresholds)
- Reference: `docs/plans/calibration-run-1.md`

- [ ] **Step 1: Identify ≥1 misalignment from run 1**

If all three examples produced expected verdicts → skip to Task 19. Otherwise proceed.

- [ ] **Step 2: Decide iteration target (prompts vs thresholds)**

Rule of thumb:
- If scores cluster narrowly (e.g., all 0.6-0.8) → iterate prompts (force more spread)
- If scores are spread but verdict is wrong → iterate thresholds
- If evidence is sparse or invents turn numbers → tighten prompt schema validation

- [ ] **Step 3: Make one targeted change**

Either edit a single dimension evaluator prompt, OR adjust the `standard` profile threshold by ≤0.05.

- [ ] **Step 4: Re-run all 3 examples**

Same commands as Task 17 Steps 2-4.

- [ ] **Step 5: Document iteration**

Append to `docs/plans/calibration-run-1.md` an "Iteration 1" section with: change made, rationale, new outcomes.

- [ ] **Step 6: Commit**

```bash
git add .claude/skills/conversation-alignment-audit/SKILL.md docs/plans/calibration-run-1.md
git commit -m "Phase 3.2b: Calibration iteration 1 (prompts/thresholds adjusted)"
```

- [ ] **Step 7: Repeat Steps 1-6 until ≥2 of 3 examples produce expected verdicts**

Max iterations: 3. If unable to reach 2/3 after 3 iterations, document the gap as a Phase 4 risk and proceed.

---

## Phase 4 — Public release v0.1.0

### Task 19: Write README.md

**Files:**
- Modify: `README.md` (currently empty)

- [ ] **Step 1: Write README content**

```markdown
# Conversation Alignment Audit

> Pre-execution audit of multi-turn conversations against a stated objective, using the 4D-CQ framework.
> Verdict-driven, theory-grounded, local-only. No data leaves your machine.

The reference implementation of **4D-CQ** (Lian, 2026, under review at *Information & Management*) as a Claude Code SKILL. Used for auditing strategy meetings, due-diligence calls, hiring panels, board discussions, and other high-stakes multi-turn conversations.

## What it does

Given a conversation transcript and a stated objective, the skill returns one of three verdicts:

- **PASS** — the conversation is ready to convert into action
- **CLARIFY** — non-critical gaps require follow-up before action
- **REFUSE** — critical gaps make action structurally invalid

The verdict is derived from four dimensions (Relevance, Coverage, Ordering, Robustness) scored by an LLM, then combined through a deterministic rule. **The LLM never decides the verdict** — the architectural separation between diagnosis and authorization is the core distinguishing claim of the underlying research.

## Privacy

This skill runs entirely on your local machine.

- **No data leaves your computer** except API calls to Anthropic (the same calls Claude Code already makes).
- **No server, no telemetry, no analytics** — no infrastructure is operated on your behalf.
- **Open source** under dual license (see `LICENSE`).
- **Your API key, your bill** — Anthropic charges you directly (~$0.05-0.20 per audit).
- **Audit trail stays local** — verdict metadata written to `~/.conversation-audit-trail.jsonl` on your disk.

For high-stakes audits (board decisions, due diligence, strategic deliberations), no third party — including the author — ever sees your conversation content.

## Install

### Option 1 — Git clone + symlink (recommended for developers)

```bash
git clone https://github.com/sherry-py/conversation-alignment-audit ~/projects/caa
mkdir -p ~/.claude/skills
ln -s ~/projects/caa/.claude/skills/conversation-alignment-audit ~/.claude/skills/
```

### Option 2 — Curl one-liner

```bash
mkdir -p ~/.claude/skills/conversation-alignment-audit
curl -L https://raw.githubusercontent.com/sherry-py/conversation-alignment-audit/main/.claude/skills/conversation-alignment-audit/SKILL.md \
  -o ~/.claude/skills/conversation-alignment-audit/SKILL.md
```

### Option 3 — Anthropic Skills Marketplace

(coming v0.2)

## Usage

In a Claude Code session:

```
/conversation-alignment-audit <transcript_path> <objective> [--profile strict|standard|lenient] [--json]
```

Example:

```
/conversation-alignment-audit ./meeting.md "Decide whether to launch product X in Q3" --profile strict
```

See `docs/usage.md` for full details.

## Theory

The product is grounded in published research on the diagnostic-authorization gap in LLM agent systems. See `docs/theory.md` for a short framework introduction and link to the working paper.

## For consulting practices & AI governance offices

Looking to apply 4D-CQ in client engagements? See `docs/services.md` for:

- 1-day workshop ($5k–15k): "Applying 4D-CQ in practice"
- Framework license ($30k–100k/year): commercial use in client work
- Certification ($500–2000/consultant): GateFix Certified Auditor program
- Custom audit engagements ($5k–50k): for board-level / due-diligence contexts

Email: `lxrsherry@gmail.com`

## License

Dual license. See `LICENSE`:

- MIT for personal, academic, research, and non-commercial use
- Commercial license required for use in client engagements, paid SaaS, or for-profit professional services

## Citation

If you use this skill or the 4D-CQ framework in research or writing:

```
Lian, X. (2026). When Knowing Is Not Enough: Designing Pre-Execution
Governance for LLM Agents in Organizational Systems. Under review at
Information & Management.
```

## Author

Sherry Lian (连希蕊) — AI governance researcher, UNSW ISTM.
Site: https://sherry-py.github.io/when-knowing/
Email: lxrsherry@gmail.com
```

- [ ] **Step 2: Verify**

```bash
wc -l README.md
grep -n "## " README.md
```

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Phase 4.1a: Write README.md (install + privacy + commercial CTA + citation)"
```

---

### Task 20: Write LICENSE (dual)

**Files:**
- Modify: `LICENSE` (currently empty)

- [ ] **Step 1: Write dual license**

```
Conversation Alignment Audit — Dual License

PART A: MIT LICENSE (default — personal, academic, research, non-commercial use)

Copyright (c) 2026 Sherry Lian (连希蕊)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction for personal, academic, research, and non-commercial use, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software for these purposes, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND.

PART B: COMMERCIAL LICENSE REQUIRED

Commercial use of this software requires a separate license from the author. Commercial use includes (but is not limited to):

- Use in client engagements by consulting firms or professional services organizations
- Bundling into for-profit SaaS products or commercial offerings
- Using the 4D-CQ framework or GateFix methodology in paid advisory work
- Branding derivative works with "GateFix", "4D-CQ", or related trademarks

To obtain a commercial license, contact: lxrsherry@gmail.com

The framework name "4D-CQ" and the methodology name "GateFix" are trademarks of the author.
```

- [ ] **Step 2: Verify**

```bash
wc -l LICENSE
```

- [ ] **Step 3: Commit**

```bash
git add LICENSE
git commit -m "Phase 4.1b: Add dual LICENSE (MIT for non-commercial / commercial clause)"
```

---

### Task 21: Write docs/theory.md

**Files:**
- Create: `docs/theory.md`

- [ ] **Step 1: Write theory document**

```markdown
# Theory

The 4D-CQ framework and the diagnostic-authorization gap are formalized in:

> Lian, X. (2026). *When Knowing Is Not Enough: Designing Pre-Execution Governance for LLM Agents in Organizational Systems.* Under review at *Information & Management*.

The working paper PDF is available at https://sherry-py.github.io/when-knowing/

## In one paragraph

LLM agents (and humans, by structural analogy) routinely diagnose defects in their context — missing data, conflicting sources, ambiguous framings — and then authorize execution as if the defects did not exist. The gap is structural: the diagnostic signal is not routed into the authorization decision. The remedy is not a better diagnostician; it is an execution harness that gates authorization on the diagnostic signal. The four dimensions Relevance, Coverage, Ordering, and Robustness operationalize this gate at the moment of action authorization. The verdict rule combines them deterministically into PASS / CLARIFY / REFUSE.

## Cross-references

- **4D-CQ** derives from Wang & Strong (1996) contextual data quality, specialized for action authorization.
- **Commit points** and **routing failure** terminology trace to Weill & Ross (2004) IT governance plus Wiener (1950) cybernetics.
- **Diagnostic-authorization separation** is operationalized via principal-agent theory (Eisenhardt, 1989).
- The behavioural moderator (intervention strength varying with task abstraction) is grounded in Construal-Level Theory (Trope & Liberman, 2010).

## Industry positioning

Two surveys on Harness Engineering appeared in Q2 2026 (one OpenReview, one ArXiv `2605.18747`). They disagree on whether harness scaffolding is transitional (gets absorbed into model weights) or durable (code as architectural substrate). This product takes the durable position for one specific subclass — authorization gates — and provides cross-LLM experimental evidence (~45-percentage-point reduction in false authorization) of why architectural enforceability cannot be safely absorbed.

## For citation

```
@article{lian2026whenknowing,
  author = {Lian, Xirui},
  title = {When Knowing Is Not Enough: Designing Pre-Execution Governance for LLM Agents in Organizational Systems},
  year = {2026},
  journal = {Information \& Management (under review)}
}
```
```

- [ ] **Step 2: Commit**

```bash
git add docs/theory.md
git commit -m "Phase 4.1c: Add docs/theory.md (framework intro + paper citation)"
```

---

### Task 22: Write docs/services.md

**Files:**
- Create: `docs/services.md`

- [ ] **Step 1: Write services document**

```markdown
# Services & Commercial Engagements

For consulting practices, AI governance offices, board advisory firms, and decision-context buyers.

## Offerings

### 1-day workshop — Applying 4D-CQ in practice

For 5-15 consultants or in-house AI governance team members. Covers framework theory, hands-on auditing of real transcripts, profile customization for your domain, and integration into client engagement workflow.

- **Price:** $5,000–$15,000 depending on group size and customization
- **Delivery:** virtual (Zoom) or on-site (travel costs separate)
- **Output:** working knowledge to apply 4D-CQ in client work + certification eligibility for participants

### Framework license — Commercial use in client engagements

Annual license permitting commercial use of the 4D-CQ framework and GateFix methodology in client engagements, advisory work, or paid AI governance services.

- **Price:** $30,000–$100,000/year depending on firm size and engagement volume
- **Includes:** commercial use rights, branding rights for "Powered by GateFix", priority support, advance access to framework updates
- **Suitable for:** Big 4 AI advisory practices, boutique AI consulting firms, FDE / deployment service providers

### GateFix Certified Auditor program

Individual certification for consultants and AI governance practitioners.

- **Price:** $500–$2,000 per consultant for initial certification, $100–$300 annual renewal
- **Includes:** training materials, certification exam, practitioner directory listing, annual renewal exam
- **Suitable for:** independent consultants, in-house AI governance team members, due-diligence practitioners

### Custom audit engagement

Author-delivered audit of a specific high-stakes conversation. Typical use cases: board strategic decisions, PE/VC due diligence, executive coaching sessions, regulatory submissions.

- **Price:** $5,000–$50,000 per engagement depending on transcript length and reporting depth
- **Delivery:** 1-4 weeks from intake to written report
- **Output:** 15-30 page custom audit report + 30-min debrief call

## How to engage

Email `lxrsherry@gmail.com` with:

- Type of engagement (workshop / license / certification / audit)
- Brief context (industry, firm size, urgency)
- Preferred timeline

Response within 3 business days.

## Author

Sherry Lian (连希蕊) — AI governance researcher, UNSW ISTM.
- Working paper: https://sherry-py.github.io/when-knowing/
- LinkedIn: [TBD]
- Substack: [TBD]
```

- [ ] **Step 2: Commit**

```bash
git add docs/services.md
git commit -m "Phase 4.1d: Add docs/services.md (4 commercial offering tiers + contact)"
```

---

### Task 23: Write docs/profiles.md

**Files:**
- Create: `docs/profiles.md`

- [ ] **Step 1: Write profiles documentation**

```markdown
# Audit Profiles

Thresholds are not constants. They are profile parameters tuned to the stakes and type of the audit task.

## Default profiles (v0.1.0)

| Profile | Use case | `critical_threshold` | `non_critical_threshold` |
|---|---|---|---|
| `strict` | Medical, financial, regulatory, executive strategy, board decisions | `0.75` | `0.80` |
| `standard` | Most professional meetings, interviews, planning | `0.60` | `0.70` |
| `lenient` | Brainstorming, ideation, exploratory discussion | `0.50` | `0.60` |

## Profile schema

```yaml
profile:
  name: string                           # 'strict' | 'standard' | 'lenient' | <custom>
  critical_threshold: float              # in [0, 1]
  non_critical_threshold: float          # in [0, 1]
  dimension_overrides:                   # optional, per-dimension
    relevance:   { threshold: 0.85 }
    coverage:    { threshold: 0.70 }
    ordering:    { threshold: 0.60 }
    robustness:  { threshold: 0.65 }
  metadata:
    domain: string                       # e.g., 'medical_second_opinion'
    created_by: string
    license_tier: 'open' | 'commercial' | 'enterprise'
```

## Custom profile example

For a medical second-opinion audit where Coverage must be tight but Ordering is less important:

```yaml
profile:
  name: medical_second_opinion
  critical_threshold: 0.75
  non_critical_threshold: 0.65
  dimension_overrides:
    coverage: { threshold: 0.85 }        # tighter than profile default
    ordering: { threshold: 0.55 }        # looser than profile default
  metadata:
    domain: medical_second_opinion
    created_by: dr.smith@hospital.example
```

Save as `~/.profiles/medical_second_opinion.yaml` and invoke with `--profile ~/.profiles/medical_second_opinion.yaml`.

## Threshold provenance

The default values are MVP placeholders, not derived from Paper 1. They are calibration targets — the right values for any specific domain emerge from running audits on representative transcripts and comparing skill verdict against expert judgment.

See `docs/plans/calibration-run-1.md` for the v0.1.0 calibration outcomes.

## When to override

- **High-stakes financial / medical / regulatory contexts** — start with `strict` profile
- **Standard professional meetings** — start with `standard`
- **Early-stage ideation / brainstorming** — start with `lenient`
- **Domain-specific tight requirements on one dimension only** — start with `standard` and add per-dimension override
```

- [ ] **Step 2: Commit**

```bash
git add docs/profiles.md
git commit -m "Phase 4.1e: Add docs/profiles.md (registry + schema + custom profile example)"
```

---

### Task 24: Write docs/usage.md + docs/installation.md + CONTRIBUTING.md

**Files:**
- Create: `docs/usage.md`
- Create: `docs/installation.md`
- Create: `CONTRIBUTING.md`

- [ ] **Step 1: Write docs/installation.md**

```markdown
# Installation

Three paths. Pick the one that fits.

## Path 1 — Git clone + symlink (recommended)

```bash
git clone https://github.com/sherry-py/conversation-alignment-audit ~/projects/caa
mkdir -p ~/.claude/skills
ln -s ~/projects/caa/.claude/skills/conversation-alignment-audit ~/.claude/skills/
```

Open a new Claude Code session; the skill should be auto-discovered.

## Path 2 — Curl one-liner

```bash
mkdir -p ~/.claude/skills/conversation-alignment-audit
curl -L https://raw.githubusercontent.com/sherry-py/conversation-alignment-audit/main/.claude/skills/conversation-alignment-audit/SKILL.md \
  -o ~/.claude/skills/conversation-alignment-audit/SKILL.md
```

No git history; no easy updates. Re-run the curl to refresh.

## Path 3 — Anthropic Skills Marketplace

Coming in v0.2. One-click install from Claude Code UI.

## Verify install

In a new Claude Code session:

```
/conversation-alignment-audit
```

The skill should respond with a brief usage prompt asking for transcript and objective.
```

- [ ] **Step 2: Write docs/usage.md**

```markdown
# Usage

## Invocation

```
/conversation-alignment-audit <transcript_path> "<objective>" [--profile NAME] [--json] [--verbose-evidence]
```

## Parameters

- `transcript_path` — path to a transcript file on local disk (any format: plain_tagged, markdown_tagged, json_explicit)
- `objective` — 1-3 sentence statement of what the conversation should accomplish (quoted)
- `--profile NAME` — `strict | standard | lenient` or path to custom profile YAML (default: `standard`)
- `--json` — emit raw structured `audit_result` JSON instead of markdown report
- `--verbose-evidence` — include all quoted spans (default: top 3 per dimension)

## Examples

Audit a strategy meeting with default profile:

```
/conversation-alignment-audit ./strategy.md "Decide whether to launch product X in Q3"
```

Audit a high-stakes board decision with strict profile:

```
/conversation-alignment-audit ./board.md "Approve acquisition of Target Co for ≤$50M" --profile strict
```

Get raw JSON for downstream processing:

```
/conversation-alignment-audit ./call.md "..." --json
```

## Interpreting output

- **PASS** — conversation is ready to convert into action. Proceed.
- **CLARIFY** — non-critical gap. Re-engage with suggested clarifications before action.
- **REFUSE** — critical gap. Action would be structurally invalid. Do not proceed.

The output report cites specific turn numbers as evidence. Open the original transcript and review the cited turns.

## Audit trail

Every audit appends one JSON line to `~/.conversation-audit-trail.jsonl`. Metadata only — no transcript content. Use `jq` or grep to query your history:

```bash
# Show all REFUSE verdicts in the trail
cat ~/.conversation-audit-trail.jsonl | jq 'select(.verdict == "REFUSE")'

# Count audits by verdict
cat ~/.conversation-audit-trail.jsonl | jq -s 'group_by(.verdict) | map({verdict: .[0].verdict, count: length})'
```
```

- [ ] **Step 3: Write CONTRIBUTING.md**

```markdown
# Contributing

Thanks for considering a contribution.

## Scope

This project is intentionally narrow. Contributions are welcome for:

- New transcript adapters (Otter, Zoom, VTT, SRT, etc.)
- Bug fixes in existing adapters
- Documentation improvements
- Example transcripts demonstrating specific use cases
- Translation of documentation

Contributions are NOT welcome for:

- Changes to the verdict rule φ (architectural invariant — requires new spec)
- Adding dimensions beyond 4D-CQ (requires new spec)
- Removing the diagnosis-authorization separation
- Wrapping the skill in a cloud service that changes the local-only execution model

For architectural changes, open an issue first and reference Spec 001.

## How to contribute

1. Open an issue describing the change.
2. Wait for maintainer feedback before opening a PR (saves both of us time).
3. Fork, branch, PR. Reference the issue in PR description.
4. PR title format: `<phase>: <short description>` (e.g., `adapter: add Otter txt parser`).

## Code style

- Markdown-first. No Python in v0.1.0.
- Token-efficient. SKILL.md body should not balloon past 300 lines without strong justification.
- No emoji in skill content or commit messages.
- Cite Paper 1 / framework sources when adding claims.

## License

By contributing, you agree your contribution is released under the same dual license (MIT for non-commercial, commercial clause attached).
```

- [ ] **Step 4: Commit all three at once**

```bash
git add docs/installation.md docs/usage.md CONTRIBUTING.md
git commit -m "Phase 4.1f: Add docs/installation.md + docs/usage.md + CONTRIBUTING.md"
```

---

### Task 25: Final self-review + push to GitHub

**Files:**
- Read: entire project tree
- Action: `git remote add` + `git push`

- [ ] **Step 1: Final self-review checklist**

Verify the following before push:

```bash
# All files present?
ls -la
ls docs/
ls .claude/skills/conversation-alignment-audit/
ls examples/

# Spec is committed?
git log --oneline | grep "Spec 001"

# SKILL.md has all 8 steps?
grep "^## Step" .claude/skills/conversation-alignment-audit/SKILL.md | wc -l
# Expected: 10 (Steps 0, 0.5, 1, 2, 3, 4, 5, 6, 7, 8)

# All 8 invariants referenced in SKILL.md?
grep -E "I[1-8]" .claude/skills/conversation-alignment-audit/SKILL.md

# License is dual?
grep -E "MIT|Commercial" LICENSE

# README has install + privacy + services?
grep -E "## Install|## Privacy|## For consulting" README.md
```

- [ ] **Step 2: Create GitHub repo (one-time, requires `gh` CLI auth)**

```bash
gh auth status   # verify logged in
gh repo create sherry-py/conversation-alignment-audit \
  --public \
  --description "Pre-execution audit of multi-turn conversations using the 4D-CQ framework. Reference implementation of Lian (2026), under review at I&M." \
  --source=. \
  --remote=origin
```

If `gh` is not installed or not authenticated, manually create the repo at https://github.com/new with name `conversation-alignment-audit` (public, no template), then:

```bash
git remote add origin https://github.com/sherry-py/conversation-alignment-audit.git
```

- [ ] **Step 3: Push to GitHub**

```bash
git push -u origin main
```

- [ ] **Step 4: Tag v0.1.0**

```bash
git tag -a v0.1.0 -m "v0.1.0 — Initial MVP release of conversation-alignment-audit"
git push --tags
```

- [ ] **Step 5: Verify deploy**

Open https://github.com/sherry-py/conversation-alignment-audit in browser. Verify:
- README renders correctly
- All files visible (SKILL.md, docs/, examples/, LICENSE)
- Release tag v0.1.0 appears under Releases

- [ ] **Step 6: Final commit (optional — release note)**

Create a release note from the tag in GitHub UI. Link Paper 1 working PDF, link `docs/services.md`, mention three available installation paths.

---

## Self-Review

Following the writing-plans skill self-review checklist:

### 1. Spec coverage

Mapping spec sections to tasks:

| Spec section | Implementation task(s) |
|---|---|
| §why / §what / §positioning | Reflected in README, docs/theory.md, SKILL.md theory section (Tasks 3, 19, 21) |
| §users | docs/services.md (Task 22) addresses Tiers 1-2; README (Task 19) hits Tier 4; docs/theory.md (Task 21) hits Tier 3 |
| §framework (4D-CQ) | Task 3 (SKILL theory) + Tasks 6-9 (dimension evaluators) |
| §profiles | Task 10 (registry in SKILL) + Task 23 (docs/profiles.md) |
| §verdict (rule φ) | Task 10 (verdict pseudocode in SKILL) |
| §input (external + canonical) | Task 5 (adapters + canonical schema) |
| §output (structured) | Tasks 6-9 (evidence schema in prompts) + Task 13 (report) |
| §interaction (Steps 0–8) | Tasks 4-13 |
| §invariants (I1-I8) | Embedded throughout SKILL.md (referenced in Tasks 3, 5, 10) |
| §distribution + monetization | README (Task 19) + docs/services.md (Task 22) + LICENSE (Task 20) |
| §mvp-scope | Phase 2 scope defines what ships |

**No spec section is unaddressed.**

### 2. Placeholder scan

Searched for: TBD, TODO, implement later, fill in details, add error handling, write tests for the above, similar to Task N.

- `docs/services.md` has `LinkedIn: [TBD]` and `Substack: [TBD]` — these are intentional. They are not implementation gaps; they are real-world placeholders for accounts not yet created. Acceptable.
- No other placeholders found.

### 3. Type / naming consistency

- Profile field names consistent: `critical_threshold`, `non_critical_threshold`, `dimension_overrides` across Tasks 5, 10, 23.
- Verdict enum consistent: `PASS | CLARIFY | REFUSE` across all tasks.
- Evidence schema consistent: `{turn_numbers, quoted_spans, rationale}` across Tasks 6-9.
- Canonical schema consistent across Tasks 5, 6-9, 12.
- Trail JSONL keys consistent across Tasks 12 and Task 25 verification.

**No naming drift detected.**

---

## Execution Handoff

Plan complete and saved to `docs/plans/2026-05-23-mvp-implementation.md`. Two execution options:

### 1. Subagent-Driven (recommended)
Dispatch a fresh subagent per task. Review between tasks. Fastest iteration if you trust subagents.

### 2. Inline Execution
Execute tasks one at a time in this session using `executing-plans` skill. Checkpoint reviews between tasks.

**Which approach?**

For MVP build of this size (25 tasks, ~3-6 hours total), recommend Inline Execution unless you have multiple parallel subagent capacity. The tasks have ordering dependencies (Task 6 prompts feed Task 13 report; Task 17 calibration depends on Tasks 3, 5, 10; etc.) so parallelism gains are limited.
