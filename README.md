# pr-review-session

A [Claude Code](https://claude.com/claude-code) skill for **pair reviewing**
pull requests with an agent. The agent doesn't review for you. It reviews
alongside you.

```
/review-session 1234 1240 other-repo#88
```

You stay the reviewer. You read the code, write your own comments, and decide
the verdict. The agent does the parts that wear you down: a first pass over
every diff, checking out branches, digging through the codebase when you ask
"wait, what calls this?", and turning your half-formed reactions into
well-phrased comments. The review that gets posted is yours, under your name,
in your voice.

## Why pair instead of delegate

Automated review bots are shared, public agents. The whole team works with one
bot, it posts on the PR under its own account, and you talk to it in PR
threads that everyone can see. That's a useful job, but a different one.

review-session runs on **your own agent**: the Claude Code you already use,
with your tools, your memory, your local checkouts, and whatever else you've
connected to it. It's there to help you think a PR through, not to take part
in the PR's conversation.

| | Review bot | review-session |
|---|---|---|
| Whose agent | Shared by the team, posts as itself | Yours, posts nothing until you close the PR |
| Who's reviewing | The bot | You, with the agent's help |
| Asking questions | In PR threads, visible to everyone | In private, as you read |
| Your own comments | A separate review | Part of the same review as the drafts you accepted |
| What gets posted | Whatever the bot found | Only what you accepted or wrote |
| What it learns | The project's or team's conventions | *Your* taste: what you rejected as noise, what you added that it missed, how you phrase things |

The private back-and-forth is the point. Reviewing goes faster when you can
ask questions of something that has already read the diff and can open any
file in your checkout. You don't have to hold the whole PR in your head, and
you don't have to post a half-formed question to find out whether it's a real
concern.

## How it works

- **Drafts in the background.** One subagent per PR drafts a review: a
  summary, ranked comments, and notes on what other reviewers already said.
  None of it floods your chat.
- **You review in conversation.** The session goes through PRs in the order
  you gave. Each PR is checked out (or linked) right away so you can read
  ahead. Draft comments come a few at a time, so you're never facing a wall of
  findings. Accept, edit, reject, or defer each one. Ask questions about the
  code whenever they come up. Dictate your own comments, or leave them in your
  editor or on GitHub. Either way, they end up in the same review.
- **Each PR is posted when you close it.** The review goes up in one batch,
  with an approve or request-changes verdict, merged into any pending review
  you already started on GitHub. If you stop early, everything you finished is
  already posted.
- **It learns your taste.** After each session, a short retro looks at what you
  accepted, rejected, and added yourself. It then proposes updates to your
  personal lessons and example comments, and you approve each one. The next
  session's drafts sound more like you and flag less noise.

## Install

```bash
npx skills add wjones127/pr-review-session
```

Update with `npx skills update`. Your profile lives outside the skill folder,
so updates never overwrite what it has learned.

**Requirements:**
- Claude Code
- the [GitHub CLI](https://cli.github.com/) (`gh`), logged in with
  `gh auth login`

## First run

The first `/review-session` asks four quick questions:

1. **Comment style.** Conventional comments (`issue(blocking): …`), plain
   prose, or your own description.
2. **Workflow.**
   - *Check out locally:* follow along in your editor.
   - *GitHub in a browser:* nothing is checked out.
3. **Posting.**
   - *Post when you say "close".*
   - *Ask first.*
   - *Leave as a pending review that you submit yourself.*
4. **Learn your voice (optional).** It can read your recent GitHub review
   comments and propose a short "how I review" note plus a few example
   comments, so drafts sound like you from the first session. You approve
   everything before it's saved. You can also start from the
   [example profile](skills/review-session/example-profiles/wjones127).

Run `/review-session setup` to redo setup at any time.

## Your profile

Everything personal lives in `~/.config/review-session/` (or
`$REVIEW_SESSION_HOME`). All of it is plain text you can edit:

| File | What it is |
|---|---|
| `config.json` | Workflow, posting mode, items per round (default 4) |
| `standard.md` | What you look for and what you block on |
| `comment-style.md` | How posted comments are formatted |
| `lessons.md` | What retros learned: do more, do less, calibration |
| `examples.md` | Comments you wrote, as examples of your voice |
| `core-feedback.md` | Notes from retros about the skill itself |
| `parked.md` | Repo-specific learnings, kept out of the general lessons |
| `log/` | Per-session logs that retros use to spot trends |

Nothing in the profile leaves your machine. The only thing ever posted is the
review for a PR you close. If the retro finds something wrong with the skill
itself, it notes it in `core-feedback.md`. Feel free to open an issue with
anything you're comfortable sharing.

## Reviewing several repos at once

For the local-checkout workflow, make a folder that holds a standalone
checkout of each repo. Clones or `git worktree`s both work, but **not git
submodules**. Open that folder in your editor and start Claude there:

```
~/review/
  server/   ← worktree of your server repo
  client/   ← worktree of your client repo
```

VS Code finds repos one folder down, so the GitHub Pull Requests extension
shows all of them. When PRs in one session are related, for example the
client and server halves of a change, the drafts check them against each
other.

## License

MIT
