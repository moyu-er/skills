---
name: codify
description: Use when project terminology needs a canonical meaning because it is ambiguous, conflicting, overloaded, or inconsistent with code, or when an already-settled architectural choice qualifies for a durable ADR because it is hard to reverse, surprising, and chosen among real alternatives.
---

# codify

Owns two artifacts: the project's glossary (`CONTEXT.md`) and qualifying ADRs. Reversible decisions, specs, scratch notes, and implementation details are out of scope.

## Locate the context

If a root `CONTEXT-MAP.md` exists, read it and pick the context the topic belongs to; if unclear, ask. Otherwise the root `CONTEXT.md` is the single context. Create the relevant `CONTEXT.md` lazily when its first term is ready; ADR files are independent.

## Sharpen terms

When a term conflicts with the glossary or arrives fuzzy and overloaded, surface it immediately. Propose one canonical word and list the rest under _Avoid_. Keep each definition to what the concept IS, domain-specific, implementation-free. General programming notions (timeouts, error types) stay out even if heavily used.

Before writing or editing a term, read [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

## Stress-test relationships

When term relationships come up, invent concrete edge cases that force precise boundaries.

## Cross-check against code

When the user states how something works, verify the code agrees. On contradiction, surface the gap and ask which is right, then write the settled meaning down at once, not batched.

## Gate ADRs

Offer an ADR only when all three hold: hard to reverse, surprising without rationale, and a genuine tradeoff with real alternatives. A reversible Markdown or local-storage choice fails the first test; it is not an ADR merely for being a decision. If the gate fails, skip explicitly and name the failed condition.

Before writing an ADR, read [ADR-FORMAT.md](./ADR-FORMAT.md).

## Done

Every resolved term is written and cross-checked against code; every ADR candidate is written or explicitly skipped with the failed gate named.
