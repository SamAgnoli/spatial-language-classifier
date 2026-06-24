# Spatial Language Classifier (DeBERTa-v3)

Detect **spatial language at the word level**: given an utterance and a target word, the
model decides whether that word is being used **spatially** (`1`) or **not** (`0`) *in
utterance context*. At inference it scores every dictionary-candidate word in an utterance, so you get
per-word spatial labels with a calibrated confidence for each.

**Model:** [`SamAgnoli/deberta-v3-base-spatial-language-detection`](https://huggingface.co/SamAgnoli/deberta-v3-base-spatial-language-detection)
· fine-tuned from [`microsoft/deberta-v3-base`](https://huggingface.co/microsoft/deberta-v3-base).

> Input is a **sentence pair** — `(utterance, target_word)` — not a whole sentence. The same
> word can be spatial in one utterance ("go *up* the ramp") and not in another ("what's *up*?").

## Notebooks

| Notebook | What it does | Open |
|---|---|---|
| `notebooks/03_quickstart_apply.ipynb` | **Start here.** Load the published model and label spatial words in your own text. No training. | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SamAgnoli/spatial-language-classifier/blob/main/notebooks/03_quickstart_apply.ipynb) |
| `notebooks/02_train_and_analyze.ipynb` | Load (or fine-tune) the model, then run held-out evaluation, confidence calibration, inference, and clinical-style metrics. | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SamAgnoli/spatial-language-classifier/blob/main/notebooks/02_train_and_analyze.ipynb) |

> Replace `SamAgnoli/spatial-language-classifier` in the badge URLs if you fork this under a
> different repo name.

## Quickstart

### Just classify your text (no training)
1. Open `03_quickstart_apply.ipynb` (Colab badge above, or locally).
2. Put your sentences in a CSV with an **`utterance`** column (see `data/example/`).
3. Point `INPUT_CSV` at it and run all cells → per-word predictions land in `predictions.csv`.

### Train or evaluate
1. Open `02_train_and_analyze.ipynb`.
2. Keep `train_from_scratch = False` to evaluate the **published** model, or set it `True` to
   fine-tune your own (Google Colab's T4 GPU recommended).
3. It reads `data/example/{train,val,test}.csv` by default — swap in your own data in the
   same schema.

## Bring your own data

The CSV format the notebooks expect is documented in
[`data/example/SCHEMA.md`](data/example/SCHEMA.md), with a runnable example from Zhou et al., (in prep.) in
[`data/example/`](data/example/). In short:

- **Applying the model** → you only need an `utterance` column.
- **Training / evaluating** → you need `utterance, coded_word, spatial_or_not, session_id,
  line, is_candidate`, split into `train/val/test.csv` (grouped by `session_id` so a speaker
  never spans splits).

## How it works

1. **Dictionary gate.** A spatial wordlist from Cannon et al., 2007 and extended by Zhou et al., (in prep.)
   (`data/spatial dictionary.txt`, parsed into exact + prefix matchers in `data/spatial_matcher.json`)
   flags candidate words. **At inference** (`classify_utterance` / notebook 03), only candidates are
   scored — non-candidates are taken as non-spatial and skipped, never reaching the model. *(During
   training and evaluation, every word is still passed through the model, non-candidates labeled `0`;
   that's why the eval reports both a "candidates-only" headline view and an "overall (every word)" view.)*
2. **Contextual classification.** Each candidate word is scored by an already fine-tuned DeBERTa-v3
   (using coding data from Zhou et al., in prep.) *with its utterance* as context.
3. **Calibrated confidence.** Raw probabilities are over-confident, so a temperature
   (`data/calibration.json`, `T = 1.816`) rescales them — the hard 0/1 decision is unchanged,
   but the confidence can be read literally.

## Requirements

See [`requirements.txt`](requirements.txt). Colab already has most of these. Notebook 02
builds the base `microsoft/deberta-v3-base` tokenizer, which needs `sentencepiece` +
`protobuf`; notebook 03 loads the published model's fast `tokenizer.json` and needs neither.

## Data & privacy

The example data in `data/example/` is **deidentified** data from Zhou et al. (in prep.).
Please cite the source if you use it.
<!-- TODO: add your consent/IRB statement and a dataset-access contact here if appropriate. -->

## Citation

If you use this model or pipeline, please contact Sam Agnoli (SamAgnoli[at]u.northwestern.edu)

## License

- **Code** (notebooks, scripts): [MIT](LICENSE). -> FIX to CC BY 4.0
- **Data** (`data/`, including the example datasets and spatial dictionary): [CC BY 4.0](data/LICENSE) — free to reuse, including commercially, but you **must give credit** (see the attribution note in [`data/LICENSE`](data/LICENSE)).
