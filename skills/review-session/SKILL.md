---
name: review-session
description: Interactive PR review session. Use only when explicitly invoked as `/review-session <PR numbers or URLs>` (or `/review-session setup`). Fans out background subagents that draft a review per PR, then walks the PRs one at a time in the order given, discussing a few draft comments per round, posting each PR's review when it is closed, and ending with a retro that updates the user's review profile so future drafts fit them better.
---

# review-session

Review a list of PRs together with the user. Background subagents draft each
review. You and the user go through each draft a few items at a time, discuss
the code, settle the comments, and post. At the end, a retro turns the user's
feedback into lessons that make the next session's drafts better.

## Paths

- `SKILL_DIR`: the directory containing this file. Use its absolute path,
  because subagents need it.
- `PROFILE_DIR`: `$REVIEW_SESSION_HOME` if set, else `~/.config/review-session`.
  This is the user's personal profile. Skill updates never touch it.
  - `config.json`: preferences (see below)
  - `standard.md`: what this user looks for and blocks on
  - `comment-style.md`: how posted comments are formatted
  - `lessons.md`, `examples.md`: learned from retros; drafts follow them
  - `core-feedback.md`: retro notes about the skill itself (local only)
  - `parked.md`: repo-specific learnings with no home yet
  - `log/`: archived session logs
  - `sessions/<YYYY-MM-DD>-<HHMM>/`: this session's working files (`SESSION_DIR`)

**Never edit files in `SKILL_DIR`.** They're replaced on every update. All
learning goes into `PROFILE_DIR`.

## 0. Load the profile

If `PROFILE_DIR/config.json` doesn't exist, or the argument is `setup`, follow
`SKILL_DIR/setup.md`, then continue with the PR list if one was given.

`config.json`:

```json
{
  "github_user": "octocat",
  "name": "Mona",
  "workflow": "local_checkout",
  "posting": "post_on_close",
  "round_size": 4
}
```

- `workflow`:
  - `local_checkout`: each PR is checked out locally, so the user can follow
    along in their editor.
  - `browser`: nothing is checked out. The user has GitHub open in a browser.
- `posting`:
  - `post_on_close`: saying "close" is approval to post.
  - `confirm`: show the final batch and wait for an explicit yes.
  - `pending_only`: add everything to the user's pending review and leave
    submitting to them.
- `round_size`: items per round. Default 4. Never more than 6.

Address the user by `name`. Read `standard.md` and `comment-style.md` now. You
need both for discussing and posting.

## 1. Setup

1. **Map repos to checkouts** (`local_checkout` only). The session runs from a
   **review root**: either one repo checkout, or a folder whose immediate
   subfolders are standalone checkouts (clones or worktrees, **not
   submodules**). In VS Code, opening the review root shows every repo in the
   GitHub Pull Requests extension.

   Look at the current directory and each immediate subfolder that is a git
   checkout, and collect its remotes (`git -C <dir> remote -v`). A checkout
   belongs to `owner/repo` if any remote points at it. When several checkouts
   match, prefer the one where `origin` or `upstream` points at it.
2. **Resolve the PR list.** Accept:
   - full URLs
   - `owner/repo#N`
   - `repo#N`, matched against the mapped repos, or against the current repo in
     `browser` mode
   - bare `N`, only if exactly one repo is in play

   In `local_checkout` mode, if a PR's repo has no checkout, stop and ask. Offer
   to add a worktree for it under the review root.
3. Make `SESSION_DIR` and write `session.json` with:
   - the repo → dir map. Each repo gets a `link_dir`: its checkout path relative
     to the session's working directory, e.g. `lance` or `.`.
   - the PR list in the user's order
   - the start time
   - an empty `arrivals` list

   Name files `<repo>-<n>.*`. Two repos can share a PR number.
4. Launch **one background subagent per PR**, all in one message (`Agent`,
   `subagent_type: general-purpose`). The prompt is `SKILL_DIR/subagent.md` with
   these filled in: `{{REPO}}`, `{{PR}}`, `{{OUT}}`, `{{SKILL_DIR}}`,
   `{{PROFILE_DIR}}`, `{{USER}}` (github_user), `{{NAME}}`, and
   `{{SESSION_PRS}}`, one line per PR as `owner/repo#N title`. The subagent
   replies with one line only.
5. In the same turn, make the **first PR in the user's list** current (§3), so
   they can start reading while the drafts run. Tell them in one line: "N
   reviews drafting. #X is ready to read; I'll present it when its draft
   lands."

## 2. The queue

- **The order is the order the user gave.** People often put the most
  important PR first. Never reorder by arrival time or relatedness. If they
  ask to skip ahead or reorder, do it.
