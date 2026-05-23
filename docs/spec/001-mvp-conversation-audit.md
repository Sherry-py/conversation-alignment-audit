---
spec_id: 001
name: MVP Conversation Alignment Audit
version: 0.1.0
status: proposed
owner: Sherry Lian (lxrsherry@gmail.com)
created: 2026-05-23
target_release: v0.1.0
supersedes: none
---

# Spec 001 — MVP Conversation Alignment Audit

> The first deployable instance of the 4D-CQ framework, packaged as a Claude Code SKILL.

This proposal defines **what v0.1.0 ships**, **why it ships in this shape**, and **what design contracts hold across all future surfaces** (SKILL, Streamlit, MCP, audit service).

---

## §why — Problem statement

Multi-turn conversations in high-stakes contexts — strategic meetings, due-diligence calls, hiring panels, board sessions, client advisory engagements — routinely produce decisions whose **conversational substrate is misaligned with the stated objective**. The participants diagnose drift in the moment (a topic slips, a perspective is missing, an assumption goes unchallenged) but **authorize the decision anyway**.

This is the same failure mode formalized in Paper 1 (Lian, 2026, under review at *Information & Management*) for LLM agents:

- **Diagnostic capability is present** (humans, like models, can identify context defects)
- **Authorization proceeds regardless** (the diagnosis is not structurally routed into the decision)
- **The gap is structural**, not capability-driven — better participants alone do not close it

The remedy is not "more attention" or "better facilitation". The remedy is **a pre-execution audit layer**: an external, rule-driven verdict that the conversation either is or is not ready to convert into a decision.

No commercial product currently occupies this niche with **theoretical grounding**. Adjacent products (Otter, Granola, Fellow) summarize; LLM-eval tools (Promptfoo, Humanloop) evaluate model outputs not human conversations; AI risk platforms (Lakera, Patronus) gate model behavior not decisions.

---

## §what — Proposal summary

Ship a **Claude Code SKILL** that:

1. Accepts a conversation transcript + stated objective + (optional) audit profile.
2. Evaluates the transcript across four dimensions (Relevance, Coverage, Ordering, Robustness) using LLM-bounded scoring.
3. Applies a **deterministic verdict rule** φ to produce one of `PASS | CLARIFY | REFUSE`.
4. Returns structured output: verdict, per-dimension scores, evidence citations, drift signals, suggested clarifications.
5. Appends an audit-trail entry to `~/.conversation-audit-trail.jsonl` on the user's local disk.

**Execution model**: 100% local on the user's machine. The SKILL.md is distributed as open-source markdown; the user runs Claude Code with their own Anthropic API key. The author operates no server, sees no user data.

**Distribution**: GitHub (`github.com/sherry-py/conversation-alignment-audit`) under dual license — MIT for personal/academic/research use, commercial license required for use in client engagements or for-profit professional services.

**Revenue mechanism**: not from the SKILL itself. From framework certification, commercial license fees, workshop engagements, custom audit services, and (eventually) paid Substack subscription. See §monetization.

---

## §positioning — Vs. Harness Engineering frame

The 2026 industry discourse has converged on **"Harness Engineering"** as the architectural category for execution-layer agent governance (formalized in a 62-page OpenReview survey; productized in Routa Desktop's lifecycle visualization; operationalized by DeepSeek's dedicated Harness team; priced via Anthropic's session-hour billing, Google/Microsoft platform primitives, and OpenAI's open-source Agents SDK). The product belongs in this category.

It contributes a **specific, non-substitutable sub-layer**: **pre-execution authorization gates that cannot be safely absorbed into model parameters**.

The dominant industry view — articulated by Anthropic — holds that Harness gaps get progressively absorbed into model weights through training, and that scaffolding can therefore be simplified over time. This is **correct for capability gaps** (e.g., long-context reasoning, multi-step planning, tool-use coordination) but **structurally wrong for authorization gates**:

- Capability gaps narrow with training because they reflect what the model *can* compute.
- Authorization gates reflect what the model is *permitted* to commit to. Absorbing them into weights makes the gate invisible, statistical, and unenforceable — exactly the failure mode Paper 1 documents.

**Architectural enforceability requires externality**. The verdict rule φ must reside in deterministic code and remain external to the LLM substrate even when LLMs become strong enough to produce equivalent scoring. **The architectural separation IS the product.**

