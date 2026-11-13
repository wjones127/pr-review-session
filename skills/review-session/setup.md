# First-run setup

Creates `PROFILE_DIR`. Keep it quick: one round of questions, an optional
voice pass, then straight into reviewing.

## 1. Check prerequisites

- Run `gh auth status`. If it fails, tell the user to run `gh auth login`,
  and stop.
- Get the login with `gh api user --jq .login`, and a display name from
  `gh api user --jq .name`. If there's no name, use the login.

## 2. Ask (one structured question with four parts)

See *Agent capabilities* in `SKILL.md` for what a structured question is in
your agent.

1. **Comment style.** "How do you like your review comments formatted?"
   - **Conventional comments**: labels such as `issue(blocking):` and
     `suggestion:` ([conventionalcomments.org](https://conventionalcomments.org)).
   - **Plain prose**: no labels. Blocking items say so in words.
   - Other: they describe it, and you write a custom `comment-style.md` from
     the description.
2. **Workflow.** "How do you want to read the code while we review?"
   - **Check out locally**: each PR is checked out with `gh pr checkout`, so
     you can follow along in your editor (for example VS Code with the GitHub
     Pull Requests extension). Works with one repo or a folder of several.
   - **GitHub in a browser**: nothing is checked out. Links point to GitHub.
3. **Posting.** "When we close out a PR, how should the review be posted?"
   - **Post when I say close** (Recommended): the fastest option. "Close"
     counts as approval.
   - **Ask me first**: show the final review and wait for an explicit yes.
   - **Leave it pending**: comments go into your pending review, and you
     submit it on GitHub.
4. **Learn your voice.** "Want me to read your recent GitHub reviews and take
   notes on your voice and what you tend to flag? Drafts will sound more like
   you from the first session. Everything stays on this machine."
   - **Yes**
   - **Not now**: you can run `/review-session setup` later.
   - **Start from the example profile**: copy a published sample profile as a
     starting point.

## 3. Write the profile

Create `PROFILE_DIR` with `log/` and `sessions/` subfolders, and add:

- `config.json`: `github_user`, `name`, `workflow` (`local_checkout` or
  `browser`), `posting` (`post_on_close`, `confirm`, or `pending_only`),
  `round_size: 4`, and `drafting`, chosen from the tools you actually have
  (see *Agent capabilities* in `SKILL.md`). Tell the user which drafting mode
  you picked in one line.
- `standard.md`: copied from `SKILL_DIR/presets/standard.md`.
- `comment-style.md`: copied from the chosen
  `SKILL_DIR/presets/comment-styles/*.md`, or written from their description.
- `lessons.md` and `examples.md`: copied from `SKILL_DIR/presets/`.
- `core-feedback.md` and `parked.md`: one heading each.

If they chose the example profile, copy `standard.md`, `lessons.md`, and
`examples.md` from `SKILL_DIR/example-profiles/<name>/` instead, and say it's
someone else's taste: the retros will shift it toward theirs. Keep the comment
style they picked in question 1.

## 4. Voice pass (only if they said yes)

1. Fetch their recent reviews. Use the GraphQL contributions API, which covers
   the past year:
   ```
   gh api graphql -f query='query($login:String!){user(login:$login){contributionsCollection{
     pullRequestReviewContributions(first:100){nodes{pullRequestReview{
       body state url repository{nameWithOwner isPrivate}
       comments(first:30){nodes{body path}}}}}}}}' -f login=<login>
   ```
   If there are fewer than 30 substantive comments, say the sample is thin and
   use what's there.
2. Analyze them for:
   - **Voice**: length, hedging, questions versus directives, warmth, how they
     open and close a review, how often they use suggestion blocks.
   - **Substance**: what they block on, what they treat as optional, and the
     recurring themes (tests, naming, API shape, scope, performance…).
   - **Verdicts**: how often they approve versus request changes.
3. Propose, and show them before writing anything:
   - A **"How I review"** section to add at the top of `standard.md`: 5–10
     bullets in their terms, covering voice and substance.
   - Up to **8 example comments** for `examples.md`. Pick ones that are
     typical and well put, and cover different labels. Give each a one-line
     context. Generalize repo-specific nouns where the point survives without
     them.
4. **Private repos.** Comments from private repos stay in their local profile.
   That's fine, but say which examples came from private repos, so they can
   drop those if they ever share the profile.
5. Write only what they approve.

## 5. Finish

Say in two lines where the profile lives and that every file in it can be
edited by hand. If a PR list was given, continue with SKILL.md §1. Otherwise,
say how to start: `/review-session <PRs>`.