- When a subagent finishes, append `{pr, at}` to `arrivals`. **Never relay its
  content.**
  - If it's the current PR, present it. If the user is in the middle of a
    question, answer that first.
  - If it isn't, say only "#N draft ready", and say nothing while a round is in
    progress.
- While the current PR's draft is pending, the user may read the code and ask
  questions. Answer them.
- If a subagent fails, report it in one line and offer to retry it or to
  review that PR from scratch together.

## 3. Reviewing one PR

**Make it current.** This happens before its draft is ready.

1. Run `gh pr view <n> -R <o>/<r> --json state,headRefOid,headRefName,url`. If
   the PR is merged or closed, say so in one line and offer to skip it.
2. In `local_checkout` mode, run `gh pr checkout <n>` from inside that repo's
   checkout. Do the `cd <dir>` as its own Bash call first. **Don't pass
   `--detach`, and don't check out a bare commit.** Editor PR extensions only
   recognize the PR on the tracking branch that `gh pr checkout` creates. If
   the tree is dirty, stop and ask. Never stash or discard anyone's changes.
   Also stop and ask if the branch is checked out in another worktree, or if
   the local branch has diverged from the PR. Never reset or delete a local
   branch.
3. In `browser` mode, give the PR's "Files changed" URL.
4. Mark it current in `session.json`.

**Present it** once `SESSION_DIR/<repo>-<n>.review.md` exists:

1. Compare the review file's `HEAD:` line with the current head. If the author
   has pushed since, diff the two
   (`gh api repos/<o>/<r>/compare/<old>...<new>`), say in one line what
   changed, and remap any draft anchors that moved.
2. **Check for the user's pending review.** The user may have started leaving
   comments in GitHub or their editor before the draft landed. Run
   `gh api repos/<o>/<r>/pulls/<n>/reviews`. Only your own pending review is
   visible, and it's the one with `state == "PENDING"`. Save its `id` and
   `node_id`, then read its comments:
   `gh api repos/<o>/<r>/pulls/<n>/reviews/<id>/comments`. Then:
   - mention them in the intro, one short line each
   - log each one with `source: "user_pending"`, disposition `added`
   - drop or merge any draft that makes the same point, and say which
   - **re-check before each round and at close**, since the user keeps adding
     comments while you talk
3. Present the intro (format below). If the review file's `RELATED:` line
   names other PRs in this session, mention them.

**Build the item list** from the review file:

- Drafted comments (`===COMMENTS===`).
- Other reviewers' open points worth reacting to (`===OTHERS===`). These are
  for awareness: the user may +1 one, disagree, or ignore it. Never re-raise
  someone else's point as our own.

If a draft question is a factual check on existing code ("is X shallow?",
"does Y already exist?"), research it before presenting, then drop it or
answer it inline. Do the same for any claim a draft makes about code outside
the diff, such as how a called type behaves or what a helper already does.
The drafters only read the diff, so check the claim and fix the draft before
the user sees it.

Order: blocking issues, then blocking questions, then consequential OTHERS
items, then everything else by `rank`. Give each item a stable ID like `#N.3`.

**Formatting.** People often read this in a narrow pane. Use markdown
structure and whitespace, keep lines short, and never put the label,
location, and confidence on one long line.

Intro:

```markdown
## #1234 · feat: short PR title
by @author · [PR](url) · checked out on `branch-name`

- **What:** one or two sentences.
- **Draft verdict:** request changes · risk med
- **Queue:** 1 blocking · 3 questions · 2 from other reviewers · 6 others
- **CI:** one line, only if it says something non-obvious
```

Each round item, with `---` between items:

```markdown
### 3 · question (blocking)
myrepo/src/io/reader.rs:122 · conf med

> Draft body exactly as it would be posted, formatted per comment-style.md.
> Split it into short paragraphs when it makes more than one point.

*Why it matters:* one line.
```

- The heading uses the short per-PR number (`3`). The log uses the full `#N.3`.
- Put the location on its own line. Use `top-level` for a review-level
  comment.
- **Links in chat:**
  - `local_checkout`: write **plain text** `<link_dir>/<repo-relative path>:<line>`,
    relative to the session's working directory. Never wrap it in a
    markdown link. Terminals such as VS Code's make plain `path:line` text
    clickable and jump to the line, while a markdown link target fails with
    "Invalid URL". Don't use absolute paths, since editors open those outside
    the workspace, and don't use `#L<line>`.
  - `browser`: link to GitHub at the PR head SHA,
    `https://github.com/<o>/<r>/blob/<sha>/<path>#L<line>`.
- **Text that will be posted** uses GitHub permalinks, never local paths.
  Follow `SKILL_DIR/permalinks.md`. This includes comments the user dictates
  or edits. Before posting, check that no body contains a local path.
- For an OTHERS item, the heading is `### 5 · from @author (agree)`, followed
  by the quoted gist, then *Our take:*.
