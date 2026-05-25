# Conversation Alignment Audit — Project Instructions

> LLM-as-judge tool. Input: multi-turn conversation transcript + stated objective.
> Output: alignment verdict (PASS / CLARIFY / REFUSE) + drift map + missing dimensions.

This file is read by Claude Code on every session in this repo. Keep it dense, no fluff.

---

## What this project is

A **pre-execution governance layer** for conversations (human-human or human-AI).
A **SKILL.md product** (markdown-first), distributed via the Claude Code skills marketplace + GitHub.

**Architectural pattern: Hook-as-Skill.** The product packages the **Hook pattern** (probabilistic LLM generation gated by deterministic post-validation, after Anthropic Claude Code hooks and the broader agent-engineering pattern of "probabilistic generation + deterministic validation") using the **Anthropic Skills distribution format**. The Skills format was designed for Capability packaging; this product uses the same format to package Hook semantics. The result is a new conceptual unit — Hook-as-Skill — distributed, composable, fork-friendly, and externalized from any single LLM substrate.

**What it is NOT**:
- Not an agent (does not act on the conversation)
- Not a summarizer (does not condense; it evaluates)
- Not a vibes-check (verdict is rule-driven, not LLM-decided)
- Not an orchestrator (does not coordinate multiple agents; it gates a single decision point)

Theory basis: Paper 1 "When Knowing Is Not Enough" (under review at *Information & Management*, ABS 3). The paper shows LLMs achieve ~92% diagnostic accuracy but a ~41.4pp diagnostic-authorization gap. This product enforces what the paper recommends architecturally.

---

## Core invariant — diagnosis vs authorization separation

**The LLM is only agentic in diagnosis. The final verdict is deterministic.**

- 4D-CQ dimension scoring → LLM-bounded (diagnosis layer, reliable)
- Verdict logic φ → pure rule, LLM never decides

This separation is the central IP. **Non-negotiable across all code, prompts, specs, plans.**

---

## 4D-CQ framework

| Dimension | Criticality | Evaluation question | Default threshold (`standard` profile) |
|---|---|---|---|
| Relevance | CRITICAL | Conversation discusses the stated objective? | < 0.6 → REFUSE |
| Coverage | CRITICAL | Required facts / perspectives complete? | < 0.6 → REFUSE |
| Ordering | NON-CRITICAL | Discussion order coherent? | < 0.7 → CLARIFY |
| Robustness | NON-CRITICAL | Conclusion stable across participant changes? | < 0.7 → CLARIFY |

Verdict rule φ (parameterized by audit profile):
```
critical_t     = profile.critical_threshold       # standard default: 0.6
non_critical_t = profile.non_critical_threshold   # standard default: 0.7

if Relevance < critical_t or Coverage < critical_t:
    return REFUSE
elif Ordering < non_critical_t or Robustness < non_critical_t:
    return CLARIFY
else:
    return PASS
```

### Threshold provenance — **NOT Paper 1 derivations**

The default values `0.6 / 0.7` are MVP placeholders, not derived from Paper 1.
Paper 1 measures system-level accuracy (~92%) and the diagnostic-authorization
gap (~41.4pp); it does not prescribe per-dimension score cutoffs.

**Asymmetry rationale** (`critical < non-critical`): REFUSE is high-cost
(rejects the whole conversation) so it requires stronger evidence — hence the
lower threshold (score must fall further to trigger). CLARIFY is low-cost (just
asks a question) so it triggers more readily.

### Thresholds are audit-profile-dependent

**Thresholds are not global constants. They are profile parameters tuned to the
stakes and type of the audit task.** The L3 engine accepts a profile object on
invocation; the verdict rule reads thresholds from the profile, not from
hardcoded values.

Indicative profile families (registry defined in `docs/spec/001-mvp.md`):

- **High-stakes** (medical diagnosis review, financial decision, executive
  strategy, regulatory submission) → tighter thresholds, less drift tolerance
- **Standard** (most professional meetings, interviews, planning sessions) →
  MVP defaults above
- **Low-stakes** (brainstorming, casual planning, ideation) → looser
  thresholds, more drift tolerance

Per-dimension override is allowed when a specific audit demands it (e.g., a
medical second-opinion audit may need `Coverage` especially tight while
tolerating looser `Ordering`).

Phase 3 calibration may iterate on **prompts** (changing the underlying score
distribution) more than on thresholds (the surface knob). Both are tunable.

---

## Architecture invariants (do not violate)

- **I1** No verdict without input validation (schema check before any LLM call)
- **I2** LLM never decides final verdict — only outputs per-dimension scores + evidence
- **I3** Every verdict cites evidence (quoted transcript span; no vibes)
- **I4** Audit trail mandatory — every run appends to `~/.conversation-audit-trail.jsonl`
- **I5** Surface-agnostic L3 core — same engine reused across SKILL / Streamlit / MCP wrappers
- **I6** Thresholds are **profile parameters**, not constants. Verdict rule φ reads from the profile object passed in at invocation. Hardcoding thresholds anywhere outside the profile registry is a violation.

---

## Project layout

```
docs/spec/    # OpenSpec-style proposals. Write these BEFORE coding.
docs/plans/   # writing-plans skill output (ordered, testable tasks)
.claude/skills/conversation-alignment-audit/SKILL.md   # the deliverable
src/          # v2 Python implementation (MVP does not need)
tests/        # v2 test cases (MVP calibrates on real transcripts, not unit tests)
examples/     # demo transcripts
```

---

## Workflow

1. **Spec first** — every non-trivial change starts with a proposal in `docs/spec/`. Spec covers WHAT and WHY, not HOW.
2. **Plan second** — produce `docs/plans/<date>-<topic>.md` decomposing spec into ordered tasks.
3. **Implement third** — tasks become commits.
4. **Dogfood** — after meaningful changes, run the skill on its own commit summary.

---

## Build conventions

- Markdown over code (SKILL-based product, not a Python service)
- YAML frontmatter on every SKILL.md, spec, and plan
- Token-efficient: SKILL.md body < 200 lines
- Citations required: "research shows X" → cite Paper 1 section
- No defensive code in prompts — trust LLM at dimension layer; gate at deterministic layer
- No emoji in code, prompts, or commit messages

---

## What NOT to do

- Do not let LLM decide the final verdict
- Do not add dimensions beyond 4D-CQ without a spec proposal
- Do not couple L3 engine to a specific transport (Streamlit / MCP / Chrome)
- Do not write unit tests until v2 (MVP calibrates on real transcripts)
- Do not create top-level NOTES.md / IDEAS.md / TODO.md — all docs go under `docs/`
- Do not auto-commit; ask before `git commit`

---

## Reference paths

- Paper 1: `~/Library/Mobile Documents/com~apple~CloudDocs/学术论文/INFS论文/I&M/投稿/When_Knowing_Is_Not_Enough.docx`
- Skill template: `~/.claude/skills/gatefix-audit/SKILL.md`
- Dev plan: `~/Documents/Obsidian Vault/Conversation Alignment Audit · 开发计划.md`
- Public site: `https://sherry-py.github.io/when-knowing/`

---

## Author

Sherry Lian — AI governance researcher, UNSW ISTM. lxrsherry@gmail.com
