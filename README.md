# Multilingual-Tokenization-and-Language-Modeling

# How to use this repository

This file answers four questions: how to **train the tokenizers**, **train the models**, **evaluate saved models**, and **reproduce the main tables and figures**. It also says where the trained models are stored.

Everything is in one notebook, `Assignment1.ipynb`. Part 1 is the tokenizers, Part 2 is the language models, and the analysis for the report follows Part 2. The results are written to two folders:

```
Assignment1.ipynb
outputs_part1/      tokenizers, tokenizer tables, Figures 1 and 2
outputs_part2/      trained models, training logs, result tables, Figures 3 and 4
```

The data are **not** part of the repository. They are read from `/srv/data/lt2326-h26/a1/` on `mltgpu` / `mltgpu-2`.

## 0. Setup

**Environment** (what I used): Python 3, `torch` 2.4.1 (CUDA 12.1), `tokenizers` (Hugging Face), `numpy`, `pandas`, `matplotlib`, `jupyter`. Only `torch` is a tested version; the others are not pinned.

```
pip install --user tokenizers pandas matplotlib
```
**I trained on a GTX 1080 Ti.** 
```
## 1. Train the tokenizers

**Notebook section:** Part 1, sections 1.1 to 1.3.

1. Run the setup and data-loading cells (1.1).
2. Section 1.2 builds the **character tokenizer** from the training text only. Characters never seen in training are mapped to `<unk>`.
3. Section 1.3 trains the two **byte-level BPE tokenizers** (2,000 and 10,000 tokens) on `tokenizer/balanced.txt` with the Hugging Face `tokenizers` library.

**Output:** `outputs_part1/bpe_2k.json` and `outputs_part1/bpe_10k.json`.

If these two files already exist, the notebook **loads them instead of training again**. This matters: the language models only work with exactly the same token ids as in training. To really retrain the tokenizers, delete the two JSON files first. (Then the models must be retrained as well.)

Training the tokenizers takes well under a minute.

## 2. Train the models

**Notebook section:** Part 2.

1. Run the Part 2 settings cell. The main settings are:

   | Setting | Value |
   |---|---|
   | Context / hidden / feed-forward | 256 / 256 / 1024 |
   | Layers / heads / dropout | 2 / 4 / 0.1 |
   | Batch | 64 windows of 256 tokens |
   | Optimiser | AdamW, learning rate 1e-3, 200 warm-up steps, cosine decay |
   | Epochs | **10** (same for every model) |
   | Seed | 2326 |
   | `OUT_DIR` | `"outputs_part2"` |

   The settings are also written to `outputs_part2/config.json`.

2. Run the cells that build the training data (shuffled sentences, token streams), the model classes (`Block`, `TransformerLM`), the parameter-count cell and the helper functions.
3. Run the **three training cells, one per tokenizer**:

```python
   train_model("char")
   train_model("bpe_2k")
   train_model("bpe_10k")
```

   On a GTX 1080 Ti they took about 9.3, 4.4 and 4.8 minutes.

**Output (in `outputs_part2/`):**

| File | Content |
|---|---|
| `char_best.pt`, `bpe_2k_best.pt`, `bpe_10k_best.pt` | trained models (the checkpoint with the best validation loss) |
| `char_history.csv`, `bpe_2k_history.csv`, `bpe_10k_history.csv` | training and validation loss and validation BPC, every half epoch |
| `config.json` | the hyperparameters |
| `training_policy_table.csv` | tokens, steps per epoch and total steps per model |
| `parameter_counts.csv` | total, input-embedding and output-layer parameters per model |

**Warning:** running a training cell **overwrites** that model's `*_best.pt` and `*_history.csv`.

GPU training is not bit-for-bit deterministic. I have not tested it, so small differences in the last digits are possible if you retrain.

## 3. Evaluate the saved models (no retraining)

The test evaluation uses the saved checkpoints `outputs_part2/<name>_best.pt`.

1. Run **all cells from the top down to the end of section 2.1** (setup, data, tokenizers, model classes, helper functions). This rebuilds everything the evaluation needs. The BPE tokenizers are loaded from `outputs_part1/*.json`.
2. **Skip the three `train_model(...)` calls.**
3. Run the cells after the learning-curve section: loading the test data, the `total_nats` function, and the evaluation cell that fills `test_results`.

**Output:** `outputs_part2/test_results.csv`, with test bits per character (BPC) per language and overall.

The evaluation reports **bits per character**: the total negative log2-likelihood of the test text divided by its number of characters. This makes models with different tokenizers comparable. The test set is used only in this step.

## 4. Reproduce the main tables and figures

Run the notebook from top to bottom (or, with saved models, as in section 3). Every table and figure of the report comes from a specific cell:

| Report item | Notebook section | Output file |
|---|---|---|
| Table: tokenizer statistics (validation set) | 1.4 | `outputs_part1/tokenizer_stats_valid.csv`, `report_table.csv` |
| Figure 1: characters per token | 1.5 | `outputs_part1/chars_per_token1.png` |
| Figure 2: total tokens | 1.6 | `outputs_part1/total_tokens.png` |
| Examples, byte fragments, tokens per word, Chinese vocabulary | 1.5, 1.6 | printed in the notebook |
| Table: training amount per model | Part 2 (data cells) | `outputs_part2/training_policy_table.csv` |
| Table: parameter counts | 2.1 | `outputs_part2/parameter_counts.csv` |
| Figure 3: learning curves | 2.2 | `outputs_part2/learning_curves.png` |
| Table: test BPC, overall and per language | cells after 2.2 | `outputs_part2/test_results.csv` |
| Table and Figure 4: sequence length vs. BPC | 2.3 | `outputs_part2/evidence_table.csv`, `length_vs_bpc.png` |
| Table: vocabulary allocation | 2.4 | `outputs_part2/vocab_allocation.csv` |
| Table: vocabulary coverage, examples of tokens | 2.5 | `outputs_part2/vocab_coverage.csv` |
| Error analysis (sentence scores) | 2.6 | `outputs_part2/scored_sentences.csv` |

The error analysis samples 500 test sentences per language with a fixed seed (2326), so the same sentences are chosen every time.

## 5. Trained models: size and location

The saved models contain only the weights (no optimiser state), so their size follows from the parameter counts (4 bytes per parameter):

| Model | Parameters | File size (approx.) |
|---|---|---|
| `char_best.pt` | 6,480,896 | 26 MB |
| `bpe_2k_best.pt` | 2,670,080 | 11 MB |
| `bpe_10k_best.pt` | 6,766,080 | 27 MB |

Together about 64 MB. Each file is well below GitHub's 100 MB limit for single files, so **I included the three models in the repository**. The BPE tokenizers (`bpe_2k.json`, `bpe_10k.json`) are small and are included as well, because the models need exactly these tokenizers.
