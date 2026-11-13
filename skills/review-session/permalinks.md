# Referencing code in posted comments

This applies to text that gets **posted to GitHub**: review comment bodies and
the summary. Links shown in chat follow the rules in SKILL.md instead.

An inline comment is already anchored to its line, so it needs no link to
itself. Use permalinks when a comment points at *other* code: a second spot
with the same problem, the existing implementation to copy, a caller that
breaks, or what a line looked like before the change.

Use this form:

```
https://github.com/OWNER/REPO/blob/<full-40-char-sha>/path/to/file.rs#L211-L221
```

- Always use a full commit SHA, never a branch name. Branch links drift, and
  GitHub won't render them as snippets.
- `OWNER/REPO` is the PR's **base** repo, even when the commit is on a fork's
  branch. GitHub resolves fork commits through the base repo, and only
  same-repo links render as snippets.

## Which commit

Pick the commit the code you mean lives at. Don't default to main.

| You're referencing… | SHA to use |
|---|---|
| Code as the PR changes it, including added or changed lines elsewhere in the diff | The PR head: `headRefOid` from `gh pr view <n> --json headRefOid` |
| Code the PR deletes, or "before" behavior | The base side: `gh pr view <n> --json baseRefOid` |
| Existing code the PR doesn't touch, such as a sibling implementation or a caller | The base branch's current tip: `gh api repos/OWNER/REPO/commits/<baseRefName> --jq .sha` |
| Code in another repo (e.g. a related PR in this session) | That repo's PR head or main tip, by the same rules |

Line numbers must match that exact commit. Verify them with
`git show <sha>:<path>` (fetch first if the SHA isn't local). Don't trust the
working tree: the local checkout can lag if the author pushed after we checked
out. Make sure `git rev-parse HEAD` equals `headRefOid` before using local line
numbers for the PR head.

## Two forms

1. **Showing a snippet.** Put the permalink on its own line; GitHub renders it
   inline as code. Always use a range (`#L211-L221`), never a single line.
   Pick the smallest enclosing block that makes the point, at most ~15 lines.
2. **Mentioning an item in prose.** Make the item's name the link text, e.g.
   [`FragReuseIndex::remap`](permalink), pointing at the item's first line.

Snippets only render for permalinks into the same repo as the PR. For code in
another repo, put the permalink on its own line, followed by a fenced code
block with the snippet.
