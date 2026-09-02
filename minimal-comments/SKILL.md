---
name: minimal-comments
description: Write or review code comments using Steve McConnell's Code Complete 2 guidance (ch. 32) — comment intent and "why", not mechanics or "what"; prefer self-documenting code over explanation; cut comments that restate the code, decorate it, or apologize for it. Use when writing new code, when the user asks to add/clean up/reduce comments, review comment quality, or asks whether something needs a comment.
---

# Minimal comments (Code Complete 2)

Comments are a tool for what code *cannot* say, not a substitute for code that
doesn't say enough. Default to none; add one only when it survives the test below.

## The test for any comment

Before writing or keeping a comment, ask: **could this be fixed in the code
instead?** A comment explaining a confusing name, a magic number, or a tangled
conditional is a patch over bad code — rename the variable, extract the
constant, extract a well-named function instead. Comments are not a deodorant
for bad code (Code Complete 2, ch. 32).

If the answer is no — the code genuinely cannot express this — the comment
earns its place. That's true for:

- **Why, not what.** The reasoning, constraint, or trade-off behind a
  non-obvious choice ("retry here because the upstream API drops connections
  under load, not because of anything in our code").
- **Intent above a block.** One line stating the goal of a chunk of logic,
  written *before* the code, at a higher level of abstraction than the code
  itself — not a restatement of the next three lines.
- **Warnings of consequence.** A non-obvious side effect, a workaround for a
  specific bug (ideally with a ticket/issue reference), or an invariant the
  reader could violate without knowing it.
- **Legal, licensing, or attribution notices** required by the project.
- **Amplification.** A boundary case or a value that looks arbitrary but
  isn't ("0 is a valid input here, meaning 'no limit', not 'error'").

## Comments to cut on sight

Code Complete 2 catalogs these as the recurring failure modes — treat any
comment matching one as a candidate for deletion, not preservation:

- **Restating the code.** `i++; // increment i`. If a comment and the code
  say the same thing, the comment is pure noise and will rot the moment one
  of them changes without the other.
- **Mumbling.** A comment vague enough that it could apply to any line near
  it — it signals the author didn't understand the code well enough to
  explain it, or didn't bother.
- **Decoration / structure markers.** `//////// SETUP ////////` banners,
  end-of-block `// end if` markers, box-drawn section dividers. Layout and
  naming should carry this weight instead.
- **Apology or confession.** `// I know this is hacky but...`. Fix the code;
  don't leave a note explaining that you didn't.
- **Changelog-in-comments.** `// added by Josh 2024-03-01`,
  `// removed old validation here`. That's what `git log`/`git blame` are
  for; a comment referencing the current fix, the current caller, or the
  current ticket rots the moment the history moves on.
- **Redundant parameter/return docs.** Documenting an obviously-named
  parameter (`@param name the name`) or a self-evident return value adds
  ceremony, not information.
- **Commented-out code.** Delete it; version control remembers.

## Style, when a comment does earn its place

- Prefer one comment that explains the intent of a whole block over a
  comment on every line inside it.
- Write it at the same level of abstraction as the surrounding code's
  purpose, not a play-by-play of statements.
- Keep it physically next to the code it describes, so the two are far more
  likely to be updated together — a comment far from its code, or one
  covering code that has since changed, is worse than no comment: it's
  actively misleading.
- No filler: skip "This function is used to...", state the fact directly.

## Applying this when writing new code

Write self-documenting code first — descriptive names, small functions with
one clear job, named constants instead of magic numbers, guard clauses instead
of nested conditionals. Reach for a comment only after that effort still
leaves a real gap (the "why" behind a decision, a non-obvious constraint, a
warning). This matches the default: no comments unless the reasoning would
otherwise be invisible to a future reader.

## Applying this when reviewing/cleaning up existing comments

For each existing comment, classify it against the two lists above:

1. If it matches a "cut on sight" pattern, remove it (and fix the underlying
   code if the comment was compensating for a bad name or tangled logic).
2. If it explains "why", warns of a consequence, or states non-obvious
   intent, keep it — tighten the wording if it's verbose.
3. If it's ambiguous, prefer deleting and improving the code's
   self-documentation (rename, extract) over keeping a comment that might be
   restating or might be adding value — when in doubt, make the code clearer
   instead of explaining the unclear version.

Report or apply changes as deletions and rewrites, not additions — this skill
should shrink comment volume, not pad it.
