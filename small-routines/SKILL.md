---
name: small-routines
description: Write/review routines per Code Complete 2 (ch. 7) — one clear purpose per routine (functional cohesion), loose coupling, few well-ordered parameters, names that fully describe behavior. Use when writing/reviewing a function's responsibilities, splitting a routine, or naming/parameter design.
---

# Small routines (Code Complete 2)

A routine earns its existence by doing one thing well with a simple
interface — not by being under some line-count limit. Line count is not the
metric; cohesion and coupling are.

## Valid reasons to create a routine

Reduce complexity, avoid duplicating code, make a sequence of operations
easier to see as a unit, hide an implementation detail or risky operation
behind a stable name, isolate something likely to change, or give a group of
related statements a single point of change. If none of these apply, the
split isn't earning its keep — don't extract just to hit a size target.

## Functional cohesion — one clear purpose

A routine should perform exactly one well-defined operation. Signs a routine
has drifted past that:

- Its name needs "and" to describe what it does (`validateAndSave`).
- It has a boolean/mode parameter that switches its behavior between two
  unrelated things (`process(data, isDryRun)` doing genuinely different work
  per branch, not just skipping a side effect).
- Callers only ever need part of what it does, forcing them to work around
  the rest.

If a routine's job is genuinely complex, keep the routine — see
[[minimal-complexity]] for when its *control flow* has grown too tangled to
follow. Cohesion is about purpose, not line count: a long routine doing one
real job is fine; a short routine doing two unrelated things is not.

## Loose coupling between routines

Prefer connections between routines that are simple, small, visible, and
direct:

- **Simple**: pass what's needed, not a whole object when a routine only
  reads one field of it (unless the object itself is the natural unit).
- **Small**: fewer parameters and fewer points of contact.
- **Visible**: dependencies show up in the signature — no reaching into
  globals or shared mutable state to get information the caller could have
  passed in.
- **Direct**: avoid semantic coupling, where a caller must know something
  about a routine's internals or call order that isn't expressed in its
  interface (e.g. "you must call `init()` before `process()`, and the
  routine won't tell you if you don't").

## Interface design

- **Parameter count**: keep it small; beyond ~5–7 parameters, group related
  values into an object/struct or split the routine.
- **Order**: put parameters in a consistent order across related routines
  (e.g. input params before output params), so callers can predict signatures.
- **Use every parameter.** An unused parameter is a stale interface — remove
  it rather than leave it as debt.
- **Avoid output parameters that double as status flags** bolted onto an
  otherwise unrelated operation; return a result (or a small result type)
  instead of mutating an out-param to signal something orthogonal to the
  routine's main job.

## Naming

The name should fully and accurately describe everything the routine does —
if it doesn't, either the name is wrong or the routine is doing more than
one thing (back to cohesion). Avoid vague verbs that could mean anything
(`handle`, `process`, `manage`) in favor of the specific action taken.

## Applying this when writing new code

Before adding a new responsibility to an existing routine, ask whether it's
still one operation or now two — if two, give the new behavior its own
routine with its own name, rather than growing the parameter list or adding
a mode flag to the old one.

## Applying this when reviewing existing code

1. Check the name against what the routine actually does — a mismatch
   usually means split it.
2. Check parameter count and whether every parameter is load-bearing.
3. Check for semantic coupling — could this routine be called in the wrong
   order or state without anything in its signature warning the caller?
4. Don't split purely to shorten a routine; split along a real seam in
   responsibility, the way [[minimal-complexity]] splits along branching
   complexity.
