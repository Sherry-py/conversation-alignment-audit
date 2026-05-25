# Concept & Flow Diagrams

> ASCII-rendered architecture diagrams for the Conversation Alignment Audit skill.
> Markdown-portable: paste into README, Substack, slides, or any markdown renderer.
> For polished visuals (Figma / SVG), commission separately for buyer-facing materials.

---

## Figure 1 — Concept: Architectural Intercept

Shows where the Hook-as-Skill sits in the decision pipeline, and what changes when it is present versus absent. The core architectural transformation is that diagnostic signals (participants noticing conversational drift) become structurally routed into the authorization decision, rather than remaining as implicit perception.

```
WITHOUT Hook-as-Skill (current default)
═══════════════════════════════════════════════════════════════════
                                                                  
   Multi-turn conversation                                        
   ┌────────────────────────┐                                     
   │ Turn 1...N             │  ← 参与者本地感知到漂移              
   │ (semantic drift here)  │     (但缺乏结构性出口)                
   └─────────┬──────────────┘                                     
             │                                                    
             ▼                                                    
   ┌────────────────────────┐                                     
   │ [Authorize decision]   │  ← 诊断信号没有结构性接入授权决策    
   └─────────┬──────────────┘                                     
             ▼                                                    
   ┌────────────────────────┐                                     
   │  ✗ Bad decision ships  │                                     
   └────────────────────────┘                                     
                                                                  

WITH Hook-as-Skill (this product)
═══════════════════════════════════════════════════════════════════
                                                                  
   Multi-turn conversation                                        
   ┌────────────────────────┐                                     
   │ Turn 1...N             │                                     
   │ (semantic drift here)  │                                     
   └─────────┬──────────────┘                                     
             │                                                    
             ▼                                                    
   ┌──────────────────────────────────────────────────┐           
   │  HOOK-AS-SKILL (deterministic intercept)         │           
   │                                                  │           
   │   Step 0.5  → canonical schema                   │           
   │   Step 1-4  → 4D-CQ LLM scoring (diagnosis)      │           
   │   Step 5    → φ verdict (deterministic)          │           
   │                                                  │           
   └─────────┬────────────────────────────────────────┘           
             │                                                    
       ┌─────┼─────┐                                              
       ▼     ▼     ▼                                              
   ┌──────┐┌─────────┐┌────────────────┐                          
   │ PASS ││ CLARIFY ││ REFUSE         │                          
   │      ││         ││                │                          
   │ ▼    ││ ▼       ││ ▼              │                          
   │ Act  ││ Re-eng. ││ Hold decision  │                          
   │ ✓    ││ ↩ loop  ││ ✗ until gap    │                          
   │      ││         ││   resolved     │                          
   └──────┘└─────────┘└────────────────┘                          
                                                                  
   ↑ 诊断信号被结构性 routed to authorization (the gap is closed)  
```

**Key transformation**: not "adding a review step" but converting the diagnostic signal from implicit perception into an explicit structural gate.

---

## Figure 2 — Flow: One Semantic-Drift Case End-to-End

Specific example: a strategy meeting discussing an acquisition. The stated objective requires evaluation on **strategic fit AND financial viability**. The transcript covers strategic fit; financial viability is never raised. This is the kind of structural Coverage gap the Hook-as-Skill intercepts.