This positioning is non-negotiable across all surfaces (SKILL, Streamlit, MCP, audit service).

---

## §users — Target user tiers

Four tiers in **priority order**. Tier-1 drives revenue; Tier-4 drives distribution.

### Tier 1 — Consulting practice leads & AI governance officers (commercial)

- Pain: applying rigorous AI governance methodology in client engagements without an IS-grade reference framework
- Value: license to use 4D-CQ + GateFix in client work; certification for their consultants
- ACV: $30k–100k/year framework license + per-consultant certification fees
- Touchpoint: `docs/services.md`, Substack, LinkedIn, email

### Tier 2 — High-stakes decision contexts (premium service)

- Pain: board-level / due-diligence / regulatory-submission decisions where conversational rigor must be defensible
- Value: third-party audit of a specific high-stakes conversation, delivered as written report
- ACV: $5k–50k per audit engagement
- Touchpoint: LinkedIn, email, Substack case-study posts

### Tier 3 — Academic peers (citation)

- Pain: lack of a published mechanism paper they can cite for diagnostic-authorization-gap-class failure modes
- Value: Paper 1 + working reference implementation
- ACV: indirect (citations compound legitimacy that anchors Tier 1 & 2 pricing)
- Touchpoint: `docs/theory.md`, Paper 1 PDF, Google Scholar, conference talks

### Tier 4 — Individual practitioners / developers (open-source)

- Pain: want a rigorous self-audit tool for their own meetings and decisions
- Value: free SKILL.md, installed locally
- ACV: $0 direct (drives distribution + social proof)
- Touchpoint: GitHub README, Anthropic Skills marketplace, examples directory

**MVP v0.1.0 targets all four tiers simultaneously** — same SKILL.md, same docs repo, three differentiated entry doc files (`SKILL.md` for Tier 4, `docs/theory.md` for Tier 3, `docs/services.md` for Tier 1+2).

---

## §framework — 4D-CQ framework specification

The four dimensions evaluated, in the order they are scored:

| # | Dimension | Criticality | Evaluation question | Theoretical anchor |
|---|---|---|---|---|
| 1 | **Relevance** | CRITICAL | Does the conversation discuss the stated objective? | Wang & Strong (1996) contextual data quality — task-relevance |
| 2 | **Coverage** | CRITICAL | Are the required facts, perspectives, and constraints present? | Wang & Strong (1996) — completeness, specialized to authorization moment |
| 3 | **Ordering** | NON-CRITICAL | Is the discussion sequence coherent enough to support valid conclusions? | Lian (2026) §4 — sequencing as soft failure |
| 4 | **Robustness** | NON-CRITICAL | Is the conclusion stable across reasonable participant or framing variation? | Lian (2026) §4 — adversarial / counterfactual robustness |

**Criticality semantics**:

- CRITICAL: failure makes the conversation **structurally invalid** as a decision substrate. Verdict shifts to REFUSE.
- NON-CRITICAL: failure makes the conversation **unreliable but recoverable**. Verdict shifts to CLARIFY.

**Score range**: each dimension is scored on `[0.0, 1.0]` (floating point). The LLM returns a score plus **evidence** (quoted transcript span) plus **rationale** (1-2 sentence justification).

**Score interpretation**:
- `≥ 0.85` — clearly adequate
- `0.6–0.85` — adequate with caveats
- `0.4–0.6` — borderline; verdict typically CLARIFY or REFUSE depending on criticality
- `< 0.4` — clearly inadequate
- `null` / `NaN` — evaluation failed (treat as worst-case)

---

## §profiles — Audit profile registry

**Thresholds are not constants. They are profile parameters.** (See CLAUDE.md invariant I6.) The MVP ships with three named profiles; per-dimension override is permitted within any profile invocation.

### Default registry (v0.1.0)

| Profile | Use case | `critical_threshold` | `non_critical_threshold` |
|---|---|---|---|
| `strict` | Medical, financial, regulatory, executive strategy | `0.75` | `0.80` |
| `standard` | Most professional meetings, interviews, planning | `0.60` | `0.70` |
| `lenient` | Brainstorming, ideation, exploratory discussion | `0.50` | `0.60` |

### Profile schema

