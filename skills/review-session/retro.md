# Retro

Goal: turn this session's feedback into **general, repo-agnostic** guidance on
how this user reviews, and recalibrate how much to raise. Run it after the last
PR, or on whatever was closed if the user stops early.

Everything the retro writes goes into `PROFILE_DIR`. **Never edit
`SKILL_DIR`**, and never post anything anywhere.

## 1. Numbers

Compute these from `SESSION_DIR/log.jsonl` and show them as a small table, per
PR and in total:

- shown, accepted, edited, rejected, added by the user, held back
- acceptance rate: (accepted + edited) / shown
- rounds, pulse, drafted verdict versus final verdict

Look for these signals:

- **Too noisy:** low acceptance, many rejections, a pulse of "too much", or
  the user stopping before the remaining items.
- **Too quiet, or missing things:** the user added comments, or the pulse was
  "too little". Check each `added` item against the `held_back` drafts. A
  match means the cutoff was too strict. No match means the drafts missed it
  entirely.
- **Wrong rank:** an accepted item came in a late round while a rejected one
  came early.
- **Wrong blocking call:** the user flipped blocking ↔ non-blocking, or the
  verdict changed.

## 2. Self-reflection

In 3–5 short, honest bullets, say what went well or badly in how you ran the
session: round size, ordering, how clear the drafts were, and when you should
have stopped.

## 3. Ask the user

Ask one structured question with up to 3 parts. Ask about overall volume
(too much / about right / too little) and anything the numbers can't show,
such as "did the OTHERS items help?". Keep it light.

## 4. Propose changes; the user approves each one

Write each proposal as a short entry that cites its evidence (PR and item
IDs). There are four kinds:

- **Lessons** (`PROFILE_DIR/lessons.md`): "Do more: …", "Do less: …", or
  "Calibration: …".
- **Examples** (`PROFILE_DIR/examples.md`): the user's own `added` comments and
  heavily `edited` ones. Quote each with its label and one line of context.
  Examples are the best way to convey their voice.
- **Settings** (`PROFILE_DIR/config.json`, `comment-style.md`, or
  `standard.md`): for example a different round size or a formatting rule.
- **Skill feedback** (`PROFILE_DIR/core-feedback.md`): problems with the
  workflow itself rather than this user's taste, such as a step that was
  confusing, broken, or missing. These stay local. If the user wants to
  share one, they can open an issue on the skill's repo themselves. Never do
  it for them, because session notes can contain private code and
  discussion.

**Keep lessons repo-agnostic.** A lesson must be about how this user reviews:
clarity, efficiency, test rigor, phrasing, what's worth blocking on. It must
not be about a particular codebase's design or conventions. Record
repo-specific learnings in `PROFILE_DIR/parked.md` under the repo's name, so
they aren't lost. When writing examples, generalize repo-specific nouns where
the point survives. For instance, "this manifest mutation is never persisted"
becomes "this state change is never persisted".

Show proposals in batches of up to 6. The user accepts, edits, or rejects
each one. Write only the accepted ones.

**Budget.** More rules dilute each rule, so the files have hard caps:

- `lessons.md`: about 800 words at most
- `examples.md`: 10 examples at most

Before the first batch, show one line with each file's current size next to
its cap: `wc -w` for lessons, and a count of `## ` headings for examples. When
a file is at its cap, every addition must come with a merge, trim, or
deletion that keeps it under the cap. One in, one out.

**Reword lessons instead of repeating them.** If a draft broke an existing
lesson, don't add a second lesson that says the same thing. Propose making the
existing one sharper or more concrete, or moving it higher in its section.
Merge near-duplicates, and propose deleting lessons this session contradicted.

## 5. Archive

Copy `log.jsonl` to `PROFILE_DIR/log/<session-dir-name>.jsonl`. Append a
one-line summary (date, repos, PR count, acceptance rate, pulse) to
`PROFILE_DIR/log/index.md`. Future retros use these to spot trends across
sessions.
