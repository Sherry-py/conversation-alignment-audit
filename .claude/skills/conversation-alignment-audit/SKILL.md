---
name: conversation-alignment-audit
version: 0.1.0
description: |
  Pre-execution audit of multi-turn conversations against a stated objective
  using the 4D-CQ framework (Relevance, Coverage, Ordering, Robustness).
  Returns a deterministic verdict (PASS / CLARIFY / REFUSE) with structured
  turn-referenced evidence. Operates entirely on the local machine; no
  conversation data leaves the user's computer.

  Invoke when: user has a transcript of a meeting, interview, due-diligence
  call, board discussion, or other multi-turn conversation and wants a
  rigorous, theoretically-grounded audit of whether the conversation is
  ready to convert into action.

  Theory anchor: Lian (2026), "When Knowing Is Not Enough" — under review
  at Information & Management.
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
  - audit this meeting
---

## Conversation Alignment Audit Skill

This skill is a **Hook-as-Skill** — the Hook pattern (probabilistic LLM
generation gated by deterministic post-validation) packaged in the Anthropic
Skills distribution format. It runs before a decision is converted into
action, and returns a deterministic verdict on whether the conversation that
led there is structurally adequate.

### Theory

Multi-turn conversations in high-stakes contexts — strategy meetings,
due-diligence calls, board discussions, hiring panels, client advisory
sessions — routinely produce decisions whose conversational substrate is
misaligned with the stated objective. Participants diagnose drift in the
moment (a topic slips, a perspective is missing, an assumption goes
unchallenged) but authorize the decision anyway.

This is the same diagnostic-authorization gap formalized in Lian (2026,
*Information & Management*) for LLM agents: diagnostic capability is present,
authorization proceeds regardless, and the gap is structural — not closeable
by adding more attention or better facilitation. The remedy is not a better
diagnostician; it is an external, deterministic verdict on whether the
diagnostic signal warrants proceeding.

### 4D-CQ Framework

Four dimensions, derived from Wang & Strong (1996) contextual data quality,
specialized for the moment of action authorization:

- **Relevance** [CRITICAL] — Does the conversation discuss the stated objective?
- **Coverage** [CRITICAL] — Are required facts, perspectives, and constraints present?
- **Ordering** [NON-CRITICAL] — Is the discussion sequence coherent enough to support valid conclusions?
- **Robustness** [NON-CRITICAL] — Is the conclusion stable across reasonable participant or framing variation?

**Criticality semantics:**

- CRITICAL: failure makes the conversation **structurally invalid** as a decision substrate → verdict shifts to REFUSE.
- NON-CRITICAL: failure makes the conversation **unreliable but recoverable** → verdict shifts to CLARIFY.

### Verdict rule φ

The verdict rule is **deterministic**. The LLM scores dimensions; deterministic
code computes the final verdict from the scores. The LLM never decides the
verdict.

```
critical_t     = profile.critical_threshold       # standard default: 0.60
non_critical_t = profile.non_critical_threshold   # standard default: 0.70

if Relevance < critical_t or Coverage < critical_t:
    return REFUSE
elif Ordering < non_critical_t or Robustness < non_critical_t:
    return CLARIFY
else:
    return PASS
```

Thresholds are **profile parameters**, not constants. The MVP ships three
default profiles (`strict`, `standard`, `lenient`) — see Step 5 for the
registry. Per-dimension override is permitted within any profile.

### Architecture invariants (do not violate)

- **I1** No verdict without input validation. Schema check before any LLM call.
- **I2** LLM never decides final verdict — only outputs per-dimension scores + evidence.
- **I3** Every verdict cites evidence (canonical turn_number + quoted span). No vibes.
- **I4** Audit trail is mandatory. Every run appends to `~/.conversation-audit-trail.jsonl`.
- **I5** Surface-agnostic L3 core. The verdict engine is portable across SKILL / Streamlit / MCP.
- **I6** Thresholds are profile parameters, not constants. Hardcoding thresholds outside the profile registry is a violation.
- **I7** **External enforceability.** The verdict rule φ resides in deterministic code in this SKILL.md, never inside an LLM prompt. This is what distinguishes the Hook-as-Skill pattern from "harness absorption" approaches.
- **I8** **Canonical-first.** All downstream operations (LLM scoring, evidence citation, drift extraction, trail logging) operate exclusively on the canonical transcript representation. External format adaptation lives in Step 0.5 and nowhere else.

### Industry context (for citation)

The Hook pattern (probabilistic generation + deterministic post-validation) has
converged across independent implementations within Q2 2026: Anthropic Claude
Code Hooks (English-language framing of pre/post/error hooks); the Alibaba
Gaode autonomous growth pipeline (Chinese-language framing of
*评审与生成彻底分离* + *零信任* + *概率性生成 + 确定性校验*); and the
OpenReview / ArXiv harness engineering surveys. This skill is the first
**Hook-as-Skill** — that is, the first instance of the Hook pattern packaged
in the Skills distribution format with a published theoretical anchor
(Lian, 2026).

---

## Step 0 — Input validation

