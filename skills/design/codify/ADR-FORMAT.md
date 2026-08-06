# ADR Format

ADRs record durable architectural decisions only. A choice that is easy to reverse, unsurprising, or had no real alternative is not an ADR, even if it was a decision.

## Location and numbering

ADRs live in `docs/adr/` as `0001-slug.md`, `0002-slug.md`, and so on. Scan the directory for the highest existing number and increment by one. Create `docs/adr/` lazily, only when the first ADR qualifies.

## Template

```md
# {Short title}

{1 to 3 sentences: the context, the decision, and why.}
```

That is the whole required shape. An ADR can be a single paragraph. The value is recording *that* the call was made and *why*, not filling out sections.

## Optional sections

Add only when they earn their place; most ADRs need none.

- **Status** frontmatter: `proposed | accepted | deprecated | superseded by ADR-NNNN`.
- **Considered Options**: only when rejected alternatives are worth keeping.
- **Consequences**: only for non-obvious downstream effects.

## Qualifies when all three hold

1. Hard to reverse, changing your mind later costs meaningfully.
2. Surprising without rationale, a future reader will ask why.
3. A real tradeoff, genuine alternatives existed and one was picked for specific reasons.

Reversible Markdown or local-storage choices fail the first test. Skip them.
