# Theory

The 4D-CQ framework and the diagnostic-authorization gap are formalized in:

> Lian, X. (2026). *When Knowing Is Not Enough: Designing Pre-Execution Governance for LLM Agents in Organizational Systems.* Under review at *Information & Management* (ABS 3).

The working paper PDF and the author's research site are at <https://sherry-py.github.io/when-knowing/>.

---

## In one paragraph

LLM agents — and humans, by structural analogy — routinely diagnose defects in their context (missing data, conflicting sources, ambiguous framings) and then authorize execution as if the defects did not exist. The gap is structural: the diagnostic signal is not, by default, routed into the authorization decision. The remedy is not a better diagnostician; it is an execution harness that gates authorization on the diagnostic signal. The four dimensions — **Relevance, Coverage, Ordering, Robustness** — operationalize this gate at the moment of action authorization. A deterministic verdict rule combines them into PASS / CLARIFY / REFUSE. The combination of structured diagnosis + deterministic verdict is the architectural primitive this product implements.

---

## Why the verdict rule must be deterministic

If the LLM produces the final verdict, the architectural enforceability of the gate collapses the moment the model is incentivized to authorize. This is invariant **I7 (external enforceability)** in this product's spec.

The dominant industry view — articulated in some recent harness engineering surveys — holds that harness scaffolding gets progressively absorbed into model weights through training, and that scaffolding can therefore be simplified over time. This is **correct for capability gaps** (long-context reasoning, multi-step planning, tool-use coordination) but **structurally wrong for authorization gates**:

- Capability gaps narrow with training because they reflect what the model *can* compute.
- Authorization gates reflect what the model is *permitted* to commit to. Absorbing them into weights makes the gate invisible, statistical, and unenforceable — exactly the failure mode Paper 1 documents.

The verdict rule φ in this product lives in deterministic markdown logic, never inside an LLM prompt. The dimension *scoring* uses the LLM (where LLMs are good); the verdict *decision* uses pure code (where determinism is required).

---

## The 4D-CQ dimensions

Derived from Wang & Strong (1996) contextual data quality theory, specialized for the moment of action authorization.

| Dimension | Criticality | Question | Failure → |
|---|---|---|---|
| **Relevance** | CRITICAL | Does the conversation discuss the stated objective? | REFUSE |
| **Coverage** | CRITICAL | Are required facts, perspectives, and constraints present? | REFUSE |
| **Ordering** | NON-CRITICAL | Is the discussion sequence coherent enough to support valid conclusions? | CLARIFY |
| **Robustness** | NON-CRITICAL | Is the conclusion stable across reasonable participant or framing variation? | CLARIFY |

CRITICAL dimensions reflect properties whose failure makes execution structurally invalid. NON-CRITICAL dimensions reflect properties whose failure makes execution unreliable but recoverable.

## Cross-references in the underlying research

- **4D-CQ** derives from Wang & Strong (1996) contextual data quality, specialized for action authorization.
- **Commit points** and **structural routing failure** terminology trace to Weill & Ross (2004) IT governance combined with Wiener (1950) cybernetics.
- **Diagnostic-authorization separation** is operationalized via principal-agent theory (Eisenhardt, 1989).
- The behavioural moderator (intervention strength varying with task abstraction) is grounded in **Construal-Level Theory** (Trope & Liberman, 2010).
- The **Continuous Governance Architecture** thesis (Lian, 2026 working paper Stream 04) frames this product as one instance of a broader architectural pattern combining closed-loop authorization harnesses with discrete commit-point gates.

## Industry positioning

Two surveys on Harness Engineering appeared in Q2 2026 (OpenReview "Agent Systems with Harness Engineering" and ArXiv `2605.18747` "Code as Agent Harness"). They establish "Harness Engineering" as the architectural category for execution-layer agent governance. This product is the first **Hook-as-Skill** — that is, the first instance of the Hook pattern packaged in the Anthropic Skills distribution format with a published theoretical anchor.

The ETCLOVG seven-layer Harness Engineering taxonomy (CMU/Yale/JHU/NEU, 2026) places this product at **Layer G (Governance & Security)** — specifically as an H2 action guardrail — with cross-cutting into **Layer V (Verification & Evaluation)** via the 4D-CQ structured evaluation framework.

## Architectural framing in four layers

The product can be introduced at four levels of abstraction:

1. **Functional**: a Conversation Alignment Audit
2. **Architectural**: a **Hook-as-Skill** (Hook pattern packaged in Skills format)
3. **Thesis**: a **Validated Context Substrate for Agentic Execution**
4. **Architectural dual**: an **External Focus Mechanism** — complement to Transformer's internal attention. If attention is what the model needs *inside* its context window, focus is what the user-AI loop needs *outside* it.

## For BibTeX

```bibtex
@article{lian2026whenknowing,
  author  = {Lian, Xirui},
  title   = {When Knowing Is Not Enough: Designing Pre-Execution Governance
             for LLM Agents in Organizational Systems},
  year    = {2026},
  journal = {Information \& Management (under review)}
}
```

## For further reading

- The working paper PDF (Paper 1, currently under review at *I&M*) — available at <https://sherry-py.github.io/when-knowing/>
- Spec proposal that grounds this product implementation: [`docs/spec/001-mvp-conversation-audit.md`](spec/001-mvp-conversation-audit.md)
- Architecture invariants I1–I8: see project [`CLAUDE.md`](../CLAUDE.md)

## Author

Sherry Lian (连希蕊) — AI governance researcher, UNSW ISTM.
Email: `lxrsherry@gmail.com`.
