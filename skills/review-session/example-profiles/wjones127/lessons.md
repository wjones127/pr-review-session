# Review lessons

Repo-agnostic guidance on how Will reviews, from `review-session` retros.
Each entry cites its source. Repo-specific knowledge does not belong here.

## Do more

- For changes to a persisted format or public contract, check that the
  stability label and rollout path fit the size of the change. An experimental
  label on a small, low-risk change is a smell. Also check that the docs say
  what readers must support (e.g. which flags each option requires).
  *(2026-11-10 lance: #10914 — Will's own blocking/doc comments, none drafted)*
- When a design needs extra machinery to compensate for something (cleanup
  passes, single-entry workarounds), ask whether the root cause can be designed
  away, and propose that instead of poking at the machinery's edge cases.
  *(2026-11-10 lance: #10800.3, which then unblocked #10800.1)*
  Likewise, when code special-cases specific failure patterns, ask whether the
  general case is simpler and more robust (e.g. any parse error on fully
  library-generated input implies a version mismatch, not user error).
  Start every draft with a one-line premise check: is this the right approach,
  or can the cost be removed at the source? Before questioning a premise, look
  for prior design discussion and ask to be pointed to the latest decision. A
  premise comment supersedes detail comments that assume the design.
  *(#10932, #10868; 2026-11-12 #10923.2)*
- When a PR adds a field to a persisted or wire type, check that it
  round-trips through every serialization layer and language binding (JSON
  helpers, `asdict`-style converters, FFI/JNI constructors). A dropped field
  silently corrupts data the next time something writes the object back, so
  treat a gap as blocking. *(2026-11-12 lance: #10914.1, .2; Will: this kind
  of bug has broken systems before)*
- For format specs, write suggestions as rules for implementations
  ("must"/"must not"), not descriptions of what the library does. Base them on
  the invariant, verified against the code, not on an implementation quirk.
  *(2026-11-12 lance: #10702.1b, .1, .2)*
- A deleted or missing safety contract on a public or `unsafe` API (buffer
  capacity requirements, trust assumptions) is blocking.
  *(2026-11-12 lance: #10907.7 escalated by Will)*
- When a PR adds self-contained new types to an already large module, suggest
  a submodule that gets its own unit tests. A follow-up PR is fine if a broader
  refactor is warranted. Don't bundle unrelated moves.
  *(2026-11-12 lance: #10463.10 added by Will)*
- For paired or overlapping PRs (client/server, SDK/CLI, two PRs defining the
  same key or identity), carry Will's decisions across: drop points he rejected
  on the twin, and draft a point on both when it applies to both.
  *(2026-11-11: #4944; 2026-11-12: #8880.3 ↔ #10800.5; Will: helpful)*

- Flag two ways to do the same thing: overlapping API knobs, or parallel
  methods/code paths where one would do. Ask which one wins, or propose
  collapsing to one path. *(2026-11-11 lance: #10829.5 edited into this)*
- Naming: name things after what the value *is*, not how it's used
  (`partition_id`, not `mask`). Flag generic names (`data`, `inner`, `tmp`)
  and names that lie about their contents. *(from polish-pr, 2026-11-11)*
- Raise trade-offs and regressions as questions even when the author
  acknowledged them or they seem likely acceptable. Will would rather dismiss
  one with context than miss something important.
  *(2026-11-11 lance: #9858.1, .4)*

## Do less

- Push for proof in unit tests, not integration tests. Don't block on an
  end-to-end test for a small change through well-tested code. Name the
  concrete unit or contract test (a pure function or in-memory fixture); avoid
  tests that build full indexes or other expensive fixtures.
  *(2026-11-10 lance: #10914.1; 2026-11-11: #9858.2, .3)*
- Adding new caches. Prefer making the operation cheap to construct. If
  caching is really needed, reuse the project's existing size-tracked caches
  rather than adding a separate one; one place to manage eviction is easier.
  *(2026-11-11 lance: #10868, Will's stated preference)*
- Don't polish docs for transitional or compatibility behavior; ask whether
  they're needed at all. *(2026-11-11: lancedb #4944.8)*
- Don't ask authors to split a PR unless it's very large and the parts are
  clearly unrelated, and never as a blocking item.
  *(2026-11-12 lance: #10923.1, #10907.8 rejected; Will: "only if huge")*
- In OTHERS, list only other reviewers' comments that are still open and worth
  a +1. Leave out ones already fixed or answered.
  *(2026-11-12 lance: #9858.2, .3, #9604.1)*

## Calibration

- Ask for a breaking-change marker only when the public API changes. A `pub`
  item documented as internal doesn't need one.
  *(2026-11-12 lance: #10829.2 vs #9604.3)*
- Rank rare edge cases low (an unusual combination of inputs plus a specific
  old state). A gap that fails safe, with a workaround named in the error, is
  non-blocking. *(2026-11-11 lancedb: #4944.1; 2026-11-12 lance: #9604.4)*
- If nothing is blocking, recommend approve and call the rest "encouraged".
  Don't hold back approval with a plain comment review, unless Will doesn't
  endorse the PR's direction (e.g. it optimizes a path being phased out); then
  a COMMENT asking for justification fits.
  *(2026-11-12 lance: #10463 comment → APPROVE; #8880)*
- On an approve with nothing blocking, rank design and premise questions first
  and push test-helper, doc-wording and naming nits below the nit cutoff. High
  acceptance doesn't mean the volume was right.
  *(2026-11-12 lance: #10800, 91% accepted, pulse "too much")*
