---
name: creational-patterns
description: Review object-construction code for a missing or overused Singleton, Factory Method, or Builder (Design Patterns, GoF 1994) — flag reflexive abstraction, not correctness. Use when reviewing constructors, `new`-heavy code, or object-creation call sites.
---

# Creational patterns (Design Patterns)

These patterns solve specific object-construction problems — they are not a
better default than a plain constructor. This skill is a review lens: it
exists to catch a pattern that's missing (an ad-hoc, struggling version of
it already exists in the code) or present without earning its keep.

## How to review for these patterns

For each pattern below, walk through this same check — don't skip straight
to "does this match the GoF diagram," which invites pattern-name-dropping
without an underlying reason:

1. **Is the problem this pattern solves actually present?** Look for the
   real symptom (below, per pattern), not just a structural resemblance.
   No symptom, no gap to fill.
2. **Is there already an ad-hoc, unnamed version of this pattern struggling
   to do its job?** Duplicated construction logic or an informally
   maintained "only one of these" convention is often "the pattern, done
   informally and now creaking" — the strongest signal to extract it properly.
3. **Would naming/extracting the pattern reduce real duplication or
   coupling, or just relabel the same code with more indirection?** If
   nothing shrinks or decouples, don't recommend it — an unnecessary layer
   of indirection is its own complexity (see [[minimal-complexity]]).
4. **If the pattern is already present, is it earning its keep?** Patterns
   get reached for out of habit long after the need they addressed has
   stopped mattering.

## Singleton

**The real problem it solves:** ensuring a class has exactly one instance
that's globally accessible, when having more than one would be actively
wrong — a single hardware connection, a single in-memory cache, a single
configuration snapshot for the life of a process.

**Missing-pattern signal:** multiple independently-constructed instances of
something that must be process-global are drifting out of sync — e.g. two
`new ConfigLoader()` calls in different modules each caching their own copy
of config, and a config reload only updates one of them; or a connection
pool getting re-initialized per request instead of once per process.

**Misapplied/overused signal:** a `getInstance()`/static-instance wrapper
around something that has no actual uniqueness constraint — it's just a
convenient way to avoid passing a dependency through a few layers of
constructors. Tells: the singleton holds mutable state that tests have to
reset between runs; a second instance would work fine if one existed
(nothing enforces or depends on "exactly one"); or the singleton-ness exists
purely to dodge dependency injection, at the cost of hidden global coupling
and hard-to-isolate tests.

**Don't flag this pattern without a concrete symptom already in the code**
— flag *removal* only when you can point to an actual test-isolation
failure, a hidden coupling bug, or a case where a second instance is now
legitimately needed and the singleton is blocking it; don't flag
*introduction* of a singleton merely because "there's only one instance
right now" — that's true of most objects at some point and isn't itself a
reason to add the machinery.

## Factory Method

**The real problem it solves:** deferring the decision of which concrete
class to instantiate to a swappable creation step, when the caller only
needs to know it's getting "a valid instance of the family," not the exact
subtype.

**Missing-pattern signal:** the same type-discriminator `if`/`switch` that
picks which of several related subclasses to `new` up is duplicated across
more than one call site — every caller re-implementing the same
type-to-class mapping.

**Misapplied/overused signal:** a "factory" function or class that only
ever constructs one concrete type. That's a constructor with extra
ceremony, not a Factory Method — there's no variation for it to defer.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires a second real variant already existing and the
construction/dispatch logic already duplicated across call sites; removal
requires confirming the "factory" genuinely has just one product and no
near-term second one.

## Builder

**The real problem it solves:** constructing a complex object step-by-step
when the constructor would otherwise need many optional or interacting
parameters — the classic telescoping-constructor problem, or a multi-step
assembly sequence that's easy to get wrong or forget a step in.

**Missing-pattern signal:** a constructor or factory function with a long
optional-parameter list (especially several booleans), multiple overloads
standing in for different combinations, or callers passing placeholder
`null`/`undefined` just to reach a later parameter.

**Misapplied/overused signal:** a fluent `.setX().setY().build()` wrapper
around an object with two or three simple, independent fields — an object
literal or a couple of named parameters already expresses that cleanly;
the builder adds ceremony without solving an actual construction problem.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires the parameter list already being hard to call
correctly (order-sensitive, many valid combinations, easy to omit a
required step); removal requires confirming the wrapped object is simple
enough that direct construction wouldn't be error-prone.

## Applying this when writing new code

This skill is a review lens first. If you're about to reach for Singleton,
Factory Method, or Builder while writing new code, hold it to the same bar
as step 1 above: introduce it only because a concrete symptom already
exists in the code you're writing — not because the design "should" have
this pattern, the domain sounds like a textbook example, or it might be
needed later. When in doubt, write the plain constructor first; add the
pattern when the plain version actually starts hurting.

## Applying this when reviewing existing code

1. For each pattern in this file, check the missing-pattern signal against
   the code under review.
2. For each pattern already present, check the misapplied/overused signal —
   confirm it's still solving a real problem, not fossilized ceremony.
3. Report findings pattern-by-pattern: name the pattern, cite the concrete
   symptom, and state what would change if it were introduced or removed.
4. Don't recommend introducing a pattern you can't tie to a concrete signal
   already present in the code — see the methodology above.
