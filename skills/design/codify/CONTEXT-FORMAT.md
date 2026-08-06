# CONTEXT.md Format

`CONTEXT.md` is a glossary and nothing else. No specs, decisions, scratch notes, or implementation details.

## Structure

```md
# {Context Name}

{One or two sentences: what this context is and why it exists.}

## Language

**Order**:
{One or two sentences defining what the concept IS.}
_Avoid_: Purchase, transaction

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## Rules

- Pick one canonical word per concept; list the rest under _Avoid_.
- One or two sentences max. State what the concept IS, not what it does.
- Domain-specific only. General programming concepts (timeouts, error types) stay out even if heavily used.
- Subheadings only when natural clusters emerge; otherwise a flat list.

## Single vs multi-context

- No `CONTEXT.md` yet: create one at the repo root when the first term settles.
- Root `CONTEXT.md` only: single context.
- Root `CONTEXT-MAP.md` present: the repo has multiple contexts. The map lists each context's `CONTEXT.md` path and how they relate. Infer which context the current topic belongs to; if unclear, ask.

Create the directory and file lazily, only when something is ready to write.
