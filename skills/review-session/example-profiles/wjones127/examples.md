# Example comments

Comments Will wrote or substantially edited during `review-session`, kept as
voice and substance exemplars. Each entry has the label, one line of context,
and the comment text. Repo-specific nouns are generalized where that keeps
the point.

## suggestion(blocking) — format change gated as experimental/unstable

> I don't think this needs to be experimental. I think we can just make it a
> feature flag from the start and not label it unstable. Usually we do that for
> something that's really complex. But this is merely changing the file path
> for a file where the semantics are already well established and well tested.

## question — persisted field that is both a free string and a rigid format

> We both give a string path and also specify a required format
> `{uuid}.{extension}`. Have you considered either:
> 1. Just allow a path, allow arbitrary format. (Most flexible)
> 2. Just take a UUID here as 16 bytes, and stick to the exact format
>    `{uuid}.{extension}`. Extension can be derived from the type field.
>    (Most compact)

## question — option combination that silently does nothing

Context: a maintenance task scoped to one kind of work plans nothing unless a
threshold is also set. Rather than demand an error, ask what other triggers the
author intends, and ask for them to be documented.

> With `scope="repack_columns"` and no `max_data_files_per_fragment`, this run
> plans nothing. As it stands, that looks like a trap for someone who asked
> only for repacks and forgot the limit.
>
> But I can see it making sense if there are other triggers. For example,
> after an in-place column rewrite, a fragment's files carry tombstoned columns
> that are dead in the latest version. That's dead weight in columnar form,
> much like a lot of deletions, and it could be worth repacking even when no
> file-count limit is set.
>
> What triggers do you have in mind for planning a repack, now or later?
> Whatever they are, could the docs list the conditions that cause one to be
> planned? Then a run that plans nothing is explainable.

## question — perf PR that caches an expensive derived value

> Before we add a cache here, I'd like to understand why building this schema
> is expensive in the first place. The filterable schema is the dataset schema
> plus a few fixed system columns, yet we deep-copy and convert all N columns
> just to append ~4–8 fields, while the filter itself usually touches 1–3
> columns. Could we make the operation cheap instead?

## question — two methods where one path would do

> Every production caller now uses `load_shared`, and `load` is only reached
> from a test and from `load_shared`'s default body. Since the trait is
> documented as internal with no stability guarantees, could we collapse these
> into one method that returns the `Arc`? That removes the duplicated combine
> logic here and leaves one path to implement.

## issue(blocking) — ask for unit coverage, not an integration test

> The address-domain switch hinges on the version ↔ `results_are_row_addresses`
> mapping: it's stamped at train/merge time and read back at load. As far as I
> can tell nothing pins that. Could we add a unit test (in-memory, like the
> existing ones) that checks a v1 segment loads with `true`, a v0 one with
> `false`, and that merging keeps the domain? The query-time translation is
> shared with other index types, so I don't think we need a full dataset-level
> test.

## suggestion — spec text that describes behavior instead of stating rules

> Same voice point here: state what a restore must do rather than what it does.
>
> ```suggestion
> A restored manifest must use the target snapshot's `Manifest.clustering` list
> unchanged. Definitions created in later snapshots must not be merged into it, so
> restored fragments and index segments keep the definitions they reference.
> ```

## suggestion — new self-contained types added to an already huge module

> This module is already ~4.7k lines. These new types are a self-contained block
> of pure arithmetic with no I/O or scheduler dependencies. Could they live in
> their own submodule? Their unit tests would move there too, so the rules are
> tested independently of the callers. Since the code is new, moving it now is
> cheap. If you'd rather split the module up more broadly, a dedicated refactor
> PR is a fine place for this too. That would keep unrelated moves out of this
> one.
