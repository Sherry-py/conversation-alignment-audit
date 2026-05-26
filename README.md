# Conversation Alignment Audit

> **对焦, 然后 commit.** Like camera autofocus before pressing the shutter — but for the multi-turn conversations that authorize high-stakes decisions.
>
> A **Hook-as-Skill** for pre-execution conversation governance. Reference implementation of the **4D-CQ framework** (Lian, 2026, under review at *Information & Management*).

---

## What it is

A Claude Code SKILL that audits a multi-turn conversation transcript against a stated objective. It returns one of three verdicts:

- **PASS** — the conversation is ready to authorize action
- **CLARIFY** — non-critical gaps require follow-up before action
- **REFUSE** — critical gaps make action structurally invalid

The verdict is derived from four dimensions (Relevance, Coverage, Ordering, Robustness), each scored by an LLM, then combined through a **deterministic rule** in markdown. The LLM never decides the verdict. This architectural separation between *diagnosis* (LLM-scored) and *authorization* (deterministic) is the core distinguishing claim of the underlying research.

If attention is what the model needs **inside** its context window, focus is what the user-AI loop needs **outside** the model — before action.

## What it is **not**

- Not an agent (does not act on the conversation)
- Not a summarizer (does not condense; it evaluates)
- Not a vibes-check (verdict is rule-driven, not LLM-decided)
- Not an orchestrator (does not coordinate multiple agents)

## 30-second example

A strategy meeting transcript that discusses acquiring a competitor:

```
Objective: "Decide whether to acquire Competitor X for ≤$50M based on
            strategic fit AND financial viability."
```

The transcript covers strategic fit extensively — team culture, tech stack, integration risk, competitive timing — and converges on a "let's proceed" decision after 30 turns.

But the transcript **never discusses financial viability**: no revenue figures, no burn rate, no price relative to the $50M cap, no due-diligence findings.

```
VERDICT: REFUSE

Coverage: 0.30 < threshold 0.60 (CRITICAL failure)

Missing perspectives:
  • Financial viability of target (revenue, burn rate, runway)
  • Acquisition price relative to ≤$50M cap
  • Due diligence findings (legal, financial, technical)
  • Integration cost estimate

Suggested next step:
  Re-engage with financial diligence before re-running audit.
  Authorization is structurally unsafe at current Coverage level.

Audit logged: ~/.conversation-audit-trail.jsonl
Framework: 4D-CQ (Lian, 2026, Information & Management)
```

The model can know. The model can even tell you what is wrong. But the model is not, by itself, the place where the decision to act gets routed through the diagnostic signal. **That routing is what this skill does.**

## Real-world example — All-in Podcast moats discussion

