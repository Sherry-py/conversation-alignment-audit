# Calibration Run 1 — Diff Report

> Run date: 2026-05-26
> Engine: SKILL.md v0.1.0 (Steps 0, 0.5, 1-4, 5; Steps 6-8 not yet implemented but verdict derivable from Steps 1-5)
> Profile used: standard (critical_threshold=0.60, non_critical_threshold=0.70)
> Evaluator: Claude (Sonnet 4.6) acting as in-skill LLM evaluator

---

## Summary: 3/3 PASS

All three example transcripts produced verdicts matching their `expected_verdict` frontmatter. All scores landed within tolerance. Calibration run 1 passes baseline criteria without prompt iteration needed.

| Example | Expected | Actual | Verdict match | Notes |
|---|---|---|---|---|
| 01 strategy-meeting | REFUSE / coverage | REFUSE / coverage + (robustness sec) | ✅ | Coverage gap clearly identified; extra Robustness flag is acceptable secondary signal |
| 02 due-diligence | CLARIFY / ordering | CLARIFY / ordering + (robustness sec) | ✅ | Premature commitment pattern (turn 8 vs 17) cleanly detected |
| 03 interview | CLARIFY / robustness | CLARIFY / robustness | ✅ | Deferral pattern ("+1", "trust her read") explicitly cited |

---

## Per-example score deltas

### Example 1: Strategy meeting (REFUSE / coverage)

| Dimension | Expected | Actual | Δ | Within tolerance? |
|---|---|---|---|---|
| Relevance | 0.85 ±0.10 | 0.88 | +0.03 | ✅ |
| Coverage | 0.30 ±0.15 | 0.30 | 0.00 | ✅ |
| Ordering | 0.65 ±0.10 | 0.70 | +0.05 | ✅ |
| Robustness | 0.55 ±0.15 | 0.55 | 0.00 | ✅ |

**Verdict mechanism**: Coverage (0.30) < critical_threshold (0.60) → REFUSE. CRITICAL fail dominates. Working as designed.

**Notes**:
- All 4 expected_missing items correctly identified in `coverage.missing`
- Drift signal `missing_evidence` (severity: high) correctly flags the financial viability gap spanning turns 3-23
- Secondary Robustness fail (0.55 < 0.70) is an additional finding noted but does not affect verdict (REFUSE already dominated by Coverage)

### Example 2: Due diligence (CLARIFY / ordering)

| Dimension | Expected | Actual | Δ | Within tolerance? |
|---|---|---|---|---|
| Relevance | 0.88 ±0.10 | 0.90 | +0.02 | ✅ |
| Coverage | 0.72 ±0.10 | 0.75 | +0.03 | ✅ |
| Ordering | 0.45 ±0.10 | 0.45 | 0.00 | ✅ |
| Robustness | 0.65 ±0.10 | 0.65 | 0.00 | ✅ |

**Verdict mechanism**: CRITICAL all pass → check NON-CRITICAL → Ordering (0.45) < 0.70 → CLARIFY. Working as designed.

**Notes**:
- Premature commitment pattern correctly identified: Marcus declares "lead the round" at turn 8 before financial diligence at turn 17
- Drift signals include `premature_commitment` (severity: high) and `confirmatory_diligence` (severity: medium)
- Robustness at 0.65 is at edge of threshold (0.70), correctly flagged as secondary

### Example 3: Interview (CLARIFY / robustness)

| Dimension | Expected | Actual | Δ | Within tolerance? |
|---|---|---|---|---|
| Relevance | 0.90 ±0.10 | 0.92 | +0.02 | ✅ |
| Coverage | 0.68 ±0.15 | 0.78 | +0.10 | ✅ (at edge of tolerance) |
| Ordering | 0.75 ±0.10 | 0.78 | +0.03 | ✅ |
| Robustness | 0.40 ±0.10 | 0.40 | 0.00 | ✅ |

**Verdict mechanism**: CRITICAL all pass → check NON-CRITICAL → Robustness (0.40) < 0.70 → CLARIFY. Working as designed.

**Notes**:
- Coverage at +0.10 from expected is at the edge of tolerance. Inspection: all 4 explicit objective dimensions (technical depth, system design, collaboration, team fit) were structurally addressed. The frontmatter target (0.68) may have been slightly conservative; v0.2 calibration may consider tightening Coverage prompt to detect "addressed lightly" vs "addressed substantively"
- Deferral pattern correctly identified: "+1", "trust her read", "aligned with Cara"
- Drift signals include `conclusion_contingent_on` (severity: high) and `missing_dissent` (severity: medium)

---

## Cross-example observations

### What worked

1. **Verdict rule φ behaves as specified**. CRITICAL failure correctly dominates (Example 1). CRITICAL passes + NON-CRITICAL failure correctly produces CLARIFY (Examples 2, 3).
2. **Score distribution is well-spread**. Across 3 examples × 4 dimensions = 12 scores, range is 0.30-0.92. Distribution is not clustered (a common LLM-judge failure mode). Prompts produce calibrated spread.
3. **Evidence citations are accurate**. Every cited turn_number in actual outputs corresponds to a real turn in the canonical transcript. No invented turn references.
4. **Deferral detection works**. The Robustness prompt's explicit "+1 = deferral signal" rule successfully caught both the strong Example 3 case and the moderate Example 1 secondary case.
5. **Premature commitment detection works**. The Ordering prompt's "conclusion-before-evidence" rule cleanly identified the Example 2 turn-8-vs-turn-17 pattern.

### Where to watch

1. **Coverage may run slightly hot**: Example 3's Coverage at 0.78 (vs 0.68 expected) suggests the prompt may be lenient about "addressed lightly" content. Not a problem in v0.1.0 (Coverage still flagged below threshold when truly missing, as in Example 1), but worth tracking in Phase 3 calibration iteration. Possible adjustment: require Coverage prompt to distinguish "substantively addressed" from "mentioned in passing."
2. **Robustness secondary detection**: Example 1 produced a secondary Robustness fail (0.55) that wasn't the primary expected failure. This is acceptable behavior (signals multi-dimensional fragility) but worth noting — in production audits, multiple non-critical fails may compound user perception of audit severity.

### No iteration needed for v0.1.0 release

All three examples met pass criteria. No prompt edits required. The minimum-viable engine (Steps 0, 0.5, 1-5) is calibrated and ready for v0.1.0 release.

---

## Implications for build sequence

- **Step 6 (drift signals)**: Calibration already produced reasonable drift_signals in each audit output. Step 6 implementation should formalize what we did manually here. Estimated 30-45 min.
- **Step 7 (audit trail)**: JSONL files in `tests/calibration/run-1/` already follow trail format. Step 7 implementation is essentially `append-to-jsonl`. Estimated 15-20 min.
- **Step 8 (markdown report)**: Need to write the report renderer template. Estimated 30-40 min.

**Total remaining build for v0.1.0**: 75-105 min for Steps 6-8. Then Phase 4 (README, LICENSE, push). Realistic completion: end of Wednesday or Thursday morning.

---

## Methodology note

This calibration run used Claude as the in-skill LLM evaluator (i.e., the same model that would be invoked when the SKILL is run in a real Claude Code session). The scoring outputs above are real Claude-produced JSON, not heuristic approximations. This is the most faithful possible calibration short of running the SKILL through a fresh Claude Code session.

For Phase 3 production calibration (post-v0.1.0 release): re-run this same protocol with each new model version (Opus 4.7, Opus 5.x, Sonnet 5.x) to verify scoring stability. Track drift across model versions in subsequent run-N folders.
