# Audit Profiles

Thresholds are not constants. They are profile parameters tuned to the stakes of the audit task. This is invariant **I6** in the spec: hardcoding thresholds outside the profile registry is a violation.

---

## Default profile registry (v0.1.0)

Three profiles ship with the skill:

| Profile | `critical_threshold` | `non_critical_threshold` | Best fit |
|---|---|---|---|
| `strict` | 0.75 | 0.80 | Medical, financial, regulatory, executive strategy, board-level decisions |
| `standard` | 0.60 | 0.70 | Most professional meetings, interviews, planning sessions, due-diligence |
| `lenient` | 0.50 | 0.60 | Brainstorming, ideation, exploratory discussion, low-stakes decisions |

If you do not specify `--profile`, the skill defaults to `standard`.

## Profile schema (custom YAML)

For domain-specific contexts, you can author a custom profile and pass it via `--profile <path-to-yaml>`:

```yaml
profile:
  name: medical_second_opinion
  critical_threshold: 0.75
  non_critical_threshold: 0.65
  dimension_overrides:
    coverage:   { threshold: 0.85 }     # tighter than profile default
    ordering:   { threshold: 0.55 }     # looser than profile default
  metadata:
    domain: medical_second_opinion
    created_by: dr.smith@hospital.example
    license_tier: open
```

Save as `~/.profiles/medical_second_opinion.yaml` and invoke:

```bash
/conversation-alignment-audit ./consult.md "<objective>" --profile ~/.profiles/medical_second_opinion.yaml
```

### Schema fields

- `name` (required) — short identifier, used in audit_result output
- `critical_threshold` (required) — float in `[0.0, 1.0]`, applies to Relevance + Coverage by default
- `non_critical_threshold` (required) — float in `[0.0, 1.0]`, applies to Ordering + Robustness by default
- `dimension_overrides` (optional) — per-dimension threshold override; takes precedence over the profile default
- `metadata` (optional) — informational; recorded in audit trail for cross-audit pattern detection

## Verdict rule φ — how thresholds combine

The deterministic verdict rule reads the profile and applies:

```python
if Relevance < relevance_threshold OR Coverage < coverage_threshold:
    return REFUSE          # CRITICAL dimension fail → REFUSE
elif Ordering < ordering_threshold OR Robustness < robustness_threshold:
    return CLARIFY         # NON-CRITICAL dimension fail → CLARIFY
else:
    return PASS
```

CRITICAL failure dominates: if Coverage fails (REFUSE), the verdict is REFUSE regardless of Ordering or Robustness. The conservative bias is intentional — over-flagging is preferable to under-flagging in pre-execution governance.

## Asymmetric thresholds — why CRITICAL is *lower* than NON-CRITICAL

In all three default profiles, the CRITICAL threshold is *lower* than the NON-CRITICAL threshold (e.g., `standard: 0.60 / 0.70`). This may seem inverted on first read, but reflects **verdict-cost asymmetry**:

- **REFUSE is high-cost** (rejects the whole conversation as a decision substrate) → require stronger evidence to trigger → lower threshold (score must fall further before REFUSE fires)
- **CLARIFY is low-cost** (asks for clarification, does not reject the substrate outright) → trigger more readily → higher threshold

This is the standard pattern in Type I / Type II error tradeoffs: when one error type is costlier than the other, bias the decision rule toward the lower-cost error.

## Threshold provenance — *NOT* Paper 1 derivations

The default values (0.50 / 0.60 / 0.70 / 0.75 / 0.80) are MVP placeholders, not derived from Paper 1's empirical work. Paper 1 measures the diagnostic-authorization gap at the system level (~50% on capable models, reduced to <5% with governance harness); it does not prescribe per-dimension score cutoffs.

The defaults are **calibration targets**. See [`tests/calibration/diff/run-1-diff.md`](../tests/calibration/run-1-diff.md) for the v0.1.0 calibration outcomes on three reference transcripts (all 3 / 3 PASS).

For high-fidelity calibration in your specific domain, run the skill on 5–10 representative transcripts with known expected verdicts (your judgment as ground truth), then adjust thresholds — or, more often, adjust the **dimension evaluator prompts** (the underlying score distribution generator). Prompt iteration is the high-leverage knob; threshold tuning is the surface knob.

## When to override

- **Medical / financial / regulatory** — start with `strict`, may further tighten Coverage to 0.85+ via per-dimension override
- **Standard professional meetings** — `standard` profile
- **Brainstorming / ideation** — `lenient`
- **Mixed stakes** within one decision — use `standard` with per-dimension overrides matching the riskiest required dimension

## License tier metadata

The optional `metadata.license_tier` field is reserved for organizations using the framework under commercial license. The field's allowed values are:

- `open` — personal / academic / non-commercial use (MIT default)
- `commercial` — paid commercial license active
- `enterprise` — extended commercial license with multi-firm or multi-region rights

The skill itself does not enforce license tier (open source); the field exists to support license auditing in commercial deployments and audit trail compliance posture.
