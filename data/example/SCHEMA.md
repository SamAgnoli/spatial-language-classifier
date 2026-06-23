# Dataset schema

`train.csv`, `val.csv`, and `test.csv` all share the same columns — one row **per word**
that the model scores in the context of its utterance. This is the output of the (private)
`01_build_dataset` step. To use your own data, produce CSVs with these columns.

| Column | Type | Required for... | Description |
|---|---|---|---|
| `utterance` | string | **train + apply** | The full utterance — the context the word is judged in. |
| `coded_word` | string | train | The target word being classified within `utterance`. |
| `spatial_or_not` | int (0/1) | train | Label: **1 = spatial**, **0 = not spatial**. Internally renamed to `labels`. |
| `session_id` | string | train | Speaker/session group id. Used for the leakage-free group split — all rows of a session stay in one split. |
| `line` | int | train | Line number within the session (bookkeeping / joins). |
| `is_candidate` | bool | train | Whether `coded_word` matched the spatial dictionary. Lets evaluation separate "dictionary candidates only" (the meaningful view) from "every word". |

## Two ways to bring your own data

- **Just classify text (notebook `03_quickstart_apply`)** — you only need an **`utterance`**
  column. The notebook finds candidate words with the dictionary matcher and labels each one.
  No `coded_word`/`spatial_or_not` needed.

- **Train or evaluate (notebook `02_train_and_analyze`)** — you need the full schema above,
  split into `train.csv` / `val.csv` / `test.csv`. Produce these with your own equivalent of
  step 1 (group your rows by `session_id` so the same speaker never spans splits).

## Note

The files in this folder are **synthetic** — generated from fabricated utterances purely to
demonstrate the format and let the notebooks run end-to-end. They are not real research data.