The skill also works on naturally-occurring public content. We audited the AI-moats segment of an [All-in Podcast episode](https://www.youtube.com/watch?v=4Gmd5UTF4rk) (March 2026, 8 minutes, 21 turns) against the objective:

> "Form a defensible view on which AI / tech companies have durable moats in the AI era, and why, given the changing dynamics of brands, network effects, hardware, and agentic AI adoption."

**Verdict: CLARIFY** (Coverage 0.62 / Ordering 0.65 / Robustness 0.68; Relevance 0.78). Three findings the audit surfaced:

1. **Premature anchoring at turn 8** — one host's "brands go to zero" thesis arrived before counter-frames were aired; subsequent Tesla / BYD / LVMH examples functioned as confirming evidence rather than testing the anchor.
2. **Missing AI-company-specific moat analysis** — the objective explicitly mentions "AI companies", but the discussion defaulted to tech incumbents *using* AI (Apple, Tesla, Meta, Google) rather than companies whose *product is* AI (OpenAI, Anthropic, Nvidia, Perplexity).
3. **Robustness uneven but not weak** — three of four hosts contributed distinct framings; one host explicitly steelmanned a self-counterargument (exemplary), but the dominant thesis still anchored the conversation.

Full audit output: [`tests/youtube/allin-moats-summary.md`](tests/youtube/allin-moats-summary.md). The audit was run by following each dimension's prompt in `SKILL.md` on the transcript — same engine, same verdict rule, same skill — applied to a real-world conversation rather than a synthetic test case. The skill generalizes.

## Why this matters

Two empirical observations, from independent sources:

- On CORE-Bench Hard, Claude Opus 4.5 in Claude Code scores 95%; the same model in a naive Smolagents configuration scores 42%. The model is identical — the 53-percentage-point delta comes entirely from the harness around it.
- A within-subject experiment across four frontier LLMs and 500 financial reasoning tasks (Lian, 2026) shows that LLM agents accurately diagnose defective context but authorize execution on it anyway. The diagnostic-authorization gap on capable models is roughly 50%. A pre-execution governance harness reduces it to under 5% — a 45-percentage-point reduction.

Both numbers point at the same thing: **ambiguity compounds across every loop. The verdict layer must live outside the model.**

This skill implements that verdict layer for human-multi-turn conversation contexts.

## Privacy

This skill runs entirely on your local machine.

- **No data leaves your computer** except API calls to Anthropic (the same calls Claude Code already makes when you use it).
- **No server, no telemetry, no analytics.** No infrastructure is operated on your behalf.
- **Open source** under dual license — inspect every line in `SKILL.md`.
- **Your API key, your bill.** Anthropic charges you directly (~$0.05–0.20 per audit).
- **Audit trail stays local.** Verdict metadata is written to `~/.conversation-audit-trail.jsonl` on your disk; transcript content is never persisted to the trail.

For high-stakes audits (board decisions, due diligence, strategic deliberations), no third party — including the author — ever sees your conversation content.

## Install

### Option 1 — Git clone + symlink (recommended)

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

Coming with v0.2.

### Verify

Open a new Claude Code session in any directory:

```
/conversation-alignment-audit
```

The skill should respond with a usage prompt asking for transcript and objective.

## Usage

```
/conversation-alignment-audit <transcript_path> "<objective>" [--profile NAME] [--json]
```

Examples:

```bash
# Default profile (standard: critical=0.60, non_critical=0.70)
/conversation-alignment-audit ./meeting.md "Decide whether to launch product X in Q3"

# Strict profile (medical / financial / regulatory / board-level)
/conversation-alignment-audit ./board.md "Approve acquisition for ≤$50M" --profile strict

# Lenient profile (brainstorming / ideation)
/conversation-alignment-audit ./brainstorm.md "Explore ideas for Q4 OKRs" --profile lenient

# Raw JSON output for downstream processing
/conversation-alignment-audit ./call.md "..." --json
```

See [`docs/usage.md`](docs/usage.md) for full parameter reference and `examples/` for three realistic transcripts with expected verdicts.

## The 4D-CQ framework

| Dimension | Criticality | Question | Default threshold |
|---|---|---|---|
| **Relevance** | CRITICAL | Does the conversation discuss the stated objective? | < 0.60 → REFUSE |
| **Coverage** | CRITICAL | Are required facts, perspectives, constraints present? | < 0.60 → REFUSE |
| **Ordering** | NON-CRITICAL | Is the discussion sequence coherent enough to support valid conclusions? | < 0.70 → CLARIFY |
| **Robustness** | NON-CRITICAL | Is the conclusion stable across participant or framing variation? | < 0.70 → CLARIFY |

Thresholds are **profile parameters**, not constants. Three default profiles ship: `strict | standard | lenient`. Per-dimension overrides are supported via custom YAML. See [`docs/profiles.md`](docs/profiles.md).

## Architecture invariants

The skill commits to eight architectural invariants that hold across all surfaces (SKILL, future Streamlit, future MCP):

- **I1** No verdict without input validation
- **I2** LLM never decides final verdict — only outputs per-dimension scores + evidence
- **I3** Every verdict cites evidence (canonical turn references)
- **I4** Audit trail is mandatory
- **I5** Surface-agnostic L3 core
- **I6** Thresholds are profile parameters, not constants
- **I7** **External enforceability** — verdict rule φ resides in deterministic code, never inside an LLM prompt
- **I8** **Canonical-first** — all downstream operations work on the canonical transcript representation, not the raw input

Invariant I7 is the central distinguishing claim of the underlying research: pre-execution authorization gates cannot be safely absorbed into model parameters, because absorbing them makes the gate invisible, statistical, and unenforceable.

## Theory

See [`docs/theory.md`](docs/theory.md) for a short framework introduction and reading list. The full working paper PDF is available at <https://sherry-py.github.io/when-knowing/>.

## For consulting practices, AI governance offices, and high-stakes decision contexts

This skill is the free, open-source reference implementation. For commercial deployment, framework licensing, certification, or custom audit engagements, see [`docs/services.md`](docs/services.md):

- **1-day workshop** — Applying 4D-CQ in practice (5–15 consultants per session, $5k–$15k)
- **Framework license** — commercial use in client engagements ($30k–$100k/year per firm)
- **GateFix Certified Auditor** — individual certification program ($500–$2k per consultant)
- **Custom audit engagement** — author-delivered audit of a specific high-stakes conversation ($5k–$50k per engagement)

Inquiries: `lxrsherry@gmail.com`

## License

Dual license. See [`LICENSE`](LICENSE):

- **MIT** for personal, academic, research, and non-commercial use
- **Commercial license** required for use in client engagements, paid SaaS products, or for-profit professional services. Trademarks "4D-CQ" and "GateFix" are reserved.

## Citation

If you use this skill or the 4D-CQ framework in research, writing, or commercial work:

```bibtex
@article{lian2026whenknowing,
  author  = {Lian, Xirui},
  title   = {When Knowing Is Not Enough: Designing Pre-Execution Governance
             for LLM Agents in Organizational Systems},
  year    = {2026},
  journal = {Information \& Management (under review)}
}
```

## Contributing

Issues and PRs welcome. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for scope and conventions.

This project is intentionally narrow. Contributions are welcome for new transcript adapters, bug fixes, documentation improvements, example transcripts, and translation. Architectural changes (verdict rule φ, dimension definitions, invariants I1–I8) require a successor spec proposal — see [`docs/spec/001-mvp-conversation-audit.md`](docs/spec/001-mvp-conversation-audit.md).

## Status

- **v0.1.0** — current. Audit mode. Single-shot Hook. Stateless. Three transcript adapters. Three default profiles. Verdict rule φ deterministic.
- **v0.2** — planned. Discovery mode (bidirectional inversion: given a context, enumerate objectives the context can authorize).
- **v1.5** — planned. Streamlit web wrapper, YouTube transcript adapter, audit-history query command.

## Author

Sherry Lian (连希蕊) — AI governance researcher, UNSW ISTM.

- Site: <https://sherry-py.github.io/when-knowing/>
- Email: `lxrsherry@gmail.com`

---

> *Make for Agent is, at its core, Make for Responsibility. This skill is the responsibility-routing primitive for pre-execution authorization gates.*
