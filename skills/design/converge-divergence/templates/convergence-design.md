# Convergence Design — Item Template

> Fill one per "converge now" item from step 4. This is the structured
> output the skill presents to the user for each convergence proposal.

## Concern

{one phrase naming what the paths collectively do}

## Current State

### Path A (target — the path to converge toward)
- Location: `{file}:{line}`
- Mechanism: {how it achieves the concern}
- Why it's the target: {why this path is the right one — most complete,
  already used by the majority of callers, no variant-specific branching
  inside, etc.}

### Path B (to be rerouted)
- Location: `{file}:{line}`
- Mechanism: {how it achieves the concern differently}
- Why it diverged: {best guess — written before Path A existed, bypassed
  a wrapper, different author, etc.}

{Repeat Path B for each additional divergent path.}

## Convergence Plan

### Callers to reroute

Every caller currently on Path B (or any non-target path). A caller not
listed here is an incomplete convergence — it will re-diverge.

| # | Caller | File:Line | Current (Path B) | After (Path A) |
|---|--------|-----------|-------------------|-----------------|
| 1 | {function/class name} | `{file}:{line}` | {what it does now} | {what it will do after} |
| 2 | ... | ... | ... | ... |

### Dead code to remove

Code that becomes unreachable after all callers are rerouted. Do not
leave backward-compatibility shims.

| File | Symbol | Reason |
|------|--------|--------|
| `{file}` | `{function/class/field}` | {why it's dead after convergence} |

### New seam (if needed)

If no existing path is the right target, a new shared seam must be
extracted. Describe it:

- **Type**: {helper function / ABC method / factory function / shared
  adapter}
- **Location**: {where it should live — which module}
- **Signature**: {function/method signature}
- **Variant-neutral**: {confirm no `if <kind> == X` branching inside}

## Assessment

- **Blast radius**: {N callers affected}
- **Drift risk**: {high/medium/low — how likely to re-diverge if left}
- **Convergence cost**: {low/medium/high — complexity of the rerouting}
- **Files touched**: {N}
- **Tests to update**: {which test files need changes}
