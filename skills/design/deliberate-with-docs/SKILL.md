---
name: deliberate-with-docs
description: Conduct and preserve a technical or design deliberation across the project glossary, qualifying ADRs, and a durable decision record.
disable-model-invocation: true
---

# deliberate-with-docs

## Load and verify before discussing

Load `/deliberate` and `/codify`. Confirm both are in context: `deliberate` carries a private ledger and at most one material question per turn; `codify` owns glossary-only `CONTEXT.md` and the all-three ADR gate. If either is absent, stop and name which.

Drive with `/deliberate`; apply `/codify` as terms or ADR candidates settle. After each write, reply with the recommendation or one material question.

## Three artifact routes

- Canonical terms -> the relevant `CONTEXT.md`.
- All-three-gate durable decisions -> `docs/adr/NNNN-slug.md`.
- Ordinary settled decisions, assumptions, exceptions, final recommendation, and flip conditions -> `docs/deliberations/NNNN-slug.md`. Find the highest number, increment, create the directory lazily. Update as decisions settle, not only at the end. When a settled decision also has an ADR, link it from the record instead of restating the rationale.

## Record shape

    # {NNNN slug}
    ## Objective / constraints
    ## Settled decisions
    ## Assumptions
    ## Exceptions
    ## Recommendation
    ## Flip conditions

## Complete

Close per `/deliberate` (reply first). Then read back every expected file and report each layer as written or deliberately skipped with the reason.
