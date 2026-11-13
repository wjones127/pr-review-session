# pr-review-session

An agent skill for **pair reviewing** pull requests. It works in
[Claude Code](https://claude.com/claude-code) and should work in
[Codex](https://developers.openai.com/codex) or any agent that supports
skills. The agent doesn't review for you. It reviews alongside you.

```
/review-session 1234 1240 other-repo#88     # Claude Code
$review-session 1234 1240 other-repo#88     # Codex
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

review-session runs on **your own agent**: the coding agent you already use,
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

- **Drafts in the background.** One drafting agent per PR writes a review: a
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

Or copy `skills/review-session` into your agent's skills folder:
`~/.claude/skills/` for Claude Code, `~/.agents/skills/` for Codex.

Update with `npx skills update` (or by copying again). Your profile lives outside the skill folder,
so updates never overwrite what it has learned.

**Requirements:**
- a coding agent that supports skills and can run shell commands
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

## Agent support

| Agent | Status |
|---|---|
| Claude Code | Tested. Drafts run as background subagents. |
| Codex | Untested. |
| Others | Untested. Should work if the agent can run `gh`. |

Drafting adapts to what your agent can do. The mode is saved as `drafting` in
`config.json`:

- `background`: subagents run in the background and report back. You can chat
  while they work. (Claude Code)
- `blocking`: subagents run, but the session waits for them. The first PR is
  checked out first so you can read while you wait. (Codex's default)
- `command`: the session runs `drafter_command` in the background for each PR,
  e.g. `codex exec …` or `claude -p …`. This gives you background drafting in
  any agent that can start background shell jobs.
- `inline`: no subagents. The session drafts each PR itself just before
  presenting it.

If your agent has no multiple-choice question tool, questions come as
numbered options in chat.

**Codex:** the sandbox must allow network access (for `gh`) and writes to
your profile folder, e.g. `--sandbox workspace-write` with network enabled,
plus `~/.config/review-session` as a writable root.

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
submodules**. Open that folder in your editor and start your agent there:

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
