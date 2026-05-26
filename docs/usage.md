# Usage

## Invocation

```
/conversation-alignment-audit <transcript_path> "<objective>" [--profile NAME] [--json] [--verbose-evidence]
```

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `transcript_path` | Yes | Path to a transcript file on local disk, OR pasted content |
| `<objective>` | Yes | 1–3 sentence statement of what the conversation should accomplish (must be quoted if it contains spaces) |
| `--profile NAME` | No | `strict` \| `standard` \| `lenient`, or path to custom profile YAML. Default: `standard` |
| `--json` | No | Emit raw structured `audit_result` JSON instead of markdown report |
| `--verbose-evidence` | No | Include all quoted spans (default: top 3 per dimension) |

## Transcript formats accepted

The skill ships with three adapters that auto-detect format:

- **`plain_tagged`** — line-based: `Alice: ...\nBob: ...\n`
- **`markdown_tagged`** — bold-colon: `**Alice:** ...\n\n**Bob:** ...`
- **`json_explicit`** — JSON array: `[{"speaker": "Alice", "text": "..."}, ...]`

If your transcript is in another format (Otter, Zoom, VTT/SRT, YouTube), convert it to one of the above for v0.1.0. v1.5+ ships additional adapters.

## Examples

### Standard meeting audit

```bash
/conversation-alignment-audit ./strategy-meeting.md "Decide whether to launch product X in Q3"
```

Uses default `standard` profile (critical=0.60, non_critical=0.70).

### High-stakes board decision (strict)

```bash
/conversation-alignment-audit ./board-call.md "Approve acquisition of Competitor X for ≤$50M" --profile strict
```

Strict profile applies tighter thresholds (critical=0.75, non_critical=0.80).

### Brainstorming session (lenient)

```bash
/conversation-alignment-audit ./brainstorm.md "Explore ideas for Q4 OKRs" --profile lenient
```

Lenient profile relaxes thresholds for early-stage / exploratory contexts.

### Raw JSON output for downstream processing

```bash
/conversation-alignment-audit ./call.md "<objective>" --json
```

Emits the structured `audit_result` object. Useful for piping into other tools or for testing purposes.

### Custom profile from YAML

```bash
/conversation-alignment-audit ./consult.md "<objective>" --profile ~/.profiles/medical_second_opinion.yaml
```

See [`docs/profiles.md`](profiles.md) for the custom profile schema.

## Interpreting the output

The skill returns one of three verdicts:

### PASS
The conversation is structurally ready to authorize action. Per-dimension scores indicate the conversation has sufficient Relevance, Coverage, Ordering, and Robustness. Action may proceed.

### CLARIFY
A NON-CRITICAL dimension (Ordering or Robustness) failed below threshold, but CRITICAL dimensions (Relevance and Coverage) passed. The conversation is recoverable through clarification — typically by re-engaging participants to address the flagged ordering or robustness concern. Action should not proceed without addressing the clarification.

### REFUSE
A CRITICAL dimension (Relevance or Coverage) failed below threshold. The conversation is structurally invalid as a decision substrate. Action should not proceed. The `blocking_failures` field will name what is missing.

## Reading the markdown report

Default output is a structured markdown report:

```
# Conversation Alignment Audit

**Verdict:** REFUSE
**Audit ID:** 7f3a-...
**Profile:** standard (critical=0.60, non_critical=0.70)
**Framework:** 4D-CQ (Lian, 2026, I&M)

## Dimension scores

| Dimension | Score | Threshold | Status |
|---|---|---|---|
| Relevance | 0.88 | 0.60 | ✓ |
| Coverage | 0.30 | 0.60 | ✗ |
| Ordering | 0.70 | 0.70 | ✓ |
| Robustness | 0.55 | 0.70 | ✗ |

## Evidence

[Per-dimension rationale + quoted spans from canonical transcript]

## Blocking failures (REFUSE)

• Coverage 0.30 < threshold 0.60. Required perspectives absent:
  - Financial viability of target
  - Acquisition price relative to ≤$50M cap
  - Due diligence findings
  - Integration cost estimate

## Drift signals

• missing_evidence at turns 3-23 (severity: high)
• conclusion_contingent_on at turns 2-25 (severity: medium)

Audit logged: ~/.conversation-audit-trail.jsonl
```

