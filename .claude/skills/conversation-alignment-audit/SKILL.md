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
