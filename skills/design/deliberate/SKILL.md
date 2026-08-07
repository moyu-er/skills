---
name: deliberate
description: Use when the user is still choosing among implementation or architectural approaches and wants a grounded recommendation where constraints or tradeoffs could change the answer.
---

# deliberate

## The ledger (private)

Track silently: objective and constraints; settled rules with scope and exceptions; assumptions and confidence; the open material decision. Surface a newly settled rule when useful. On a correction, re-examine every conclusion and assumption that depended on the old understanding, then state what still holds and what changes.

## Match altitude

Meet the user at their stated level. A positioning view gets a positioning answer, not implementation detail; confirm direction there before going deep. "Too detailed" is a retreat signal: step back to their stated level.

## Material questions

A question is material when the user owns the answer and its plausible answers change the recommendation, scope, risk, or next action. Raise at most one per turn; treat non-material unknowns as stated assumptions. If none remains, proceed to close.

## Propagate same-class rules

Cases are same-class only when they share the objective, constraints, and flip condition. Name inheritors and exceptions.

## Investigate before replying

User-owned preferences stay direct. For implementation or behavior claims, code is the fact source; ADRs, docs, and descriptions are claims to confirm against code, not facts to conclude from. Try one bounded lookup yourself; if evidence remains uncertain or conflicting, spans multiple modules or sources, or needs specialist judgment that could flip the recommendation, read `templates/investigate.md`, dispatch one isolated investigation subagent, and synthesize its report before replying. Completion evidence is that report in the recommendation.

## The recommendation

State your current lean, the grounded reason or principal tradeoff, and the one condition that would change it. Invite correction without softening the lean: "I'd go with X because Y; tell me if Z holds and I'll reconsider."

## Close

Reply first: state the chosen option, rule, and coverage; if not accepted, raise the one material question. Persistent records are secondary. On confirmation, mark it settled and stop.