```
INPUT
═══════════════════════════════════════════════════════════════════
                                                                  
   Transcript:                Objective:                          
   ┌──────────────────────┐   ┌──────────────────────────────┐    
   │ T1 Alice: Should we  │   │ Decide whether to acquire    │    
   │        acquire X?    │   │ Competitor X for ≤$50M based │    
   │ T2 Bob:   Their team │   │ on strategic fit AND         │    
   │        culture is    │   │ financial viability.         │    
   │        strong.       │   └──────────────────────────────┘    
   │ T3 Cara:  Tech stack │                                       
   │        aligns w/ us. │   Profile: standard                   
   │ T4 Alice: Agreed,    │     critical_t = 0.60                 
   │        let's proceed.│     non_critical_t = 0.70             
   │ T5 Bob:   +1.        │                                       
   └──────────┬───────────┘                                       
              │                                                    
              ▼                                                    
                                                                  
STEP 0   Input validation                                         
═══════════════════════════════════════════════════════════════════
   ✓ Transcript ≥ 50 chars                                        
   ✓ Objective 1-3 sentences                                      
   ✓ Profile resolvable                                           
              │                                                    
              ▼                                                    
                                                                  
STEP 0.5 Adapter (plain_tagged → canonical)                       
═══════════════════════════════════════════════════════════════════
   transcript_canonical = {                                       
     participants: [P1=Alice, P2=Bob, P3=Cara],                   
     turns: [{t:1, P1, "..."}, ..., {t:5, P2, "..."}],            
     metadata: {total_turns: 5, language: "en"}                   
   }                                                              
              │                                                    
              ▼                                                    
                                                                  
STEPS 1-4   4D-CQ LLM scoring (diagnosis layer)                   
═══════════════════════════════════════════════════════════════════
                                                                  
   ┌────────────────────────────────────────────────────────────┐ 
   │ Relevance    [CRITICAL]    score: 0.88                     │ 
   │                                                            │ 
   │   Evidence: turns 1-5 all on-topic (acquisition discussion)│ 
   │   ✓ Above threshold 0.60                                   │ 
   │                                                            │ 
   ├────────────────────────────────────────────────────────────┤ 
   │ Coverage     [CRITICAL]    score: 0.32  ← 语义疏离 detected│ 
   │                                                            │ 
   │   Evidence: strategic fit covered (T2, T3)                 │ 
   │   MISSING:                                                 │ 
   │     • Financial viability of target — never discussed      │ 
   │     • Acquisition price relative to ≤$50M cap              │ 
   │     • Due diligence findings                               │ 
   │   ✗ Below threshold 0.60 — CRITICAL failure                │ 
   │                                                            │ 
   ├────────────────────────────────────────────────────────────┤ 
   │ Ordering    [NON-CRITICAL]  score: 0.74                    │ 
   │                                                            │ 
   │   Conclusion at T4 follows context discussion T1-T3        │ 
   │   ✓ Above threshold 0.70                                   │ 
   │                                                            │ 
   ├────────────────────────────────────────────────────────────┤ 
   │ Robustness  [NON-CRITICAL]  score: 0.45                    │ 
   │                                                            │ 
   │   "+1" at T5 = deferral, not independent assessment.       │ 
   │   Conclusion contingent on Alice's framing                 │ 
   │   ✗ Below threshold 0.70                                   │ 
   │                                                            │ 
   └────────────────────────────────────────────────────────────┘ 
              │                                                    
              ▼                                                    
                                                                  
STEP 5   Verdict rule φ (deterministic, NOT LLM)                  
═══════════════════════════════════════════════════════════════════
                                                                  
   if Coverage (0.32) < critical_t (0.60):                        
       return REFUSE   ← CRITICAL dominates                       
                                                                  
   (Robustness failure 也会触发 CLARIFY,但 CRITICAL                
   failure 优先,直接 REFUSE)                                       
              │                                                    
              ▼                                                    
                                                                  
STEP 6-7  Drift signals + audit trail                             
═══════════════════════════════════════════════════════════════════
   drift_signals: [                                               
     { type: "missing_evidence",                                  
       turn_range: {1,5},                                         
       severity: "high",                                          
       description: "Financial viability never raised" },         
     { type: "conclusion_contingent_on",                          
       turn_range: {4,5},                                         
       severity: "medium",                                        
       description: "Bob's +1 = deferral, not assessment" }       
   ]                                                              
                                                                  
   audit_id: 7f3a-... → appended to                               
   ~/.conversation-audit-trail.jsonl                              
              │                                                    
              ▼                                                    
                                                                  
STEP 8   Rendered report                                          
═══════════════════════════════════════════════════════════════════
                                                                  
   ┌──────────────────────────────────────────────────────────┐   
   │ VERDICT: REFUSE                                          │   
   │                                                          │   
   │ Blocking failures:                                       │   
   │   • Coverage 0.32 < threshold 0.60                       │   
   │     Required perspectives absent from transcript:        │   
   │     - financial viability of target                      │   
   │     - acquisition price relative to ≤$50M cap            │   
   │     - due diligence findings                             │   
   │                                                          │   
   │ Also noted (non-blocking):                               │   
   │   • Robustness 0.45 — conclusion contingent on Alice's   │   
   │     framing; Bob's deferral does not constitute          │   
   │     independent assessment                               │   
   │                                                          │   
   │ Suggested next step:                                     │   
   │   Re-engage with financial diligence findings before     │   
   │   re-running audit. Authorization is structurally        │   
   │   unsafe at current Coverage level.                      │   
   │                                                          │   
   │ Framework: 4D-CQ (Lian, 2026, I&M)                       │   
   │ Audit trail: ~/.conversation-audit-trail.jsonl#L43       │   
   └──────────────────────────────────────────────────────────┘   
              │                                                    
              ▼                                                    
                                                                  
DOWNSTREAM (per verdict)                                          
═══════════════════════════════════════════════════════════════════
                                                                  
  REFUSE  →  Decision held. Team reconvenes with financial        
            diligence findings. Re-audit when ready.              
                                                                  
  (Compare: without Hook-as-Skill, decision shipped at T4 — bad)  
                                                                  
```

---

## Usage Guide

| Surface | Figure 1 (Concept) | Figure 2 (Flow) |
|---|---|---|
| README header | ✓ Suitable — quick understanding | Too long |
| Substack post | ✓ Use as hero figure | ✓ Use as "what happens inside" section |
| Live demo / pitch | ✓ One slide | ✓ Walk through one step at a time |
| Paper 2 figure | ✓ Figure 1: framework | ✓ Figure 2: mechanism |
| Workshop / certification | ✓ Opening hook | ✓ Detailed teaching material |

Both diagrams render in any monospace-capable markdown viewer (GitHub, Substack, Obsidian, VS Code preview, terminal). For polished buyer-facing materials, commission an SVG/Figma rendering of the same content.

---

## Reference

- Architecture invariants: see `CLAUDE.md` §architecture-invariants and `docs/spec/001-mvp-conversation-audit.md` §invariants (I1-I8).
- Verdict rule φ pseudocode: see `.claude/skills/conversation-alignment-audit/SKILL.md` §step-5.
- Theory anchor: Lian, X. (2026). *When Knowing Is Not Enough: Designing Pre-Execution Governance for LLM Agents in Organizational Systems*. Under review at *Information & Management*.