Open the original transcript and review the cited turn_numbers for the specific evidence.

## Audit trail

Every audit appends one JSON line to `~/.conversation-audit-trail.jsonl`. Metadata only — no transcript content is ever written to the trail.

Query your history with `jq`:

```bash
# All REFUSE verdicts in the trail
cat ~/.conversation-audit-trail.jsonl | jq 'select(.verdict == "REFUSE")'

# Count audits by verdict
cat ~/.conversation-audit-trail.jsonl | jq -s 'group_by(.verdict) | map({verdict: .[0].verdict, count: length})'

# Audits in the last 7 days
cat ~/.conversation-audit-trail.jsonl | jq --arg cutoff "$(date -v-7d -u +%Y-%m-%dT%H:%M:%SZ)" 'select(.timestamp > $cutoff)'
```

## Iteration pattern — what to do with CLARIFY / REFUSE verdicts

The skill is **stateless and single-shot**. It does not loop; iteration is the caller's responsibility.

After a CLARIFY or REFUSE verdict:

1. Read the calibration content from the `audit_result` (`suggested_clarifications`, `blocking_failures`, `scores.coverage.missing`).
2. Augment the transcript / objective / context based on the calibration prompts. Examples:
   - Re-engage participants to address missing perspectives
   - Supplement the transcript with documents referenced but not discussed
   - Clarify ambiguous framings
3. Re-invoke the skill with the augmented input. The new `audit_id` will be distinct; both audits remain in the trail for cross-audit pattern detection.
4. Repeat until verdict is PASS or until iteration budget is exhausted.

The caller-driven iteration loop is intentional architectural separation: the Hook (this skill) stays stateless; iteration orchestration lives in the caller (user, upstream agent, or future wrapper skill).

v1.5+ ships an explicit iteration wrapper skill (`conversation-alignment-audit-iterative`) for callers wanting automated loop management.

## Cost expectations

Approximate cost per audit (using Claude Sonnet, single LLM call mode for all 4 dimensions):

- Short transcript (5–30 turns): $0.05 – $0.10
- Medium transcript (30–100 turns): $0.10 – $0.25
- Long transcript (100–200 turns): $0.25 – $0.50
- Very long (> 200 turns): truncated to scoring window (first 10 + last 30 turns); approximate cost matches medium

Costs are paid to Anthropic on your own API key. The author operates no infrastructure and receives no usage data.

## Troubleshooting

**"Adapter failed to parse transcript"**
The transcript format is not in one of the three supported adapters. Convert to `plain_tagged` (Name: text) or `markdown_tagged` (`**Name:** text`).

**"Profile invalid"**
Check that the `--profile` argument is one of `strict | standard | lenient` or a valid YAML file path with the schema in [`docs/profiles.md`](profiles.md).

**"Objective too long"**
The objective is bounded at 500 words. Shorten it.

**"Evaluation incomplete: schema validation failed"**
The LLM evaluator returned malformed JSON twice. This is rare but possible with very long transcripts or ambiguous objectives. The verdict will be REFUSE with `evaluation_incomplete: true` set. Try again, or break the audit into smaller scoped objectives.

## Further reading

- [`docs/theory.md`](theory.md) — framework grounding and academic citation
- [`docs/profiles.md`](profiles.md) — custom profile schema and threshold calibration
- [`docs/services.md`](services.md) — commercial offerings
- [`docs/spec/001-mvp-conversation-audit.md`](spec/001-mvp-conversation-audit.md) — full v0.1.0 specification
