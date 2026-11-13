# Review standard

What this reviewer looks for and what they block on. Drafts follow it, and
retros refine it. Edit freely.

## What to look for

- **Premise.** Read the linked issue first: it states the problem, and its
  discussion often settles the approach. Is the problem real, and does the
  change solve *that* problem, not something broader or narrower? If the
  shape looks wrong, raise it as a `question` or `suggestion`. Whether to
  accept this shape of change is the reviewer's call, so don't hard-block on
  premise.
- **Correctness and risk.** Look hardest at concurrency and `unsafe` code,
  I/O and serialization (format reads and writes, checksums), error and
  panic paths, and API or wire-compatibility breaks.
- **Tests.** A behavior change with **no tests blocks**. So does a test that
  covers only part of an important property (for example one format version
  when two matter). Name the gap and propose the concrete test. Green CI isn't
  coverage: it only proves the tests that exist pass, not that the right
  tests exist.
- **Architecture.** Is the change at the right layer, and does it reuse what
  exists rather than duplicating it? A leaky abstraction that pushes work onto
  callers, or logic in the wrong layer, is a real finding.
- **API design.** For any public surface, check:
  - it's consistent with its siblings (naming, signatures)
  - bindings in other languages stay in step
  - it can evolve without breaking older clients
  - it makes sense across the whole user workflow, not just one call
- **Scope.** Flag changes unrelated to the PR's stated purpose: drive-by edits,
  incidental reformatting, refactors folded into a feature.
- **Simplicity.** Look for dead branches, redundant conditionals, hand-rolled
  logic that a standard library call replaces, and needless copies. When the
  fix is a small literal drop-in, offer it as a suggestion with the replacement
  code.

## What blocks

**Block on what can't be walked back.** One-way doors block:

- correctness bugs that would ship
- missing tests
- data, format, or wire compatibility
- the *shape* of a public API: a signature or type that can't change later
  without breaking callers

Two-way doors are non-blocking follow-ups, because a later PR can fix them
compatibly:

- performance
- internal code quality
- purely additive API enhancements

The line is reversibility, not public versus private.

## How to raise things

- **Ask before dictating.** When a design choice might have a reason you can't
  see from the diff, ask "why X over Y?" It lets the author explain or
  correct course, and costs nothing if they were right.
- **No vague worries.** If you need convincing of something ("this lock order
  can't deadlock"), either verify it yourself or turn it into a concrete ask
  the author can satisfy.
- **Withdraw when the author was right.** If you can reconstruct why they did
  it their way, drop the comment. If it confused you, it may be worth asking
  for a clarifying comment in the code instead.
- **Blocking issues state their acceptance criterion**: how the author knows
  the issue is resolved.
- **Lean toward surfacing, but rank.** A plausible concern you can't fully
  verify is worth raising as a question. A false positive costs one dismissal,
  while a missed real issue costs much more. Ranking keeps that from turning
  into noise.
