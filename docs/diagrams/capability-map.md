# Capability Map

> ASCII-rendered capability inventory for the Conversation Alignment Audit skill.
> Markdown-portable: paste into README, Substack, slides, workshop materials, or any markdown renderer.
> Companion to [`concept-and-flow.md`](concept-and-flow.md) which covers architectural intercept + flow.

The concept and flow diagrams answer "**where does this sit and how does it work**".
This diagram answers "**what can it actually do, for whom, with what outcome**".

---

## Figure 3 — Capability Map

Five-layer bottom-up architectural causality: from user input through the 4D-CQ engine, into structured output artifacts, into the tasks they unlock, into business outcomes per task.

```
═══════════════════════════════════════════════════════════════════════════
        CONVERSATION ALIGNMENT AUDIT — Capability Map
═══════════════════════════════════════════════════════════════════════════

       INPUT (user provides)
   ┌─────────────────────────────────────────────────────────────────┐
   │  📄 Transcript        🎯 Objective         ⚙ Profile           │
   │  (3 formats accepted)  (what should be     (strict / standard / │
   │                         accomplished)        lenient or custom)  │
   └─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
       4D-CQ ENGINE (Hook-as-Skill)
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                  │
   │   🔍 Relevance     LLM scoring + evidence citation              │
   │   📋 Coverage      LLM scoring + missing-item enumeration       │
   │   ⏱  Ordering      LLM scoring + premature-commitment detection │
   │   🌐 Robustness    LLM scoring + counterfactual probing         │
   │                                                                  │
   │              ↓ scores feed into                                 │
   │                                                                  │
   │   ⚖  Verdict rule φ  (deterministic — LLM does NOT decide)     │
   │                                                                  │
   └─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
       OUTPUT (5 artifact types per audit)
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                  │
   │  ① Verdict          PASS / CLARIFY / REFUSE                     │
   │  ② Dimension scores 4 scores + thresholds + per-dim status      │
   │  ③ Evidence         Cited turn_numbers + quoted spans            │
   │  ④ Drift signals    Premature commitment / missing perspective / │
   │                     conclusion contingent on / topic drift       │
   │  ⑤ Calibration      Suggested clarifications / blocking failures │
   │                                                                  │
   │  + Audit trail entry persisted locally (metadata only)           │
   │                                                                  │
   └─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
       TASKS UNLOCKED (5 context categories)
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                  │
   │  💼 STRATEGIC                                                    │
   │     • Board-level acquisition / merger decision audit            │
   │     • Executive strategy off-site readiness check                │
   │     • Investment committee thesis stress-test                    │
   │     • Annual planning kick-off alignment validation              │
   │                                                                  │
   │  💰 DUE DILIGENCE                                                │
   │     • VC / PE term-sheet diligence call audit                    │
   │     • M&A target evaluation conversation check                   │
   │     • Vendor selection committee audit                           │
   │     • Acquisition price-justification review                     │
   │                                                                  │
   │  👥 HIRING & ORG                                                 │
   │     • Senior IC / executive hiring panel debrief audit           │
   │     • Performance review calibration session check                │
   │     • Promotion committee decision audit                         │
   │     • Compensation committee alignment validation                │
   │                                                                  │
   │  🏛 GOVERNANCE & COMPLIANCE                                       │
   │     • Pre-regulatory-submission conversation audit               │
   │     • Risk committee deliberation check                          │
   │     • Audit committee meeting readiness                          │
   │     • Internal AI deployment-governance review                    │
   │                                                                  │
   │  📺 PUBLIC CONTENT (content marketing / research)                │
   │     • Podcast / interview rigor audit                            │
   │     • Conference panel coverage check                            │
   │     • Earnings-call Q&A robustness review                        │
   │     • Public AI debate analysis                                  │
   │                                                                  │
   └─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
       OUTCOMES PER TASK
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                  │
   │  Quality decision substrate  → "PASS, proceed"                  │
   │  Recoverable gap detected    → "CLARIFY, re-engage on X / Y"    │
   │  Structural invalidity       → "REFUSE, hold until Z resolved"  │
   │                                                                  │
   │  + Drift documentation for accountability                        │
   │  + Cross-audit pattern detection over time                       │
   │  + Framework-cited evidence for regulators / boards / academia   │
   │                                                                  │
   └─────────────────────────────────────────────────────────────────┘
```

---

## Five task categories, twenty example use cases

The five categories represent the buyer / use-context spectrum the framework addresses, ordered roughly by ACV and stakes:

### 💼 Strategic (highest stakes, highest ACV)
Board-level decisions, executive strategy, investment-thesis-formation. Audit before commitment.

### 💰 Due Diligence
Pre-investment, pre-acquisition, pre-vendor-selection. Audit the diligence conversation itself, not just the data.

### 👥 Hiring & Org
Decision-quality validation for senior hires, promotions, compensation. Catches the dominant-voice pattern that distorts panels.

### 🏛 Governance & Compliance
Regulatory submissions, risk committees, internal AI deployment reviews. Produces framework-cited audit evidence.

### 📺 Public Content (research + content marketing)
Public conversations (podcasts, panels, earnings calls) audited as content. Generates Substack / academic material and serves as marketing demonstration of framework value.

---

## Usage by surface

| Surface | Suggested use |
|---|---|
| README | After "What it is" — capability overview before Install |
| Substack post | Mid-piece visual anchor for "what does this actually do" |
| Live demo / pitch slide | One slide; reader grasps 5 task categories in 30 seconds |
| Workshop opening | 5-minute orientation; participants understand audit scope before hands-on |
| `docs/services.md` companion | Pairs with commercial-tier descriptions per category |
| LinkedIn carousel | Split into 5 cards — one per task category |

---

## Calibration runs covering these categories

So far, the reference implementation has been calibrated against:

- **Strategic**: `examples/01-strategy-meeting.md` (REFUSE / Coverage gap on acquisition decision)
- **Due Diligence**: `examples/02-due-diligence.md` (CLARIFY / Ordering gap — premature commitment)
- **Hiring & Org**: `examples/03-interview.md` (CLARIFY / Robustness gap — deferral pattern)
- **Public Content**: [`tests/youtube/allin-moats-summary.md`](../../tests/youtube/allin-moats-summary.md) (CLARIFY on All-in podcast moats segment)

Governance & Compliance category is the next calibration anchor target (planned for v0.1.1+).

---

## Cross-reference to other diagrams

- [`concept-and-flow.md`](concept-and-flow.md) Figure 1 — architectural intercept (before/after)
- [`concept-and-flow.md`](concept-and-flow.md) Figure 2 — one semantic-drift case end-to-end
- This file Figure 3 — capability inventory across user contexts

The three figures together cover: **where it sits (Fig 1) · how it runs on one case (Fig 2) · what it can do across contexts (Fig 3)**.