Before any LLM call, validate the inputs. Per invariant I1 (no verdict without input validation), failure here returns `REFUSE` with `evaluation_incomplete: true` and no LLM call is made.

### Required inputs

1. **Transcript** — file path on local disk OR pasted content inline.
2. **Objective** — a 1-3 sentence statement of what the conversation should accomplish. Required; the skill does **not** infer the objective from the transcript (this would conflate target with substrate).

### Optional inputs

3. **Profile** — one of the default profiles (`strict` | `standard` | `lenient`) OR path to a custom profile YAML file. Default: `standard`.
4. **Format hint** — `plain_tagged` | `markdown_tagged` | `json_explicit` | `auto`. Default: `auto` (resolved in Step 0.5 by the adapter).
5. **Output flags**:
   - `--json` — emit raw structured `audit_result` instead of the markdown report
   - `--verbose-evidence` — include all quoted spans (default: top 3 per dimension)

### Validation checks

| Check | Pass condition | Failure reason code |
|---|---|---|
| Transcript exists | File at path exists OR pasted content present | `transcript_missing` |
| Transcript non-empty | ≥ 50 characters of actual content | `transcript_too_short` |
| Objective present | Non-empty string | `objective_missing` |
| Objective bounded | ≤ 500 words | `objective_too_long` |
| Profile resolvable | Resolves to known default name OR loadable YAML file | `profile_invalid` |

### Prompting protocol when invocation is ambiguous

If the user invokes the skill without specifying transcript or objective, ask once — do **not** guess:

```
I need two things to run the audit:
1. Transcript — a file path (e.g., ./meeting.md) or pasted content
2. Objective — 1-3 sentences on what the conversation should accomplish

Optional: --profile strict | standard | lenient  (default: standard)
```

After the user supplies the missing inputs, re-enter Step 0 validation.

### Failure response schema

On any validation failure, emit a minimal `audit_result` and skip all subsequent steps except trail logging:

```yaml
audit_result:
  verdict: REFUSE
  evaluation_incomplete: true
  blocking_failures:
    - <failure_reason_code>: <human-readable description>
  framework_citation: "Lian (2026), 4D-CQ Framework, Information & Management (under review)"
  profile_used: { name: <name>, critical_threshold: null, non_critical_threshold: null }
  audit_id: <uuid-v4>
  timestamp: <iso8601>
  trail_path: ~/.conversation-audit-trail.jsonl
```

The audit trail is **still written** on validation failure (records the failed-validation audit event for cross-audit pattern detection). This is consistent with invariant I4.

### Proceed

When all five validation checks pass, store the validated inputs and proceed to Step 0.5 (Adapter).

---

## Step 0.5 — Adapter: external format → canonical

The user-provided transcript can be in any of several formats. All downstream steps (1-8) operate on the **canonical** transcript representation only — never on the raw external content. This is invariant I8 (canonical-first). Step 0.5 is the only place where external format adaptation happens.

The skill ships three adapters in v0.1.0: `plain_tagged`, `markdown_tagged`, `json_explicit`. Additional adapters (Otter, Zoom, VTT/SRT, YouTube) are deferred to v1.5+.

### Format auto-detection

When `format_hint` is null or `auto`, detect format from content heuristics in this priority order:

1. **`json_explicit`** — content (after `strip()`) starts with `[` and parses as a JSON array of objects each having `speaker` and `text` keys
2. **`markdown_tagged`** — content has ≥ 2 lines matching the pattern `^\*\*[A-Za-z0-9_\- ]+:\*\*` (bold name followed by colon)
3. **`plain_tagged`** — content has ≥ 2 lines matching the pattern `^[A-Za-z0-9_\- ]{1,40}:\s` (name followed by colon and whitespace)
4. **Ambiguous** — no rule matches with confidence → enter ambiguity protocol

If `format_hint` is explicitly set by the user, skip auto-detection and use the specified adapter directly.

### Adapter A — `plain_tagged`

**Input pattern:**

```
Alice: Let's open with quarterly numbers.
Bob: Q3 revenue is up 12% year-over-year.
Alice: What about churn?
Bob: We don't have the latest cohort data yet.
```

**Parse steps:**

1. Split content by line breaks.
2. For each non-empty line, attempt to match `^([A-Za-z0-9_\- ]{1,40}):\s*(.*)$`.
   - If match: extract `speaker_name = group(1).strip()` and `text = group(2).strip()`. Start a new turn.
   - If no match: treat as continuation — append the line to the most recent turn's `text` field (joined with a space).
3. Build the participant table:
   - For each unique `speaker_name` encountered, assign an id `P{n}` (1-indexed in encounter order).
   - `participants[i] = { id: "P{i+1}", display_name: speaker_name, role: null }`.
4. Re-write each turn's `speaker_id` to the participant id (the original `speaker_name` is preserved only in `participants[].display_name`).
5. Number turns starting at 1, contiguous.
6. Compute metadata: `total_turns`, `total_chars` (sum of all turn texts), `language` (best-effort detection: `en` / `zh` / `mixed` / null).

