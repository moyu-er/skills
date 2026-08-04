---
name: converge-divergence
description: Scan a codebase for divergent paths — the same concern handled differently in multiple places — and propose convergence to a single shared path. User-invoked only.
disable-model-invocation: true
---

# Converge Divergence

Scan a codebase for **divergences** — places where the same concern is
handled by multiple code paths that should be one — classify each, design
a convergence, and propose the change. Divergence is the root structural
debt of a fast-iterating codebase: each quick fix adds a branch, branches
drift, and the next change diverges further.

A divergence is _not_ mere duplication. Duplication repeats one meaning
in two places; divergence is two _mechanisms_ achieving the same goal
through different routes — different call sites, different wiring,
different lifecycles — where one route would suffice. The test: if you
deleted one path and routed its callers through the other, would
behaviour be identical? If yes, it's divergence. If the paths serve
genuinely different execution models (one is justified), it's not.

## When to use

Run this skill when you suspect structural debt has built up: before a
refactor sprint, after a major feature lands, or when a bug fix reveals
that two paths were doing the same thing differently and one was silently
broken.

## Process

### 1. Scope

Decide _where_ to scan before scanning. Divergence hides where change is
frequent, so weight the search toward hot spots:

- If the user named a direction — a module, a subsystem, a recent bug —
  take it.
- Otherwise, walk `git log --oneline -50` to find files that keep coming
  up. Those are the hot spots; let them pull attention first.

Read the project's `AGENTS.md`, `rules/`, and any `CONTEXT.md` first.
They define the intended _single path_ for each concern — a divergence
is a deviation from that intent, and you can't classify what you can't
see the baseline of.

If a previous run left a divergence index (see step 6), read it first —
it records what was already found, classified, and documented as
justified, so this run only scans for _new_ divergence.

**Completion criterion:** a named scope (directory paths or module
names) and a read of the project's convergence/architecture rules. If
the project has no documented rules, note that and proceed with generic
divergence patterns (see `PATTERNS.md`).

### 2. Scan for divergences

Dispatch parallel `explore` subagents — one per hot spot — using the
brief template at `templates/divergence-hunter.md`. Fill the
`{hot_spot}`, `{patterns_path}`, and `{rules_summary}` placeholders
before dispatching; `{patterns_path}` is the absolute path to this
skill's `PATTERNS.md` in its installed location.

The subagents are _divergence hunters_: they read code, trace call
chains, and flag every place where two paths serve one concern. They
classify each as **justified** or **accidental** — the classification is
the output, not a fix.

While subagents run, do your own scan of the cross-cutting concerns most
prone to divergence (see `PATTERNS.md` for the full catalogue with
justified/accidental criteria):

- **Construction paths** — same object type built at different sites
  with different collaborators threaded.
- **Wiring mechanisms** — same collaborator injected through different
  mechanisms (wrapper vs setter vs constructor arg).
- **Persistence paths** — same data written to two stores when one
  would suffice.
- **Filtering / dispatch logic** — same filter implemented inline in
  multiple handlers.
- **Fallback guards** — `if X is None: fall back` for code with no
  remaining old-path callers.
- **Import path divergence** — same module imported through different
  routes (direct vs lazy vs TYPE_CHECKING).

**Completion criterion:** a list of divergences, each with concern,
paths (file:line + mechanism), and justified/accidental classification.
Every hot spot scoped in step 1 is accounted for — an empty result for a
hot spot is reported as "no divergence found", not silently skipped.

### 3. Classify and prioritise

For each _accidental_ divergence, assess:

- **Blast radius** — how many callers would the convergence touch?
- **Drift risk** — how likely is the next change to diverge further if
  left alone? High if the paths are in active areas; low if stable.
- **Convergence cost** — is the target already one of the existing paths
  (converge _to_ it), or does a new seam need extracting (converge
  _through_ it)?

Prioritise: high drift risk + low convergence cost first. High blast
radius + high cost last (flag for a dedicated refactor).

**Justified** divergences are _not_ discarded — they're documented. A
justified divergence that looks accidental to a future reader is a trap;
record _why_ it's justified so the next scan doesn't re-classify it.

**Completion criterion:** a prioritised list with blast radius, drift
risk, convergence cost, and the recommended action for each (converge
now / flag for refactor / document as justified). Every accidental
divergence has a recommended action — none is left unclassified.

### 4. Design convergence

For each "converge now" item, fill the convergence design template at
`templates/convergence-design.md`. The template enforces four fields:

1. **Target** — which existing path to converge _to_, or the new seam to
   extract. One mechanism serving all callers — no variant-specific
   branching (`if <kind> == X`) embedded inside.
2. **Every caller** that must be rerouted — exhaustive. A convergence
   that leaves one caller on the old path is incomplete and will
   re-diverge.
3. **Per-caller change** — what it does now → what it will do after.
4. **Dead code removals** — files and symbols to delete. No backward-
   compatibility shims for code you just wrote.

**Completion criterion:** for each "converge now" item, a filled
convergence design with all four fields. A caller not listed is a
failure of this step, not an acceptable omission.

### 5. Present

Present the results to the user:

- **Summary** — N divergences found, M accidental, K justified, J
  prioritised for convergence now.
- **Per "converge now" item** — the filled convergence design template.
- **Per "flag for refactor" item** — concern, paths, why it's too large
  for inline convergence.
- **Justified divergences** — listed with their reason.

Do not implement. This skill proposes; the user decides what to act on.
Implementation the user approves is handed to `subagent-implement`.

**Completion criterion:** the report is presented; any classification
the user disputes is resolved before the index records it.

### 6. Write divergence index

Write a markdown file to `.divergence-index.md` in the project root (or
a `.divergence/` directory if the project prefers isolation). This makes
the divergence state persistent, so the next run is incremental, not
full-scan.

The index has three sections:

- **Converged** — items this run (or a previous run) proposed convergence
  for, with the target path and date. If the convergence was
  implemented, note the commit. If not, note it as _pending_.
- **Justified** — divergences classified as justified, with the
  execution-model reason. These are the "known and accepted" divergences
  — the next scan should NOT re-classify them.
- **Pending** — "flag for refactor" items not yet addressed.

**Completion criterion:** the index file is written with all divergences
from this run categorized into the three sections. The file is
self-contained — a future reader can understand the current divergence
state of the codebase from it alone.
