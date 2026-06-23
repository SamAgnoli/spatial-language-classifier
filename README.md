# Spatial Language Classifier (DeBERTa-v3)

Detect **spatial language at the word level**: given an utterance and a target word, the
model decides whether that word is being used **spatially** (`1`) or **not** (`0`) *in
utternace context*. At inference it scores every dictionary-candidate word in an utterance, so you get
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
   fine-tune your own (Google Collab's T4 GPU recommended).
3. It reads `data/example/{train,val,test}.csv` by default — swap in your own data in the
   same schema.

## Bring your own data

The CSV format the notebooks expect is documented in
[`data/example/SCHEMA.md`](data/example/SCHEMA.md), with a runnable synthetic example in
[`data/example/`](data/example/). In short:

- **Applying the model** → you only need an `utterance` column.
- **Training / evaluating** → you need `utterance, coded_word, spatial_or_not, session_id,
  line, is_candidate`, split into `train/val/test.csv` (grouped by `session_id` so a speaker
  never spans splits).

## How it works

1. **Dictionary gate.** A spatial wordlist (`data/spatial dictionary.txt`, parsed into exact +
   prefix matchers in `data/spatial_matcher.json`) selects candidate words. Non-candidates are
   non-spatial by definition and never sent to the model.
2. **Contextual classification.** Each candidate word is scored by DeBERTa-v3 *with its
   utterance* as context.
3. **Calibrated confidence.** Raw probabilities are over-confident, so a temperature
   (`data/calibration.json`, `T = 1.816`) rescales them — the hard 0/1 decision is unchanged,
   but the confidence can be read literally.

## Requirements

See [`requirements.txt`](requirements.txt). Colab already has most of these. The DeBERTa-v3
tokenizer is loaded from `tokenizer.json` (fast), so no SentencePiece build is required.

## Data & privacy

The data shipped here is **synthetic** — fabricated utterances that only demonstrate the
format. The real parent–child research transcripts the model was trained on are **not**
included. Treat any human-subjects data you bring accordingly.

## Citation

If you use this model or pipeline, please contact Sam Agnoli (SamAgnoli[at]u.northwestern.edu)

## License

[MIT](LICENSE).
