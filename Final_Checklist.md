# Final Checklist: Assignment 1


---

## A. Repository (GitHub)

- [ ] The repository is public (or accessible to the graders) and the URL works when opened in a private browser window.
- [ ] `Assignment1.ipynb` is the final version, run once from top to bottom, with all outputs visible.
- [ ] `README.md` says what the project is and points to `rquirements.md`.
- [ ] `rquirements.md` explains how to: train the tokenizers, train the models, evaluate saved models, reproduce the main tables and figures.
- [ ] `outputs_part1/` contains `bpe_2k.json`, `bpe_10k.json`, `tokenizer_stats_valid.csv`, `report_table.csv`, `chars_per_token1.png`, `total_tokens.png`.
- [ ] `outputs_part2/` contains the three models `char_best.pt`, `bpe_2k_best.pt`, `bpe_10k_best.pt`, the three `*_history.csv` files, `config.json`, `training_policy_table.csv`, `parameter_counts.csv`, `test_results.csv`, `evidence_table.csv`, `vocab_allocation.csv`, `vocab_coverage.csv`, `scored_sentences.csv`, `learning_curves.png`, `length_vs_bpc.png`.
- [ ] `char_vocab.json` and readable vocabulary files for the BPE tokenizers are saved in `outputs_part1/`.

## B. Part 1: the tokenizers (7 points)

- [✅] Three conditions exist: character-level, BPE small (2,000), BPE large (10,000).
- [✅] The vocabulary sizes are explained (report, Section 2.2).
- [✅] Both BPE tokenizers are trained on the combined en/tr/zh training data, with one shared vocabulary (`tokenizer/balanced.txt`).
- [✅] The character tokenizer is built from training data only; unseen characters are handled with `<unk>` and the `<unk>` rate is reported.
- [✅] Predictions are written (four questions, with reasoning) and were not edited after seeing the results.
- [✅] The validation-set table has, for each tokenizer and language: vocabulary size, total tokens, tokens per sentence, characters per token.
- [✅] Example sentences from each language under all three tokenizers are shown (report, Appendix A).
- [✅] The report explains **why** the units look the way they do (training-data statistics), not only what they look like.

## C. Part 2: the language models (11 points)

- [ ✅] One decoder-only Transformer per tokenizer, trained from scratch (no pretrained models).
- [ ✅] The architecture and hyperparameters are identical across the three models.
- [ ✅] The training-control policy is stated with the reason **and one limitation** (10 epochs each; unequal updates and compute).
- [ ✅] Parameter counts are reported for each model: total, input embeddings, output layer.
- [ ✅] Training and validation loss are recorded and the learning curves are plotted.
- [ ✅] The report gives: architecture, training-control policy, important hyperparameters, problems or failed choices, changes and why.
- [ ✅] Any subsampling is described (none: all training data were used).
- [ ✅] Representative trained models are saved (`*_best.pt`).

## D. Part 3: evaluation and analysis (12 points)

- [ ✅] Each model is evaluated separately on English, Turkish and Chinese **test** data.
- [ ✅] The measure is bits per character, and the report explains why it is comparable across tokenizers.
- [ ✅] Results are reported overall **and** per language.
- [ ✅] At least two original hypotheses/questions are revisited, each with: what I expected, the final results, supported or not, a possible explanation.
- [ ✅] At least one of them connects tokenization behaviour to language-model behaviour.
- [ ✅] "Who gets the vocabulary?": the method for assigning tokens to languages (with the threshold) is described.
- [ ✅] That section answers a question from the checkpoint, with statistics and a few examples.
- [ ✅] "Relating tokenization to model behaviour": one pattern is investigated with at least two kinds of evidence, and the report says what changed my mind.
- [ ✅] Error analysis: several test sentences with their tokenizations and model scores.
- [ ✅] The report separates what the experiment **shows** from what I **interpret**.
- [ ✅] Conclusion: one supported expectation, one unsupported/surprising result, what the experiment says about a shared tokenizer, which approach I would choose and why.
---

## Status summary

| Section | Status |
|---|---|
| A. Repository |  ✅ |
| B. Part 1 |  ✅ |
| C. Part 2 |  ✅ |
| D. Part 3 |  ✅ |
| E. Report |  ✅ |