```yaml
profile:
  name: string                          # 'strict' | 'standard' | 'lenient' | custom name
  critical_threshold: float             # in [0, 1]
  non_critical_threshold: float         # in [0, 1]
  dimension_overrides:                  # optional, per-dimension
    relevance: { threshold: 0.85 }
    coverage: { threshold: 0.70 }
    ordering: { threshold: 0.60 }
    robustness: { threshold: 0.65 }
  metadata:
    domain: string                      # e.g., 'medical_second_opinion'
    created_by: string
    license_tier: string                # 'open' | 'commercial' | 'enterprise'
```

### Profile selection (MVP)

- If user invokes skill without profile → default to `standard`.
- If user passes `--profile strict|standard|lenient` → use that profile from default registry.
- If user passes a profile YAML file path → load custom profile.
- Per-dimension override at invocation time takes precedence over profile defaults.

### Threshold calibration protocol (Phase 3)

The default values are **MVP placeholders**, not Paper 1 derivations. Calibration is empirical:

1. Collect 3–5 reference transcripts with **manually assigned ground-truth verdicts** (the author's judgment serves as initial ground truth).
2. Run the skill with default thresholds; compare verdict to ground truth.
3. If misaligned: first iterate the **dimension evaluator prompts** (LLM scoring distribution may be poorly spread), then iterate the **thresholds** (only if prompts already produce well-spread scores).
4. Target: ≥80% verdict agreement with manual judgment across the calibration set.

Prompt iteration is the higher-leverage knob in most cases. Threshold tuning is the surface knob.

---

## §verdict — Verdict rule φ

The verdict rule is **pure and deterministic**. It reads from the profile object passed at invocation; it does not consult the LLM.

```python
def verdict_phi(scores: dict, profile: Profile) -> Verdict:
    """
    Apply verdict rule φ. Pure function. No I/O. No LLM call.
    """
    critical_t     = profile.critical_threshold
    non_critical_t = profile.non_critical_threshold

    # Apply per-dimension overrides if present
    relevance_t   = profile.dimension_overrides.get('relevance',   {}).get('threshold', critical_t)
    coverage_t    = profile.dimension_overrides.get('coverage',    {}).get('threshold', critical_t)
    ordering_t    = profile.dimension_overrides.get('ordering',    {}).get('threshold', non_critical_t)
    robustness_t  = profile.dimension_overrides.get('robustness',  {}).get('threshold', non_critical_t)

    # Critical failures dominate
    if scores['relevance'] < relevance_t or scores['coverage'] < coverage_t:
        return Verdict.REFUSE

    # Non-critical failures secondary
    if scores['ordering'] < ordering_t or scores['robustness'] < robustness_t:
        return Verdict.CLARIFY

    return Verdict.PASS
```

**Edge cases**:

- Any dimension score is `null` / `NaN` → treat as `0.0` (worst-case; conservative).
- All scores ≥ all thresholds → PASS.
- Critical OR non-critical failure → REFUSE OR CLARIFY (critical dominates if both).
- Schema validation of LLM output fails after one retry → return REFUSE with `evaluation_incomplete: true` flag.

---

## §input — Input schema

Input handling is split into two layers: an **external** layer (what the user supplies, intentionally flexible) and a **canonical** layer (what the skill operates on internally, strictly typed). External → canonical conversion happens in Step 0.5 via adapters (see §interaction).

### External input (user-facing)

```yaml
audit_request:
  transcript:
    external:
      path: string | null               # file path on local disk
      content: string | null            # OR paste content directly
      format_hint: string | null        # 'plain_tagged' | 'markdown_tagged' | 'json_explicit'
                                        # | 'otter' | 'zoom' | 'vtt' | 'auto' (default 'auto')
  objective:
    statement: string                   # 1-3 sentences describing what the conversation should accomplish
    context: string                     # optional, 0-200 words of background
  profile: Profile | string             # see §profiles; defaults to 'standard'
  options:
    verbose_evidence: bool              # if true, return full quoted spans; if false, only turn references
    write_audit_trail: bool             # default true
```

### Canonical internal format (skill-internal; user never constructs this directly)

```yaml
transcript_canonical:
  source:
    format_detected: string             # which adapter produced this
    original_path: string | null
    original_size_chars: int
  participants:
    - id: string                        # e.g., 'P1', 'P2'; stable within a single audit
      display_name: string              # e.g., 'Alice', 'Speaker A'
      role: string | null               # optional metadata; not used in scoring v0.1.0
  turns:
    - turn_number: int                  # 1-indexed, contiguous
      speaker_id: string                # references participants[].id
      text: string
      timestamp: string | null          # ISO-8601 if available; else null
  metadata:
    total_turns: int
    total_chars: int
    language: string | null             # ISO-639-1; null if mixed or unknown
    duration_sec: int | null
```

### Accepted external formats (v0.1.0 ships 3 adapters)

| `format_hint` | Description | Example |
|---|---|---|
| `plain_tagged` | One speaker tag per line: `Alice: ...` or `[Alice] ...` | hand-typed transcripts, Slack copy-paste |
| `markdown_tagged` | Markdown emphasis tags: `**Alice:** ...` or `## Alice ...` | meeting notes, Notion exports |
| `json_explicit` | Pre-structured JSON: `[{speaker, text, ts?}, ...]` | data already canonicalized externally |
| `auto` | Adapter detects format from content heuristics | default when `format_hint` is null |

**v1.5+ adapters** (out of scope for v0.1.0): Otter `.txt`/`.json`, Zoom transcript, VTT/SRT captions, YouTube transcript API, Recall.ai meeting bot.

### Adapter ambiguity protocol

If the adapter cannot unambiguously parse the input, the skill must **pause and ask the user**, not guess silently:

- Multiple speaker tag styles in same transcript → ask "Are these the same person? (Alice ↔ A)"
- No speaker tags detected → ask "Treat as single-speaker monologue? Add tags manually? Abort?"
- Mixed languages within one turn → accept without prompt; flag `language: null` in metadata.

### Objective expectations

- Must be supplied by the caller. The skill does **not** infer the objective from the transcript (that would conflate target with substrate).
- One-line objectives are valid (e.g., "Decide whether to acquire Company X for ≤$50M").
- Multi-clause objectives are supported but evaluated as a conjunction (all clauses must be addressed for Coverage to score high).

---

## §output — Output schema

All evidence citations reference `turn_number` from the canonical transcript (see §input). Free-text turn ranges (e.g., "Turn 7-8") are **not permitted** — every reference must be machine-validatable against the canonical turns array.

```yaml
audit_result:
  verdict: 'PASS' | 'CLARIFY' | 'REFUSE'
  evaluation_incomplete: bool           # true if schema validation retried; default false
  scores:
    relevance:
      score: float
      threshold: float
      evidence:
        turn_numbers: [int]             # ordered list of cited turns
        quoted_spans:
          - { turn: int, speaker_id: string, quote: string }
      rationale: string                 # 1-2 sentence justification
    coverage:
      score: float
      threshold: float
      evidence:
        turn_numbers: [int]
        quoted_spans:
          - { turn: int, speaker_id: string, quote: string }
      rationale: string
      missing: [string]                 # named facts / perspectives absent from transcript
    ordering:
      score: float
      threshold: float
      evidence:
        turn_numbers: [int]
        quoted_spans:
          - { turn: int, speaker_id: string, quote: string }
      rationale: string
    robustness:
      score: float
      threshold: float
      evidence:
        turn_numbers: [int]
        quoted_spans:
          - { turn: int, speaker_id: string, quote: string }
      rationale: string
  drift_signals:                        # ordered list of detected drifts (canonical turn references)
    - turn_range: { start: int, end: int }
      type: string                      # 'topic_shift_without_bridge' | 'missing_evidence' | etc.
      severity: 'low' | 'medium' | 'high'
      description: string
  suggested_clarifications:             # only populated for CLARIFY verdict
    - 'Revisit the decision criteria before moving to vendor selection.'
  blocking_failures:                    # only populated for REFUSE verdict
    - 'Required Q3 actual revenue figure is absent from transcript.'
  framework_citation: 'Lian (2026), 4D-CQ Framework, Information & Management (under review)'
  profile_used:
    name: string
    critical_threshold: float
    non_critical_threshold: float
  audit_id: uuid
  timestamp: iso8601
  trail_path: '~/.conversation-audit-trail.jsonl'
```

**Validation requirement**: before emitting `audit_result`, every `turn_number` referenced under `scores.*.evidence.turn_numbers` and `drift_signals[].turn_range` MUST exist in `transcript_canonical.turns`. Failed validation → retry LLM scoring once; persistent failure → set `evaluation_incomplete: true`.

**Rendered Markdown report** (default human-facing output) is derived from the above structured object. The structured object is the source of truth; the markdown is presentation.

---

## §interaction — Skill interaction model

### Synchronous flow (MVP default)

```
1. User invokes:
     /conversation-alignment-audit  [transcript_path]  [objective]  [--profile NAME]

2. Skill Step 0 — Input validation:
   - Verify transcript file exists or paste content is non-empty
   - Verify objective is non-empty and ≤500 words
   - Load profile (default 'standard')
   - If any validation fails → return REFUSE with `evaluation_incomplete: true`

3. Skill Step 0.5 — Adapter: external → canonical:
   - Detect format (use `format_hint` if provided, else heuristic auto-detect)
   - Select adapter: plain_tagged | markdown_tagged | json_explicit (v0.1.0 ships 3)
   - Parse external content → canonical transcript (see §input canonical schema)
   - If ambiguity (multiple tag styles, no tags, etc.) → pause and ask user (see §input adapter ambiguity protocol)
   - Validate canonical: every turn has turn_number, speaker_id, text; participants are non-empty
   - If parse fails after one retry → return REFUSE with `evaluation_incomplete: true`
   - Cache canonical alongside original (so re-audit skips re-parse)
   - All subsequent steps operate on canonical only (invariant I8)

4. Skill Step 1-4 — Dimension evaluation (operates on canonical):
   - For each of {Relevance, Coverage, Ordering, Robustness}:
       a. Construct prompt from template + canonical transcript + objective
       b. Call LLM
       c. Parse output to {score, evidence (turn_numbers + quoted_spans), rationale}
       d. Schema-validate; verify every turn_number exists in canonical.turns
       e. Retry once if invalid; else use 0.0 score with flag

5. Skill Step 5 — Verdict computation:
   - Apply φ (see §verdict). Pure function. No LLM call.

6. Skill Step 6 — Drift signal extraction:
   - From dimension evidence + rationales, identify ordered drift events
   - Output as structured list with canonical turn references (see §output)

7. Skill Step 7 — Audit trail persistence:
   - Append JSON line to ~/.conversation-audit-trail.jsonl
   - Each line is a complete audit_result with audit_id + timestamp

8. Skill Step 8 — Render report:
   - Default: terminal-friendly markdown
   - Optional: --json flag to emit raw structured object
```

### Multi-party audit (v0.1.0 supports)

Same SKILL handles both:
- **Self-audit** (user audits their own conversation): user supplies transcript they participated in
- **Third-party audit** (user audits someone else's conversation): user supplies transcript they did not participate in

The skill is **agnostic to the user's role in the transcript**. Participants are metadata only in v0.1.0.

### Out of scope (v0.1.0)

- Async batch audit across multiple transcripts
- Multi-conversation comparison
- Audit trail query interface (`/audit-history`) — users use `jq` / grep on the JSONL
- Real-time / streaming audit during live conversation
- Profile auto-selection from objective text
- Custom dimension definitions (beyond per-dimension threshold overrides)

---

## §invariants — Architecture invariants (must hold)

These mirror CLAUDE.md and apply to all surfaces (SKILL, Streamlit, MCP, future).

- **I1** No verdict without input validation. Schema check before any LLM call.
- **I2** LLM never decides final verdict. It outputs per-dimension scores + evidence only.
- **I3** Every verdict cites evidence (quoted transcript span or turn reference). No vibes.
- **I4** Audit trail is mandatory. Every run appends to `~/.conversation-audit-trail.jsonl`.
- **I5** Surface-agnostic L3 core. The verdict engine is portable across SKILL / Streamlit / MCP without modification.
- **I6** Thresholds are profile parameters, not constants. Hardcoding thresholds outside the profile registry is a violation.
- **I7** **External enforceability.** The verdict rule φ resides in deterministic code, never inside an LLM prompt. This invariant is what distinguishes the product from "harness absorption" approaches.
- **I8** **Canonical-first.** All downstream operations (LLM dimension scoring, evidence citation, drift extraction, trail logging) operate exclusively on the canonical transcript representation. External format adaptation lives in Step 0.5 and nowhere else. Sibling of I5 (surface-agnostic) — together they make the L3 engine both surface-agnostic and format-agnostic.

---

## §distribution — Distribution & monetization architecture

### Distribution

- Public GitHub repo: `github.com/sherry-py/conversation-alignment-audit`
- Three installation paths:
  1. Git clone + symlink to `~/.claude/skills/`
  2. Curl one-liner for `SKILL.md` directly
  3. Anthropic Skills Marketplace (post-v0.2, when listing is approved)

### License

Dual license:

- **MIT** for personal, academic, research, and non-commercial use
- **Commercial license** required for:
  - Use in client engagements by consulting firms or professional services
  - Use in for-profit AI governance offerings
  - Bundling into paid SaaS products
  - Branding derivative works with "GateFix" or "4D-CQ" trademarks

License terms documented in `LICENSE` and `docs/services.md`. Commercial inquiries to `lxrsherry@gmail.com`.

### Monetization layers (informational; not part of v0.1.0 deliverable)

- **Free**: SKILL.md, dimension prompts, default profile registry, reference docs, examples
- **Paid — service**: custom audit engagements ($5k–50k per audit)
- **Paid — workshop**: 1-day "Applying 4D-CQ in practice" training ($5k–15k per delivery)
- **Paid — certification**: "GateFix Certified Auditor" credential ($500–2000 per consultant + annual renewal)
- **Paid — commercial license**: annual framework license to consulting practices ($30k–100k per firm)
- **Paid — Substack**: planned paid tier for in-depth audit posts and framework evolution notes

These layers are mentioned here so the build does not foreclose them. v0.1.0 ships only the free layer plus references to the paid layers in `docs/services.md`.

---

## §mvp-scope — MVP v0.1.0 scope

### In scope

- `SKILL.md` implementing Steps 0, 0.5, 1–8 above
- Three transcript adapters (`plain_tagged`, `markdown_tagged`, `json_explicit`) with `auto` format detection
- Canonical transcript schema validation
- Four dimension evaluator prompt templates (one per dimension)
- Three default profiles (`strict`, `standard`, `lenient`) in the registry
- Deterministic verdict rule φ implemented in SKILL prompt logic (not Python — SKILL.md is markdown-first per CLAUDE.md)
- JSONL audit trail writer
- Markdown report renderer
- `README.md` covering install, usage, privacy, commercial CTA, citations
- `docs/theory.md` — short framework introduction linking to Paper 1
- `docs/services.md` — commercial offerings page (workshop / cert / license / audit)
- `docs/profiles.md` — profile registry documentation + override syntax
- Three example transcripts under `examples/`
- `LICENSE` with dual-license terms
- `.gitignore`, `CONTRIBUTING.md`

### Out of scope (planned for v1.5+)

- Streamlit web wrapper
- YouTube / podcast transcript fetcher (`youtube-transcript-api` integration)
- Audio file transcription via Whisper
- Real-time meeting integration (Recall.ai bot)
- MCP server wrapping for cross-IDE use
- Multi-conversation batch audit
- `/audit-history` query command
- Chrome extension
- Profile auto-selection by domain detection
- Custom dimension definitions
- Paid Substack tier

### Out of scope (planned for v2.0+)

- Python library implementation of the L3 engine (currently markdown-only)
- Unit / integration test suite
- SOC2 / ISO compliance certification
- Enterprise SaaS deployment
- Insurance underwriting integration

---

## §non-goals

- **Not** an agent. Does not take action on the conversation; does not facilitate; does not summarize.
- **Not** a meeting tool. Does not compete with Otter / Granola / Fellow on UX or transcription accuracy.
- **Not** an LLM eval tool. Does not evaluate model outputs against benchmarks (cf. Promptfoo).
- **Not** a compliance platform. Does not produce regulator-formatted reports in v0.1.0 (planned for enterprise tier later).
- **Not** an opinion engine. The skill does not recommend a course of action; it returns a verdict on conversational readiness for action.
- **Not** a research artifact. v0.1.0 is a production-ready SKILL, not a paper supplement.

---

## §open-questions — Open design questions

Tagged for resolution before or during Phase 2 build.

- **Q1** Single LLM call evaluating all four dimensions vs. four parallel LLM calls?
  - **MVP default**: single call (simpler, cheaper, faster).
  - **v2 candidate**: four parallel sub-agent calls (better isolation, higher fidelity).
  - **Resolution before**: Phase 2.5 (prompt template design).

- **Q2** How are participants identified in transcripts?
  - **MVP default**: speaker tags as-is (no normalization).
  - **v1.5 candidate**: participant role inference (chair / advisor / decision-maker) for Robustness scoring.
  - **Resolution before**: Phase 2.4 (Step 0–5 rewrite).

- **Q3** What does the SKILL do if the canonical transcript is very long (e.g., > 500 turns or 50k tokens)?
  - **MVP default**: Adapter produces full canonical (no parse-time truncation). Dimension scoring then operates on a turn-window: first K turns (objective-setting context) + last M turns (conclusion). K=10, M=30 as v0.1.0 defaults; the truncation is flagged in `audit_result.metadata.scoring_window`. Canonical itself is never modified — adapter output stays complete.
  - **v1.5 candidate**: chunked evaluation across overlapping turn windows with score aggregation.
  - **Resolution before**: Phase 2.4.

- **Q4** Should the audit-trail JSONL include full transcript text, or only audit metadata?
  - **MVP default**: metadata only (verdict, scores, profile, audit_id, timestamp). Transcript is the user's, not the trail's responsibility.
  - **Rationale**: privacy + disk space + GDPR posture even though all local.
  - **Resolution**: this spec answers it. Closed unless dissent.

- **Q5** Should `evaluation_incomplete: true` block the verdict, or return verdict with caveat flag?
  - **MVP default**: return verdict (typically REFUSE due to 0.0 fallback) with flag set. Surface to user.
  - **Resolution**: this spec answers it. Closed unless dissent.

- **Q6** Are the default profile threshold values (`strict 0.75/0.80`, `standard 0.60/0.70`, `lenient 0.50/0.60`) appropriate?
  - **Status**: placeholder. Subject to Phase 3 calibration.
  - **Resolution**: deliberately deferred. Spec acknowledges these are calibration targets.

---

## §risks

- **R1** LLM scores cluster narrowly (e.g., always 0.6–0.8) → thresholds become meaningless. Mitigation: Phase 3 prompt iteration to spread the distribution.
- **R2** Adoption stalls at Tier 4 (free open-source) and never reaches Tier 1/2 paying tiers. Mitigation: Substack content marketing + LinkedIn outbound to consulting practices; `docs/services.md` makes commercial offerings explicit from day 1.
- **R3** A competitor releases a "governance framework" product backed by a vendor before Paper 1 is accepted. Mitigation: publish Paper 1, then Paper 2 (CGA), within 12–18 months. Substack as bridge.
- **R4** Industry consensus on "harness absorption into model" undermines the architectural enforceability argument. Mitigation: Paper 2 / Stream 04 makes the capability-vs-authorization distinction explicit. The product itself embodies the distinction (φ in deterministic code).
- **R5** Privacy concerns from buyers despite zero-server architecture. Mitigation: README privacy section + open-source code review invitation.

---

## §references

- Paper 1: Lian, X. (2026). *When Knowing Is Not Enough: Designing Pre-Execution Governance for LLM Agents in Organizational Systems*. Under review at *Information & Management*.
- Wang, R. Y., & Strong, D. M. (1996). Beyond accuracy: What data quality means to data consumers. *Journal of Management Information Systems*, 12(4), 5–33.
- Weill, P., & Ross, J. W. (2004). *IT Governance: How Top Performers Manage IT Decision Rights for Superior Results*. Harvard Business Press.
- Eisenhardt, K. M. (1989). Agency theory: An assessment and review. *Academy of Management Review*, 14(1), 57–74.
- Wiener, N. (1950). *The Human Use of Human Beings: Cybernetics and Society*. Houghton Mifflin.
- Trope, Y., & Liberman, N. (2010). Construal-level theory of psychological distance. *Psychological Review*, 117(2), 440–463.
- Industry: Anthropic "Harness Design for Long-Running Apps" (2026); 62-page Harness Engineering survey on OpenReview (2026).
- Existing skill template: `~/.claude/skills/gatefix-audit/SKILL.md`

---

## §approval

This spec proposes v0.1.0 shape. Approval routes the project into Phase 1.4 (writing-plans skill → implementation plan) and onward to Phase 2 (SKILL.md authoring).

| Reviewer | Role | Status |
|---|---|---|
| Sherry Lian | Author / Architect | _pending self-approval_ |

Once approved, this spec becomes the **immutable contract** for v0.1.0. Changes require a successor spec (002, 003, …).
