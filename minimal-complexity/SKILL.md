---
name: minimal-complexity
description: Write or review code for cyclomatic complexity using Steve McConnell's Code Complete 2 guidance (ch. 19, 24) — count decision points, flag routines above the risk threshold, and reduce complexity by restructuring (extract routines, guard clauses, table-driven logic, polymorphism) rather than by adding explanation. Use when writing new logic with several branches, when the user asks to simplify/refactor a function, review complexity, split up a long routine, or asks whether something is too complex/nested.
---

# Minimal complexity (Code Complete 2)

Complexity in a routine should be reduced by restructuring it, not managed by
comments or careful naming around it. A tangled routine explained well is
still a tangled routine.

## Measuring complexity

Cyclomatic complexity (McCabe's metric, as presented in Code Complete 2 ch. 19):
count the routine's decision points, then add 1.

A decision point is any `if`, `else if`, `while`, `for`/`until`, `case`
branch, `catch` clause, and each `and`/`or` inside a compound conditional
(each one is an independent branch, not just the containing `if`).

This number is also the minimum number of test cases needed to exercise
every independent path through the routine — high complexity isn't just
"hard to read," it's directly "expensive to test correctly."

## Risk thresholds (a rule of thumb, not a hard law)

- **1–10** — straightforward, low risk. No action needed on complexity grounds alone.
- **6–10** — getting busy; worth noticing while touching the routine, not necessarily worth a standalone refactor.
- **>10** — treat as the practical ceiling. This is the signal to decompose, not a hard failure — check it against the guidance below before splitting.
- **>20** — hard to test and hard to reason about with confidence; strong candidate for restructuring even outside a review.
- **>50** — effectively untestable; restructure before adding to it further.

These numbers correlate empirically with defect rates but the relationship
isn't perfectly linear. Use complexity as one signal among several (routine
length, nesting depth, number of local variables) — not a metric to chase
for its own sake, and not proof a routine under the threshold is safe.

## How to actually reduce it (McConnell's techniques)

In order of preference — restructure first, only add process (tests, review)
around what's left:

1. **Extract routines.** Pull a branch, a loop body, or a cohesive chunk of
   logic into its own well-named routine. This is the single highest-leverage
   fix: it removes decision points from the caller entirely rather than
   hiding them.
2. **Use guard clauses instead of nested conditionals.** Return/continue
   early on the exceptional or boundary case so the main logic isn't nested
   three levels deep inside `if`s. Deep nesting multiplies the number of
   paths a reader has to hold in their head at once, independent of the raw
   decision count.
3. **Replace large `case`/`switch` or `if`/`else if` chains** that dispatch
   on type or state with a lookup table, a strategy/polymorphism pattern, or
   a map of handlers, when the branches represent variants of the same
   operation.
4. **Simplify boolean expressions.** Break a compound condition into
   well-named intermediate boolean variables (`isEligible = a && !b || c`)
   instead of one dense expression — this doesn't lower the cyclomatic count,
   but it's often the step that makes the next simplification (deduplicating
   branches, spotting a redundant condition) visible.
5. **Consolidate duplicate branches.** If two branches of an `if`/`case` do
   almost the same thing, that's usually a sign the condition was modeling
   the wrong distinction — merge them and move the real difference into a
   parameter or a smaller inner branch.
6. **Split by responsibility, not by size.** When a routine exceeds the
   threshold because it's doing several unrelated jobs (validate, then
   compute, then format), split along those seams. Don't split an
   arbitrarily-long-but-single-purpose routine just to hit a line count.

## What not to do

- Don't leave complexity in place and write a comment explaining the control
  flow instead — that's documenting a problem, not fixing it (see
  [[minimal-comments]]).
- Don't reduce the visible complexity number by inlining a call that just
  hides the same branches one level down; the reduction has to remove real
  decision points, not relocate them.
- Don't split a routine into pieces that only make sense read together in
  order — that trades cyclomatic complexity for a different kind of
  complexity (having to jump between routines to follow one path). Each
  extracted routine should be independently understandable.

## Applying this when writing new code

Before adding another branch to a routine that already has several, count
the decision points. If it's climbing past ~10, look for the guard-clause,
extraction, or table-driven alternative before adding more — most complexity
growth happens one "just one more `if`" at a time.

## Applying this when reviewing existing code

1. Count decision points per routine; flag anything over ~10.
2. For each flagged routine, identify which technique above actually applies
   — don't reach for extraction by default if the real fix is guard clauses
   or a lookup table.
3. Apply the restructuring, then recount to confirm the complexity actually
   dropped (not just moved into a sibling routine with the same total).
4. Report changes as structural refactors (extracted routines, flattened
   nesting, replaced branching), not as comments or documentation added
   around the existing structure.
