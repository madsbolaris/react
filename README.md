---
configs:
- config_name: default
  data_files:
  - split: examples
    path: labeling-data/samples.jsonl
dataset_info:
  features:
  - name: id
    dtype: string
  - name: source
    struct:
    - name: repository
      dtype: string
    - name: number
      dtype: int64
    - name: url
      dtype: string
    - name: author
      struct:
      - name: login
        dtype: string
    - name: created_at
      dtype: string
    - name: updated_at
      dtype: string
    - name: merged_at
      dtype: string
    - name: base_sha
      dtype: string
    - name: head_sha
      dtype: string
    - name: merge_commit_sha
      dtype: string
    - name: collection_started_at
      dtype: string
    - name: collection_finished_at
      dtype: string
  - name: title
    dtype: string
  - name: body
    dtype: string
  - name: files
    list:
    - name: path
      dtype: string
    - name: status
      dtype: string
    - name: blob_sha
      dtype: string
    - name: patch
      dtype: string
    - name: previous_path
      dtype: string
  - name: comments
    list:
    - name: id
      dtype: string
    - name: kind
      dtype:
        class_label:
          names:
            '0': issue_comment
            '1': review_comment
            '2': review
    - name: url
      dtype: string
    - name: author
      struct:
      - name: login
        dtype: string
    - name: created_at
      dtype: string
    - name: updated_at
      dtype: string
    - name: body
      dtype: string
    - name: in_reply_to_id
      dtype: string
    - name: location
      struct:
      - name: path
        dtype: string
      - name: line
        dtype: int64
      - name: side
        dtype: string
      - name: start_line
        dtype: int64
      - name: start_side
        dtype: string
      - name: original_line
        dtype: int64
      - name: original_start_line
        dtype: int64
      - name: subject_type
        dtype: string
      - name: commit_sha
        dtype: string
      - name: original_commit_sha
        dtype: string
    - name: review_state
      dtype: string
    - name: commit_sha
      dtype: string
  - name: labels
    list:
      class_label:
        names:
          '0': 'integration-tests: failing'
          '1': 'integration-tests: passing'
          '2': 'integration-tests: recommended'
          '3': 'integration-tests: skipped manually'
          '4': skip changeset
          '5': staff
---

# Primer React PR labeling examples

This data-only branch contains ten real public Primer React pull requests.
The YAML above uses **Hugging Face Datasets dataset-card metadata**, not a custom
schema or manifest. It explicitly declares the JSONL file and its complete
nested features. See [dataset documentation](labeling-data/README.md) for source
attribution, collection details, and limitations.

## Feature semantics

- `id` is the stable source PR identity.
- `labels` is a `List(ClassLabel)`, allowing multiple observed labels per PR.
  The declared names come from [the label catalog](labeling-data/labels.json);
  their numeric IDs are Hugging Face category indices, not GitHub label IDs.
- `comments.kind` is a `ClassLabel` for the three collected API record kinds.
- `source` is a struct; `files` and `comments` are lists of structs.
- `Value("string")` fields remain strings, not inferred enums. Nullable and
  missing fields are represented as null when loaded into the typed dataset.
- `examples` is a Hugging Face loading split name for this single inspection
  collection, **not** a train/test partition or a new field in the records.

The original JSONL remains unchanged, including its human-readable labels,
patches, comments, nulls, and provenance. Hugging Face's in-memory representation
encodes ClassLabels as category indices; use `int2str` to recover their names.
The current labels include workflow/status outcomes and must not be treated as
leakage-safe opening-time training targets.

## Load locally

After checking out this branch, using `datasets==4.5.0`:

```python
from datasets import load_dataset

dataset = load_dataset(".")["examples"]
print(dataset.features)
names = dataset.features["labels"].feature.names
print([names[index] for index in dataset[0]["labels"]])
```

This loads the standard metadata directly from this root dataset card; no custom
loader script or model-specific questions are required.
