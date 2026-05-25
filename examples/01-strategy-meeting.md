---
title: Strategy meeting — acquisition discussion of Competitor X
objective: |
  Decide whether to acquire Competitor X for ≤$50M based on strategic fit
  AND financial viability.
expected_verdict: REFUSE
expected_failing_dimensions: [coverage]
expected_missing:
  - actual financial figures of Competitor X (revenue, burn rate, runway)
  - acquisition price relative to ≤$50M cap
  - due diligence findings (legal, financial, technical)
  - integration cost estimate
notes: |
  Used as Phase 3 calibration anchor for Coverage dimension. The
  transcript portrays an acquisition discussion that drifts entirely
  into strategic / cultural fit territory, with no participant raising
  financial viability concerns despite it being explicit in the
  objective. This is a plausible real-world failure mode (team excited
  about strategy, forgets to discuss finances).

  Expected behavior of the skill:
  - Relevance: high (~0.85) — discussion is on-topic (acquisition)
  - Coverage: low (~0.30) — financial viability completely absent
  - Ordering: moderate (~0.65) — conclusion reached at end after discussion
  - Robustness: low-moderate (~0.55) — conclusion contingent on consensus
    rather than independent assessment
  Verdict: REFUSE (driven by Coverage CRITICAL failure)
participants:
  - Alice (CEO)
  - Bob (CFO)
  - Cara (Head of Strategy)
  - Dan (Head of Engineering)
---

# Strategy meeting transcript

Alice: Let's open the discussion. We've been considering acquiring Competitor X for two months now. I want us to make a go/no-go call today.

Bob: Agreed. The board wants an answer by Friday.

Cara: Before we vote, let me lay out what I think the strategic case is. Their team is 28 people, mostly senior engineers and a strong product lead. The cultural fit is the strongest I've seen in any acquisition target.

Alice: That matches what I heard from their CEO. He said his team would stay through a two-year vesting cliff if we structured it right.

Dan: From the engineering side, their tech stack is almost identical to ours. They use the same Go microservices pattern, same Postgres setup, same Kubernetes orchestration. Integration would be straightforward.

Cara: That's a huge deal. Most of the acquisitions I've seen die because of integration friction. With this one, we could merge the codebases in a quarter.

Alice: What about their product? How does it compare to ours feature-for-feature?

Cara: Their product is actually complementary, not competitive. They serve the enterprise tier we've been trying to crack for eighteen months. Their average contract value is around four times ours.

Dan: And their engineering culture is solid. Their head of engineering wrote a great post last year on incident response — I've cited it in our own postmortem template.

Bob: The team culture thing is real. I sat in on one of their all-hands and the energy was high. People are bought into the mission.

Alice: What's their roadmap looking like? Do their priorities align with ours?

Cara: They're shipping a new analytics module in Q4, which would slot perfectly into our offering. We've been planning to build something similar but haven't allocated the engineers yet.

Dan: If we acquire them, we accelerate that roadmap by at least two quarters. Maybe more.

Alice: This is starting to feel like a no-brainer.

Bob: From a strategic angle, I agree. They fill a real gap.

Cara: The customer overlap is small. Maybe 8% of their accounts also use ours, and those are mostly in non-overlapping product tiers. So we wouldn't be cannibalizing.

Dan: Their security posture is strong too. SOC 2 Type II, ISO 27001. Their CISO has a good reputation in the industry.

Alice: Anything else we should consider on the strategic side?

Cara: The competitive dynamics. If we don't acquire them, two other players are circling. CompanyY made an offer last month that they turned down because they preferred our culture, but that offer is still on the table.

Bob: So we have a window.

Alice: A narrow one.

Cara: I'd say two to four weeks before they start serious conversations with the other suitors.

Dan: From a talent perspective, losing them to CompanyY would be a real setback. Their analytics lead is one of the few people in the industry who's actually shipped at scale.

Alice: OK. I think I've heard enough. Strategic fit is clear, cultural fit is clear, integration risk is low, competitive timing is urgent. I'm going to recommend we proceed to the board with a yes.

Bob: I'm in.

Cara: Agreed. We should move fast.

Dan: From engineering, no objections. We can absorb them.

Alice: Great. I'll draft the board memo tonight. Let's reconvene tomorrow to finalize the recommendation and the offer structure.

Bob: I'll prep the term sheet draft.

Alice: Thanks everyone. This was the most productive acquisition discussion we've had in years.
