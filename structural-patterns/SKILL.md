---
name: structural-patterns
description: Review wrapper/subsystem-facing code for a missing or overused Adapter, Decorator, or Facade (Design Patterns, GoF 1994) — flag reflexive indirection, not correctness. Use when reviewing wrappers, integration code, or a subsystem's public entry points.
---

# Structural patterns (Design Patterns)

These patterns solve specific object-composition problems — they are not a
better default than calling something directly. This skill is a review
lens: it exists to catch a pattern that's missing (an ad-hoc, struggling
version of it already exists in the code) or present without earning its
keep.

## How to review for these patterns

For each pattern below, walk through this same check — don't skip straight
to "does this match the GoF diagram," which invites pattern-name-dropping
without an underlying reason:

1. **Is the problem this pattern solves actually present?** Look for the
   real symptom (below, per pattern), not just a structural resemblance.
   No symptom, no gap to fill.
2. **Is there already an ad-hoc, unnamed version of this pattern struggling
   to do its job?** Copy-pasted translation code or a repeatedly
   hand-orchestrated call sequence is often "the pattern, done informally
   and now creaking" — the strongest signal to extract it properly.
3. **Would naming/extracting the pattern reduce real duplication or
   coupling, or just relabel the same code with more indirection?** If
   nothing shrinks or decouples, don't recommend it — an unnecessary layer
   of indirection is its own complexity (see [[minimal-complexity]]).
4. **If the pattern is already present, is it earning its keep?** Patterns
   get reached for out of habit long after the need they addressed has
   stopped mattering.

## Adapter

**The real problem it solves:** converting an existing interface into the
one callers actually need, without modifying either side — typically
because the source is third-party, legacy, or otherwise out of your
control.

**Missing-pattern signal:** the same field-renaming, type-coercion, or
shape-translation logic for a third-party/legacy API is copy-pasted at
every call site that touches it, instead of living in one place.

**Misapplied/overused signal:** an "adapter" that forwards every call 1:1
with identical method names and types — nothing is actually being bridged.
It's usually added speculatively ("in case we swap providers later"), which
is indirection with no current payoff.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires a real, currently-existing interface mismatch
already causing duplicated translation code across two or more call sites;
removal requires confirming there's no actual shape difference being
bridged.

## Decorator

**The real problem it solves:** attaching behavior to an individual object
dynamically by wrapping it, so optional cross-cutting concerns (logging,
caching, retry, auth checks) can be combined independently instead of
requiring a subclass for every combination.

**Missing-pattern signal:** a combinatorial explosion of subclasses, or a
constructor with a boolean flag per optional feature, standing in for
add-ons that should be independently composable.

**Misapplied/overused signal:** a decorator chain deep enough that the
object's actual active behavior can't be read at the call site without
tracing every wrapper, or a "decorator" that changes the wrapped object's
contract instead of adding to it — that's not decoration, it's a different
type wearing the same interface.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires a concrete cross-cutting concern that genuinely
needs to vary independently per instance; removal requires confirming the
chain has grown unreadable or the "decoration" no longer behaves like an
addition.

## Facade

**The real problem it solves:** giving external callers one simplified
entry point over a subsystem's internal collaborators, so they don't need
to know the subsystem's internal call order or object graph.

**Missing-pattern signal:** callers outside the subsystem repeatedly
orchestrate the same multi-step call sequence, in the same order, against
the subsystem's internals — duplicated orchestration logic living outside
the subsystem it belongs to.

**Misapplied/overused signal:** a facade method that forwards a single call
to a single underlying method with no simplification — pure indirection
that adds a hop without hiding any real complexity.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires the same orchestration sequence duplicated across
two or more callers, not just observed once; removal requires confirming
the facade isn't actually collapsing multiple steps into one.

## Applying this when writing new code

This skill is a review lens first. If you're about to reach for Adapter,
Decorator, or Facade while writing new code, hold it to the same bar as
step 1 above: introduce it only because a concrete symptom already exists
in the code you're writing — not because the design "should" have this
pattern, the domain sounds like a textbook example, or it might be needed
later. When in doubt, call the thing directly first; add the pattern when
the direct version actually starts hurting.

## Applying this when reviewing existing code

1. For each pattern in this file, check the missing-pattern signal against
   the code under review.
2. For each pattern already present, check the misapplied/overused signal —
   confirm it's still solving a real problem, not fossilized ceremony.
3. Report findings pattern-by-pattern: name the pattern, cite the concrete
   symptom, and state what would change if it were introduced or removed.
4. Don't recommend introducing a pattern you can't tie to a concrete signal
   already present in the code — see the methodology above.
