# Recursive Method-Centric AI Workflow Pattern

> Figure 4 in the diagram series.
> ASCII-rendered.
> Companion to [`concept-and-flow.md`](concept-and-flow.md) (Fig 1, 2) and [`capability-map.md`](capability-map.md) (Fig 3).

What does the right AI-in-workflow pattern actually look like? This figure contrasts the **old one-shot paradigm** with the **new recursive method-centric paradigm**, and shows how Hook-as-Skill instantiates the new pattern's central METHOD.

---

## Figure 4 — Recursive Method-Centric Workflow

```
═══════════════════════════════════════════════════════════════════════════
              RECURSIVE METHOD-CENTRIC AI WORKFLOW
═══════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────┐
  │  PANEL 1 — OLD paradigm: AI as one-shot tool                        │
  │                                                                     │
  │            User prompt ──→ AI ──→ output                            │
  │            (single shot, human iterates manually if needed)         │
  │                                                                     │
  │   Properties:                                                       │
  │     ✗ Doesn't scale (every quality check is human work)             │
  │     ✗ Doesn't compound (each invocation independent)                │
  │     ✗ Silent failure (no completion criterion)                      │
  │     ✗ Tied to single output (one prompt = one shot)                 │
  └─────────────────────────────────────────────────────────────────────┘

                                  ▼

  ┌─────────────────────────────────────────────────────────────────────┐
  │  PANEL 2 — NEW paradigm: AI as recursive worker around METHOD       │
  │                                                                     │
  │                                                                     │
  │              ┌────────────────────┐                                 │
  │              │  ① METHOD          │  ← criteria explicit            │
  │              │                    │     ("what counts as done")     │
  │              │  • structured      │                                 │
  │              │  • named           │                                 │
  │              │  • deterministic   │                                 │
  │              │    completion rule │                                 │
  │              └─────────┬──────────┘                                 │
  │                        │ applies to                                 │
  │                        ▼                                            │
  │              ┌────────────────────┐                                 │
  │              │  ② AGENT WORK      │  ← does the substantive work    │
  │              │                    │                                 │
  │              │  • LLM reasoning   │                                 │
  │              │  • tool calls      │                                 │
  │              │  • analysis        │                                 │
  │              └─────────┬──────────┘                                 │
  │                        │ produces output                            │
  │                        ▼                                            │
  │              ┌────────────────────┐                                 │
  │              │  ③ METHOD VERDICT  │  ← method judges output         │
  │              │                    │                                 │
  │              │  Done?             │                                 │
  │              │   Yes → exit       │                                 │
  │              │   No → iterate     │                                 │
  │              └─────────┬──────────┘                                 │
  │                        │ if NOT done                                │
  │                        ▼                                            │
  │              ┌────────────────────┐                                 │
  │              │  ④ RESOURCE PULL   │  ← agent fetches what's missing │
  │              │                    │                                 │
  │              │  • CLI invocation  │                                 │
  │              │  • SDK call        │                                 │
  │              │  • web fetch       │                                 │
  │              │  • re-engage       │                                 │
  │              │    participants    │                                 │
  │              └─────────┬──────────┘                                 │
  │                        │ augments context                           │
  │                        ▼                                            │
  │                  (back to ②)                                        │
  │                  ↺  recurse  ↺                                      │
  │                                                                     │
  │                                                                     │
  │   Properties:                                                       │
  │     ✓ Scales (method does quality check, not human)                 │
  │     ✓ Compounds (each iteration builds on last)                     │
  │     ✓ Loud failure (verdict is explicit)                            │
  │     ✓ Tied to outcome, not single output                            │
  └─────────────────────────────────────────────────────────────────────┘

                                  ▼

  ┌─────────────────────────────────────────────────────────────────────┐
  │  PANEL 3 — Hook-as-Skill instantiates the METHOD                    │
  │                                                                     │
  │                                                                     │
  │  ① METHOD       =  Hook-as-Skill                                    │
  │                    • 4D-CQ structured criteria                      │
  │                    • Verdict rule φ (deterministic completion)      │
  │                    • Calibration prompts (what to fix)              │
  │                                                                     │
  │  ② AGENT WORK   =  caller (human or upstream agent)                 │
  │                    drafts a transcript / decision context           │
  │                                                                     │
  │  ③ VERDICT      =  PASS / CLARIFY / REFUSE                          │
  │                    PASS → exit                                      │
  │                    CLARIFY → re-engage missing perspectives         │
  │                    REFUSE → hold until blocking failures resolved   │
  │                                                                     │
  │  ④ RESOURCE     =  augment context via:                             │
  │     PULL           - re-engaging participants                       │
  │                    - fetching referenced documents                  │
  │                    - generating clarifying questions                │
  │                    - LLM-driven research (v2+)                      │
  │                                                                     │
  │                                                                     │
  │   v0.1.0 status:                                                    │
  │     ✓ ① METHOD             shipped (the SKILL.md)                   │
  │     ✓ ③ VERDICT            shipped (deterministic φ)                │
  │     ✓ Manual ② + ④         supported (caller iterates by hand)      │
  │     ⏳ Auto ② + ④           v1.5 wrapper skill                       │
  │                                                                     │
  │   v1.5 wrapper:  conversation-alignment-audit-iterative             │
  │                  - auto-invokes audit on completion triggers        │
  │                  - generates clarifying questions for missing       │
  │                    perspectives                                     │
  │                  - re-invokes audit on augmented context            │
  │                  - terminates on PASS or iteration budget exhaust   │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## What this pattern requires (and what it does not fit)

The recursive method-centric pattern fits when **all four** properties hold:

1. **Outcome quality matters more than speed.** If a fast wrong answer is acceptable, one-shot is fine.
2. **Method can be cleanly defined.** Vague criteria collapse the loop. The method must be writable as code or as a structured spec.
3. **Completion can be deterministically assessed.** "Looks good" is not a completion criterion. PASS / CLARIFY / REFUSE is.
4. **Resource access can be automated, or is low-friction.** If every iteration requires human authorization, the loop stalls.

It does **not** fit when:

- The task is a quick lookup or simple transform (one-shot is faster)
- The criteria for "done" are inherently subjective (art evaluation, taste)
- Resource access requires high-friction human approval (the loop becomes a meeting)

---

## Industry convergence on this pattern (Q2 2026)

The pattern is being independently re-derived across the field within ~6 months:

- **Karpathy's autoresearch** (Mar 2026): `program.md` (method) + `train.py` (action) + locked evaluator (completion criterion) + auto-loop
- **Anthropic Claude Code main loop**: `while(true)` with `stopHooks` as completion gate (16-step internal loop)
- **DeerFlow 2.0**: orchestrator loop + sub-agents + commit-point check + three-tier memory for state across loops
- **Codex `/goal` CLI**: persistent goal state + Ralph-loop until budget exhausted
- **OpenAI Operator**: task-completion-driven action loop
- **DeepSeek Harness team** (May 2026): "Model + Harness = Agent" — Harness is precisely the method-centric loop infrastructure
- **Jarrod Watts's long-running agent SKILL** (May 2026): /interview phase reduces ambiguity *before* the loop starts ("ambiguity compounds")

Hook-as-Skill sits at the **method** position in this taxonomy: it is the artifact that any of these agent loops can call to determine completion for the *conversation-alignment* sub-problem.

---

## Cross-reference to other diagrams

- [`concept-and-flow.md`](concept-and-flow.md) Figure 1 — architectural intercept (where the Hook sits)
- [`concept-and-flow.md`](concept-and-flow.md) Figure 2 — one semantic-drift case end-to-end (how it runs)
- [`capability-map.md`](capability-map.md) Figure 3 — task inventory across user contexts (what it can do)
- This file Figure 4 — the workflow pattern Hook-as-Skill is the METHOD for (why this shape)

The four figures together form a complete architectural narrative: **where it sits / how it runs / what it can do / why the recursive shape is right**.
