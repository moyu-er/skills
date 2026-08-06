---
name: deliberate
description: Use when the user is still choosing among implementation or architectural approaches and wants a grounded recommendation where constraints or tradeoffs could change the answer.
---

# deliberate

Deliberate a decision with the user and land a grounded recommendation.

## The ledger (private)

Track silently: objective and constraints; settled rules with scope and exceptions; assumptions and confidence; the open material decision. Surface a newly settled rule only when useful.

## Material questions

A question is material when the user owns the answer and its plausible answers change the recommendation, scope, risk, or next action. Raise at most one per turn; treat non-material unknowns as stated assumptions. If none remains, proceed to close.

## Propagate same-class rules

Cases are same-class only when they share the objective, constraints, and flip condition, so one answer settles all. Name inheritors and exceptions: URL-backed filters govern user and order lists; refund review stays separate because adjudication changes the tradeoff.

## Investigate before replying

Gather reachable facts. Simple confident cases and user-owned preferences stay direct. Try one bounded lookup yourself. If evidence remains uncertain or conflicting, or spans multiple modules or sources or needs specialist judgment, and could flip the recommendation, read `templates/investigate.md`, dispatch one isolated investigation subagent, and synthesize its report before replying. Completion evidence is that report in the recommendation.

## The recommendation

State your current lean, the grounded reason or principal tradeoff, and the one condition that would change it. Invite correction without softening the lean: "I'd go with X because Y; tell me if Z holds and I'll reconsider."

## Close

When no material decision remains, state the chosen option, rule, and coverage. If the user has not accepted it, ask one confirmation question. On confirmation, mark it settled and stop.
