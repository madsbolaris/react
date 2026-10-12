# PR labeling dataset: first 10 real examples

This is an unencrypted snapshot of **10 real merged pull requests from the
public [primer/react](https://github.com/primer/react) repository**. Nothing in
these records comes from private `github/github-ui` data or the earlier
synthetic samples.

The snapshot was collected on October 12, 2026, using the GitHub REST API.
It is an MVP for inspecting and reviewing PR labeling data, not a trained model
or a statistically representative training/evaluation set.

## Files

- [Root dataset card](../README.md): standard Hugging Face `configs.data_files`
  and `dataset_info.features` metadata, including nested types and multi-label
  `List(ClassLabel)` semantics. The `examples` loading split is not a train/test
  assignment.
- [samples.jsonl](samples.jsonl): one complete PR record per line, in UTF-8 JSONL.
- [labels.json](labels.json): the six distinct labels observed on these PRs,
  with their actual GitHub IDs, descriptions, and colors.

This is an orphan data-only branch: its initial commit has no parents and
inherits no application files or history from another branch. It contains only
this dataset, Hugging Face dataset card, label catalog, documentation, and the source excerpts'
[MIT license](LICENSE) and copyright notice. The fork's default branch is
unchanged.

## Record shape

| Field | Contents |
| --- | --- |
| `id` | Stable source identity, such as `primer/react#8477` |
| `source` | Source repository, PR number/URL/author, lifecycle timestamps, API-reported base/head/merge SHAs, and collection start/end timestamps |
| `title`, `body` | Original PR title and description, without rewriting |
| `files` | Every changed-file entry returned by the API, with path, status, blob SHA, and patch; renamed files also retain the previous path |
| `comments` | Conversation comments, inline review comments, and submitted reviews, including empty-body review decisions |
| `labels` | All label names on the source PR at collection time, as a multi-value array |

There are no model questions, groups, dataset partitions, synthetic flags, or
additional source-context fields. A future training exporter can translate the
domain records into a model-specific format.

### Patches

The snapshot contains **72 changed-file entries**: 45 have the exact text patch
returned by GitHub, and 27 are PNG files for which the API supplies no patch.
Those image entries remain present with `patch: null`; no textual patch is
invented, and image bytes are not embedded.

These are API patch representations, not a claim that the dataset contains
every byte of each changed file. Source revisions and blob SHAs are retained
for traceability. The API-reported base/head references are a collection-time
snapshot; they are not a separately reconstructed patch merge base.

### Comments and reviews

All pages were fetched for the PR files, issue comments, inline review comments,
and reviews endpoints. The dataset contains:

- 45 `issue_comment` records: PR conversation comments.
- 24 `review_comment` records: inline code comments, including reply references.
- 52 `review` records: submitted review decisions and their summary bodies.

Each entry retains its ID (prefixed by kind to avoid collisions), source URL,
author login, timestamp, original body, reply reference, and location where
applicable. Deleted authors would be represented by `author: null`, not an
invented user. Review summaries retain `review_state` and `commit_sha`.
GitHub's reviews endpoint does not supply an update timestamp, so those
records have `updated_at: null`.

Inline locations retain the path, current and original line positions, side,
multi-line start positions, subject type, and current/original commit SHAs.
`LEFT` and `RIGHT` follow GitHub's patch-side convention. Current line positions
can be null on outdated comments; their original positions remain available.
Conversation comments and review summaries have `location: null`.

Comments are ordered by their creation/submission timestamp, then ID. Human and
bot comments are both preserved. Empty review bodies are not discarded because
their approval or changes-requested state still matters.

## What the labels mean

These are **observed source labels**, not newly inferred classifications or
verified training targets. The selected PRs currently carry workflow labels
such as `integration-tests: passing`, `integration-tests: failing`,
`integration-tests: skipped manually`, `skip changeset`, and `staff`.

The dataset does not translate them into invented `bug`, `accessibility`, or
`performance` answers. Multiple and potentially historical/conflicting labels
are preserved as returned. An absent label is not proof of a reviewed negative.

The records include discussions and review decisions from the full collected
history, and label membership as observed at collection time. They are **not**
a point-in-time reconstruction of what was known when a PR was opened.
Before training an opening-time auto-labeler, define a prediction cutoff,
exclude later comments and status outcomes, and decide which labels are
appropriate targets. Never include expected labels in model input evidence.

## Included pull requests

These ten merged PRs were selected manually for varied component and
documentation changes, with manageable file counts. This is a convenience
sample, not a random or balanced corpus.

| Source PR | Title |
| --- | --- |
| [primer/react#8477](https://github.com/primer/react/pull/8477) | Implementing small size variant for ActionList.Item |
| [primer/react#8484](https://github.com/primer/react/pull/8484) | fix: TreeView navigation when nested controls have focus |
| [primer/react#8412](https://github.com/primer/react/pull/8412) | fix(Button): apply disabled styles while loading under FF |
| [primer/react#8473](https://github.com/primer/react/pull/8473) | Upgrade @primer/live-region-element to 0.8.1 |
| [primer/react#8463](https://github.com/primer/react/pull/8463) | Truncate: infer props from `as` and forward `inline` to custom `as` components |
| [primer/react#8442](https://github.com/primer/react/pull/8442) | Allow SegmentedControl labels to wrap without overflow |
| [primer/react#8431](https://github.com/primer/react/pull/8431) | docs: update rollout strategy section |
| [primer/react#8434](https://github.com/primer/react/pull/8434) | AvatarStack: Prevent a thin edge from a masked avatar over the next avatar |
| [primer/react#8429](https://github.com/primer/react/pull/8429) | Prevent virtualized SelectPanel measurement warnings |
| [primer/react#8443](https://github.com/primer/react/pull/8443) | UnderlineNav: Avoid default # href for custom links |

PR descriptions, comments, paths, patches, authors, and labels are copied from
those public API responses, not fabricated or paraphrased. Original authors
and links provide attribution. Repository license terms still apply to source
code excerpts; collection does not assert new ownership over contributors'
discussion content.

## Inspect a sample

```sh
python3 -c 'import json; print(json.dumps(json.loads(open("labeling-data/samples.jsonl").readline()), indent=2))'
```

Validation checked the exact record count, unique source/comment identities,
complete fetched file/comment counts, label-catalog references, reply targets,
timestamps and revision formats, and equality of copied content against the
captured API responses. This does not certify training suitability, successful
model execution, or a rendered data-diff experience.
