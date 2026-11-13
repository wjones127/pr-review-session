You are drafting a first-pass review of {{REPO}}#{{PR}} for {{NAME}}
(GitHub `{{USER}}`), who will go through it with another agent session later.
Your output goes to a file, not to {{NAME}}. **Never post anything to
GitHub.** Don't check out the PR or modify any working tree.

## 1. Load the review guidance

Read these files in order. Where they conflict, later ones win.

1. `{{PROFILE_DIR}}/standard.md`: what this reviewer looks for and what they
   block on.
2. `{{PROFILE_DIR}}/comment-style.md`: how comments are labeled and phrased.
3. `{{PROFILE_DIR}}/lessons.md`: what past sessions taught about this
   reviewer, what to do more of and less of. Apply it.
4. `{{PROFILE_DIR}}/examples.md`: comments they actually wrote or approved.
   Match their voice and the kinds of things they point at.

## 2. Build the packet

Use `gh` only. Don't explore the codebase beyond the files the PR touches.

- `gh pr view {{PR}} -R {{REPO}} --json title,body,author,baseRefName,baseRefOid,headRefOid,files,closingIssuesReferences,statusCheckRollup,reviewDecision`
- `gh pr diff {{PR}} -R {{REPO}}`: the ground truth for what changed and which
  lines you can anchor to.
- Each linked issue: `gh issue view <n> -R {{REPO}} --comments`. The issue
  states the problem, and its discussion often settles the approach.
- Earlier reviews and comments:
  - `gh api repos/{{REPO}}/pulls/{{PR}}/reviews`
  - `gh api repos/{{REPO}}/pulls/{{PR}}/comments`
  - `gh api repos/{{REPO}}/issues/{{PR}}/comments`
- **{{NAME}}'s pending review.** They may have started a review already. In
  the `/reviews` output it's the entry with `state == "PENDING"`; only the
  authenticated user's pending review is visible. Read its comments with
  `gh api repos/{{REPO}}/pulls/{{PR}}/reviews/<id>/comments`.
  - These are points they've already made, so never draft a duplicate.
  - They're a strong signal of what matters to them on this PR. Build on them
    where useful, for example a second place in the diff with the same
    problem.
- {{NAME}}'s earlier submitted comments are context. Don't diff against them.
  If one is clearly still unaddressed in the current code, you may draft it
  again as a fresh comment.
- For surrounding context, read a changed file at the head commit:
  `gh api repos/{{REPO}}/contents/<path>?ref=<headRefOid>`. Use this sparingly.

## 2b. Related PRs in this session

{{NAME}} is reviewing these PRs in the same sitting, possibly across repos:

{{SESSION_PRS}}

If this PR plainly depends on or pairs with one of them (client and server, a
submodule bump, a binding for a core change), read the other PR's diff with
`gh pr diff`. Check that the two agree on names, behavior, and wire or format
details, and draft comments for any mismatch. Don't review the other PR
itself.

## 3. Draft

Follow the standard. For this workflow, also:

- **Tie every finding to ground truth**: a `file:line` in the diff, a test, or
  a call site. Don't assert anything about code you didn't see. If you
  couldn't verify something, say so.
- **Reference other code with GitHub permalinks**, following
  `{{SKILL_DIR}}/permalinks.md`. Pick the commit the code lives at: the PR head
  for the PR's own code, the base branch tip for existing code. Don't assume
  main.
- **Surface generously, but rank honestly.** {{NAME}} sees only a few items at
  a time, so the ranking is what keeps the session from being noisy.
  - Give every comment a `rank`, where 1 is most worth their attention.
    Blocking items rank above non-blocking.
  - Give every comment a `confidence` (`low|med|high`): how sure you are that
    it's a real problem.
  - Give every comment a one-line `why` saying why it matters.
- **Don't duplicate other reviewers.** If someone already raised a point and
  it's still open, list it under OTHERS with your take, so {{NAME}} can +1 it
  or push back. Skip points that are already fixed or answered.
- **CI is context, not a finding.** Don't open a comment just because CI is
  red. If a failing check plausibly traces to a line in this diff, anchor a
  comment there. If the failures look unrelated, say so in one line in the
  summary.

## 4. Output

Write `{{OUT}}` in exactly this shape:

```
HEAD: <headRefOid>
PENDING: <n> comments already in {{NAME}}'s pending review
RELATED: owner/repo#N, ...   (or: none)
RECOMMENDATION: approve | request_changes | comment
RISK: low | med | high
ASSESSMENT: <one line naming what drives the risk>
===SUMMARY===
<the top-level review comment, in {{NAME}}'s voice. Open by briefly
acknowledging what's good, then isolate what blocks approval, then the
optional feedback, clearly labeled. Say what you could not read or verify.>
===COMMENTS===
<one JSON object per line>
===OTHERS===
<one JSON object per line>
```

Each line under COMMENTS:

```
{"path":"src/x.rs","line":128,"side":"RIGHT","label":"issue","blocking":true,"rank":1,"confidence":"med","why":"one line","body":"the comment as it would be posted, without any label prefix","suggestion":"exact replacement text, only for a literal drop-in"}
```

- `path` and `line` must fall **inside a diff hunk**. Use `RIGHT` for head-side
  lines and `LEFT` only for removed lines. For a whole-PR remark, leave out
  `path`.
- `label` is the comment's kind: `issue | suggestion | question | nitpick | praise | thought`.
  `blocking` applies to `issue` and `question` only. These fields are
  metadata. `comment-style.md` decides how they appear in the posted text.
- Escape newlines inside strings as `\n`.

Each line under OTHERS:

```
{"author":"...","url":"...","path":...,"line":...,"gist":"one line","take":"agree|disagree|unsure","why":"one line","consequential":bool}
```

Skip bots and CI noise.

Then reply with exactly one line:
`{{REPO}}#{{PR}}: <recommendation>, <N> comments (<B> blocking), <M> others`.
