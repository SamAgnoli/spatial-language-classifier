# Example data schema

Word-level spatial-language data: **one row per word occurrence**, each row pairing a target
word with the utterance it appeared in.

## Columns

| Column | Description |
|---|---|
| `utterance` | The full utterance — the context the word is judged in. |
| `coded_word` | Target word being classified within the utterance context (lowercased, punctuation stripped). |
| `spatial_or_not` | Label: **1 = spatial**, **0 = not spatial**. Internally renamed to `labels`. |
| `session_id` | Speaker/session group id. Used for the leakage-free group split — all rows of a session stay in one split. |
| `line` | Line number within the session (bookkeeping / joins). |
| `is_candidate` | Whether this exact token was selected by the bag-of-words dictionary pass and shown to a human coder. Lets evaluation separate "dictionary candidates only" (the meaningful view) from "every word". |

### Note on `is_candidate`

`is_candidate` marks the tokens that carry a **real human label**. Rows with
`is_candidate = False` were never presented to a coder and are labelled `0` by construction —
the dictionary already ruled them out as plausibly spatial. Report the candidates-only view when
judging model quality; the overall view is padded with trivially non-spatial tokens.

This is the `in_dictionary` flag from the evaluation pipeline. It is derived from what human
coders actually saw, *not* from re-running the dictionary regex — so it is reproducible against
the human coding rather than sensitive to matcher edge cases. At **inference** on new text there
is no human coding, so the dictionary matcher in `data/spatial_matcher.json` performs the
equivalent gating role.

## Splits

Grouped by `session_id` (70/15/15, seed 42) so no conversation spans two splits.

| File | Rows | Sessions | Candidates | Spatial |
|---|---:|---:|---:|---:|
| `train.csv` | 23,100 | 106 | 2,398 | 995 |
| `val.csv` | 5,714 | 23 | 592 | 249 |
| `test.csv` | 4,470 | 23 | 526 | 211 |
| **total** | **33,284** | **152** | **3,516** | **1,455** |

## Usage requirements

- **Quick classification only:** requires the `utterance` column.
- **Training / evaluation:** requires the complete schema above, split into
  `train` / `val` / `test` grouped by `session_id`.

## Source

Deidentified parent–child conversational reflections recorded at a museum tinkering exhibit
(Polinsky et al., 2023). Speaker names and unintelligible speech appear as placeholders
(`M_name`, `xxx`). Cite the source if you reuse this data (CC BY 4.0).
