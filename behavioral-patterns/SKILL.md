---
name: behavioral-patterns
description: Review dispatch/event/state-transition code for a missing or overused Strategy, Observer, Command, Template Method, or State (Design Patterns, GoF 1994) — flag reflexive abstraction, not correctness. Use when reviewing type-switches, listener lists, or status-field logic.
---

# Behavioral patterns (Design Patterns)

These patterns solve specific object-collaboration problems — they are not
a better default than a direct call or a plain conditional. This skill is a
review lens: it exists to catch a pattern that's missing (an ad-hoc,
struggling version of it already exists in the code) or present without
earning its keep.

## How to review for these patterns

For each pattern below, walk through this same check — don't skip straight
to "does this match the GoF diagram," which invites pattern-name-dropping
without an underlying reason:

1. **Is the problem this pattern solves actually present?** Look for the
   real symptom (below, per pattern), not just a structural resemblance.
   No symptom, no gap to fill.
2. **Is there already an ad-hoc, unnamed version of this pattern struggling
   to do its job?** A hand-rolled dispatch table, a manually maintained
   listener list, or a duplicated status `switch` is often "the pattern,
   done informally and now creaking" — the strongest signal to extract it
   properly.
3. **Would naming/extracting the pattern reduce real duplication or
   coupling, or just relabel the same code with more indirection?** If
   nothing shrinks or decouples, don't recommend it — an unnecessary layer
   of indirection is its own complexity (see [[minimal-complexity]]).
4. **If the pattern is already present, is it earning its keep?** Patterns
   get reached for out of habit long after the need they addressed has
   stopped mattering.

## Strategy

**The real problem it solves:** making an algorithm swappable at runtime
behind a common interface, when more than one interchangeable way of doing
the same job genuinely exists.

**Missing-pattern signal:** a type or mode-flag `switch` selecting between
interchangeable algorithms, duplicated at more than one call site — every
caller re-implementing the same dispatch.

**Misapplied/overused signal:** a strategy interface with exactly one
production implementation. There's no variation to swap, so the interface
is pure ceremony around a single function.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires at least two real variants existing (or concretely
imminent) with dispatch logic already duplicated; removal requires
confirming only one implementation exists with no second one on the
horizon.

## Observer

**The real problem it solves:** one-to-many notification, so a subject's
state change reaches its dependents without the subject needing to know
their concrete types.

**Missing-pattern signal:** a subject explicitly calling a hardcoded list
of specific objects' update methods after every state change — the
notification logic is duplicated and the subject is coupled to types it
shouldn't need to know about.

**Misapplied/overused signal:** "observer soup" — a cascade of listeners
triggering listeners triggering listeners, impossible to trace from one
state change to its effects; or an observer used for a single,
always-synchronous 1:1 relationship where a direct call would be clearer
and easier to follow.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires genuinely multiple independent dependents that
shouldn't know about each other; removal (or simplifying to a direct call)
requires confirming there's really just one dependent, or that the
notification chain has become unreadable.

## Command

**The real problem it solves:** encapsulating a request — receiver, action,
and parameters — as an object, so it can be queued, logged, retried, or
undone independently of when it's invoked.

**Missing-pattern signal:** undo/redo, retry, or queueing logic that
re-derives "what action to perform" from scattered caller state each time,
instead of capturing the action once as a reusable object.

**Misapplied/overused signal:** a command object that's created and
executed immediately in the same place, never queued, logged, retried, or
undone — it's standing in for a plain function call with extra
indirection.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires a concrete need to defer, queue, log, or undo the
action; removal requires confirming the command is always invoked
immediately with no such need.

## Template Method

**The real problem it solves:** defining an algorithm's skeleton once in a
base method, deferring only the steps that genuinely vary to subclasses.

**Missing-pattern signal:** several sibling implementations repeat the same
multi-step sequence, differing only in one or two steps — the shared
structure is duplicated instead of factored out once.

**Misapplied/overused signal:** a hierarchy with only one subclass (nothing
to share the skeleton with yet), or one where the overridden steps have
grown to be nearly the whole method — at that point the "shared skeleton"
isn't shared with anything real, and a composition-based alternative
(passing the varying steps in directly, closer to Strategy) is usually
clearer than forcing it through inheritance.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires the same step sequence duplicated across two or
more real implementations; removal or restructuring requires confirming
subclasses have diverged enough that little of the base skeleton is
actually shared anymore.

## State

**The real problem it solves:** letting an object change its behavior as
its internal state changes, by giving each state its own encapsulated
behavior instead of checking the state field everywhere that behavior
depends on it.

**Missing-pattern signal:** the same `switch(status)` or
`if (state == 'x')` block, checking the same field, duplicated across
multiple methods of a class — every method re-implementing the same set of
state-dependent branches.

**Misapplied/overused signal:** a full State class hierarchy built around a
status field with only two values and one transition — a boolean or enum
check inline is clearer and the hierarchy is ceremony without a real
transition graph to justify it.

**Don't flag this pattern without a concrete symptom already in the code**
— introduction requires the state-conditional logic already duplicated
across multiple methods/call sites for the same field; removal requires
confirming the field only has one or two states with no meaningful
per-state behavior to encapsulate.

## Applying this when writing new code

This skill is a review lens first. If you're about to reach for Strategy,
Observer, Command, Template Method, or State while writing new code, hold
it to the same bar as step 1 above: introduce it only because a concrete
symptom already exists in the code you're writing — not because the design
"should" have this pattern, the domain sounds like a textbook example, or
it might be needed later. When in doubt, write the plain conditional or
direct call first; add the pattern when the plain version actually starts
hurting.

## Applying this when reviewing existing code

1. For each pattern in this file, check the missing-pattern signal against
   the code under review.
2. For each pattern already present, check the misapplied/overused signal —
   confirm it's still solving a real problem, not fossilized ceremony.
3. Report findings pattern-by-pattern: name the pattern, cite the concrete
   symptom, and state what would change if it were introduced or removed.
4. Don't recommend introducing a pattern you can't tie to a concrete signal
   already present in the code — see the methodology above.