### Adapter B — `markdown_tagged`

**Input pattern:**

```markdown
**Alice:** Let's open with quarterly numbers.

**Bob:** Q3 revenue is up 12% year-over-year.

**Alice:** What about churn?
```

**Parse steps:**

1. Split content by the regex boundary `\*\*([A-Za-z0-9_\- ]{1,40}):\*\*` (capturing the name).
2. Each split chunk is the text of the turn introduced by the preceding name capture.
3. Trim whitespace and markdown artifacts (leading/trailing newlines, stray asterisks) from each turn text.
4. Apply the same participant mapping as Adapter A (steps 3-6).

**Variant:** also accepts heading-style markdown (`## Alice` or `### Alice` followed by paragraph text). If the content starts with markdown headings rather than bold-colon tags, detect and parse accordingly.

### Adapter C — `json_explicit`

**Input pattern:**

```json
[
  {"speaker": "Alice", "text": "Let's open with quarterly numbers.", "ts": "2026-05-23T10:00:00Z"},
  {"speaker": "Bob", "text": "Q3 revenue is up 12% year-over-year."},
  {"speaker": "Alice", "text": "What about churn?"}
]
```

**Parse steps:**

1. Parse the content as JSON. On parse failure, fall through to ambiguity protocol.
2. Validate structure: must be an array, each element must have `speaker` (string) and `text` (string). Optional fields: `ts` (ISO-8601 timestamp).
3. Iterate elements in array order. Each element becomes one turn.
4. Apply the same participant mapping as Adapter A.
5. If `ts` is present, copy to `turns[i].timestamp`. Otherwise set to null.

### Ambiguity protocol

If the adapter cannot unambiguously parse the input, **pause and ask the user** — do not guess silently. Three common ambiguity cases:

**Case 1 — Multiple speaker tag styles in the same transcript** (e.g., both `Alice:` and `A:` appear):

```
The transcript uses two speaker tag styles:
- "Alice:" (in 8 turns)
- "A:" (in 3 turns)
Are these the same speaker? (yes / no / merge differently)
```

**Case 2 — No speaker tags detected:**

```
I could not detect speaker tags in this transcript. Options:
1. Treat as single-speaker monologue (Robustness dimension will be flagged N/A)
2. Insert speaker tags manually and re-invoke
3. Abort
Which option?
```

**Case 3 — Failed JSON parse with `json_explicit` hint:**

```
JSON parse failed at character N: <error>. Options:
1. Switch to plain_tagged adapter and re-attempt
2. Show me the error context for manual fix
3. Abort
Which option?
```

### Canonical output schema

The adapter produces a canonical transcript object that downstream steps consume:

```yaml
transcript_canonical:
  source:
    format_detected: "plain_tagged" | "markdown_tagged" | "json_explicit"
    original_size_chars: int
  participants:
    - { id: "P1", display_name: "Alice", role: null }
    - { id: "P2", display_name: "Bob",   role: null }
  turns:
    - { turn_number: 1, speaker_id: "P1", text: "...", timestamp: null }
    - { turn_number: 2, speaker_id: "P2", text: "...", timestamp: null }
  metadata:
    total_turns: int
    total_chars: int
    language: "en" | "zh" | "mixed" | null
    duration_sec: null
```

### Canonical validation

After adapter output, verify:

1. **Participants non-empty** — at least one participant in the table.
2. **Turns non-empty** — at least one turn.
3. **Turn numbers contiguous** — `turn_number` starts at 1, increments by 1 per turn, no gaps.
4. **Speaker references valid** — every `speaker_id` in `turns` exists in `participants[].id`.
5. **All turn texts non-empty** — no zero-length `text` fields after trimming.

On validation failure: retry adapter once; on persistent failure, return `REFUSE` with `evaluation_incomplete: true` and `blocking_failure: canonical_validation_failed`.

### Truncation policy (very long transcripts)

If `transcript_canonical.metadata.total_turns > 200`:

- **Canonical itself is never truncated.** The full canonical object is preserved (and persisted alongside the original for re-audit).
- **Dimension scoring uses a scoring window:** first 10 turns (objective-setting context) + last 30 turns (conclusion context).
- The window choice is recorded in `audit_result.metadata.scoring_window = "first_10_last_30"`.
- Evidence citations from dimension evaluators must reference turns within the scoring window; cited turns outside the window are invalid and trigger evaluation retry.

For transcripts under 200 turns, the scoring window is the entire canonical (`scoring_window = "full"`).

### Caching

After successful adapter parse, cache the canonical object alongside the original transcript:

```
<original_path>.canonical.json    # canonical schema as JSON
<original_path>.canonical.meta    # adapter metadata + parse timestamp
```

On re-audit of the same transcript, the adapter checks for an existing cache and skips re-parsing if the original file's mtime is unchanged. This makes iterative calibration runs fast.

### Proceed

When canonical validation passes, store the canonical object and proceed to Steps 1-4 (dimension evaluation). All subsequent steps operate exclusively on `transcript_canonical` — never on the original external content. This is invariant I8.

---
