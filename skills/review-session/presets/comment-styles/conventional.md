# Comment style: conventional comments

Posted comments follow [Conventional Comments](https://conventionalcomments.org):

`<label>(<decoration>): <subject>`

| Label | Meaning | Merge condition? |
|---|---|---|
| `issue(blocking)` | Must be resolved before merge. | Yes |
| `issue(non-blocking)` | Should be fixed, but it's the author's call. | No |
| `suggestion` | Optional improvement. | No |
| `question` | Needs an answer before I can assess. Add `(blocking)` when the answer gates approval. | Soft |
| `nitpick` | Trivial or style. | No |
| `praise` | A positive callout. | No |
| `thought` | An idea to consider, no action needed. | No |

## Rules

- Prefix each posted body with its label and decoration, built from the
  draft's `label` and `blocking` fields, e.g. `issue(blocking): `.
- Use `(blocking)` deliberately. Over-calling reads as timid, and
  under-calling reads as careless. When unsure, say what would make it
  blocking.
- Keep the subject to one crisp sentence. Put detail in the paragraphs below
  it.
- A blocking issue states its acceptance criterion.