- Head each round `## Round 1 — items 1–4 of 12`. End it with one line:
  "Reply per item: accept / edit / reject / defer (e.g. `1 accept, 2 reject: …`)."

**Rounds.** Show `round_size` items per round, fewer only if fewer remain. The
user responds to each item:

- accept
- edit
- reject (a reason is welcome but never required)
- defer to a later round

They may also dictate their own comment, for example about lines selected in
their editor. Record it as `added`, anchored to those lines.

Between rounds, answer questions from the code: the checkout, or `gh` in
`browser` mode. This is when you explore the codebase; the subagents didn't.
If what you find changes a draft, update it and say so.

**Stopping.** Keep going round by round through all drafts. Near the bottom,
ask "K nits left. Show them?" only when what remains is pure nits or typos.
Log items the user never sees as `held_back`; the retro checks this cutoff.

**Close.** When the user says close (or done, or ship it):

1. Settle the verdict. The default is the draft's recommendation, adjusted for
   what was accepted: any accepted blocking item means `REQUEST_CHANGES`,
   otherwise `APPROVE`. Use a plain `COMMENT` only if the user asks for one.
2. Rewrite the top-level summary from the draft to match the final comments.
3. Show the final batch: verdict, summary, and each comment as path:line plus
   its first line. Then follow `posting`:
   - `post_on_close`: post right away unless the user objects in the same
     message.
   - `confirm`: ask with one `AskUserQuestion` (**Post** / **Edit first**) and
     post only on a yes.
   - `pending_only`: add the comments to the pending review without
     submitting it. Tell the user the suggested verdict and that it's ready
     to submit on GitHub.
4. Post **exactly one review**. Re-check for a pending review first: GitHub
   allows one per user per PR, and the user's own pending comments must end up
   in the same review as ours. Format each body per `comment-style.md`, and
   put any `suggestion` text into the body as a ```` ```suggestion ```` block.
   - **A pending review exists:** add our comments to it, then submit it.
     Never create a second review.
     ```
     gh api graphql -f query='mutation($r:ID!,$p:String!,$l:Int!,$s:DiffSide!,$b:String!){
       addPullRequestReviewThread(input:{pullRequestReviewId:$r,path:$p,line:$l,side:$s,body:$b}){thread{id}}}' \
       -f r=<node_id> -f p=<path> -F l=<line> -f s=RIGHT -f b=<body>
     ```
     ```
     gh api graphql -f query='mutation($r:ID!,$e:PullRequestReviewEvent!,$b:String){
       submitPullRequestReview(input:{pullRequestReviewId:$r,event:$e,body:$b}){pullRequestReview{url}}}' \
       -f r=<node_id> -f e=APPROVE|REQUEST_CHANGES -f b=<summary>
     ```
     If the pending review already has a body, merge it into the summary
     rather than overwriting it.
   - **No pending review:** write
     `{"commit_id": <head sha>, "event": ..., "body": ..., "comments": [{path, line, side, body}]}`
     to `SESSION_DIR/<repo>-<n>.post.json`, then run
     `gh api repos/<o>/<r>/pulls/<n>/reviews --method POST --input <file>`.
     In `pending_only` mode, leave `event` out. That creates a pending review
     the user submits.

   If GitHub rejects an anchor, move that comment into the summary rather than
   dropping it. If something fails partway, stop and say exactly what was
   added. Never retry blindly, since that can duplicate comments.
5. Ask the pulse question with one `AskUserQuestion`: "For #N, the amount I
   raised was…" with **Too much / About right / Too little**. A note is
   optional.
6. Append the PR's records to `SESSION_DIR/log.jsonl` and mark the PR done.
   Make the next PR current right away, and present it if its draft is ready.

## 4. Disposition log

`SESSION_DIR/log.jsonl` holds one JSON object per line:

- Item:
  `{"type":"item","pr":"owner/repo#N","id":"#N.3","source":"draft|others|user|user_pending","label":...,"blocking":bool,"rank":int,"confidence":"low|med|high","round":int|null,"disposition":"accepted|edited|rejected|deferred_unseen|held_back|added","reason":"...","draft_body":"...","final_body":"..."}`
- PR:
  `{"type":"pr","pr":"owner/repo#N","rounds":int,"drafted_verdict":...,"final_verdict":...,"pulse":"too_much|about_right|too_little","pulse_note":"...","posted_review_url":"..."}`

Log as you go, not only at close, so stopping early still leaves a record.

## 5. Retro

Run it after the last PR, or when the user stops early. Follow
`SKILL_DIR/retro.md`.

## Rules

- Post only at close, and only for the PR being closed.
- Never stage, commit, or push anything in a repo under review.
- Never edit `SKILL_DIR`.
- Keep chat brief. The review files hold the detail, so pull from them when
  needed.
