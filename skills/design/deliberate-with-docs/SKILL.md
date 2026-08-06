---
name: deliberate-with-docs
description: Conduct and preserve a technical or design deliberation across the project glossary, qualifying ADRs, and a durable decision record.
disable-model-invocation: true
---

# deliberate-with-docs

User-invoked shell: drive with `/deliberate`, persist across three layers, apply `/codify` inline as terms or ADR candidates settle.

## Load and verify before discussing

Explicitly load `/deliberate` and `/codify`. Confirm both are in context by their distinctive contracts: `deliberate` carries a private ledger and at most one material question per turn; `codify` owns glossary-only `CONTEXT.md` and the all-three ADR gate (hard to reverse, surprising, real tradeoff). If either is absent, stop and name which skill failed to load. Do not claim a load you did not verify.

## Drive

Use `/deliberate` as the conversation driver. Apply `/codify` inline as terms sharpen or ADR candidates appear. Do not restate their instructions here.

## Three artifact routes

- Canonical terms -> the relevant `CONTEXT.md`.
- All-three-gate durable decisions -> `docs/adr/NNNN-slug.md`.
- Ordinary settled decisions, assumptions, exceptions, final recommendation, and flip conditions -> `docs/deliberations/NNNN-slug.md`. Scan for the highest existing number, increment, and create the directory lazily. Update the record as decisions settle, not only at the end. When a settled decision also has an ADR, link it from the record instead of restating the rationale.

## Record shape

    # {NNNN slug}
    ## Objective / constraints
    ## Settled decisions
    ## Assumptions
    ## Exceptions
    ## Recommendation
    ## Flip conditions

## Complete

Read back every expected file. Report each layer as written or deliberately skipped with the reason. Do not write ordinary decisions into `CONTEXT.md` or inflate them into ADRs.
