# Final Checkpoint: Assignment 1

## 1. BPE vocabulary sizes and why ✅

I trained two byte-level BPE tokenizers (Hugging Face `tokenizers`) on the shared multilingual `balanced.txt` training data, plus a character-level tokenizer built from the training text only.

| Tokenizer | Vocabulary size | Vocabulary of the model (+1 end-of-sentence token) |
|---|---|---|
| Character-level | 9,443 (+ `<unk>` for unseen characters) | 9,444 |
| BPE-small | 2,000 | 2,001 |
| BPE-large | 10,000 | 10,001 |

Why these sizes:

* 2,000 is small: 256 byte symbols plus about 1,750 merges shared by three languages. It stays close to the raw characters, so I expected it to work well for English and Turkish but to struggle with Chinese, which has thousands of different characters.
* 10,000 is 5x larger, enough for many whole words and common Chinese characters, so the difference between the two BPE tokenizers should be visible.
* The embedding table stays small compared with large models, which suits a small Transformer trained on about 9M characters.
* I used byte-level BPE so that no text is ever unknown. A normal character-based BPE with only 2,000 tokens could not even contain all Chinese characters.

**Did the choice hold up?** Yes. The difference between the two sizes is large: BPE-2k leaves Chinese worse than characters (6.575 vs 6.245 bits per character), while BPE-10k is best in every language (see Sections 2 and 6).

## 2. Initial hypotheses (written before running anything) and what the final results showed

The hypotheses are unchanged from the checkpoint. **Additional hypothesis about LM performance (written before training): none was written.**

| # | What I expected | What the final results showed | Verdict |
|---|---|---|---|
| 1 | Chinese needs the most tokens, English the fewest, Turkish in between (a Chinese character is 3 bytes). | Chinese needs by far the most (410k and 295k tokens vs about 164k to 169k and 114k to 115k). English needs the fewest at 2k (163.8k vs 168.8k for Turkish), but at 10k Turkish needs very slightly fewer (114.3k vs 114.7k). | ✅ Mostly supported (Turkish is not "in between" at 10k) |
| 2 | A larger vocabulary gives fewer tokens and more characters per token for every language, with the biggest improvement for Chinese. | Tokens fall by 30.0% (en), 32.3% (tr), 28.2% (zh). From 2k to 10k, Chinese BPC improves most (−0.389, −5.9%, against −0.082 for English and −0.074 for Turkish). But in compression Chinese improves least in relative terms (characters per token +39% vs +43% and +48%), and relative to the character model Chinese gains almost nothing (−0.059). | ⚠️ Partly supported |
| 3 | English learns frequent whole words and endings. I had no prediction for Turkish and Chinese. | English: `▁the ▁of ▁in ▁and ▁to ing ▁was ion`. Turkish: the letters `ı ü ç ğ`, the apostrophe, suffix pieces (`ler`, `lar`, `ın`) and function words (`▁ve`, `▁bir`). Chinese: single frequent characters and punctuation at 2k, common two-character words at 10k (列車, 服務, 取代, 大幅, 中华人民共和国). | ✅ Supported for English; explored for the other two |
| 4 | BPE-2k best (leaning), the character model slowest and probably worst, least sure about BPE-10k. | BPE-10k is best in every language. BPE-2k only ties with characters overall (3.593 vs 3.589). The character model was slowest (559 s and 5,480 updates, against 262 s and 285 s). It is the worst only for English and Turkish; for Chinese BPE-2k is worst. | ❌ Best tokenizer not supported; ✅ "slowest" supported; ⚠️ "worst" only partly |

What I expected vs. what Part 1 showed (the checkpoint notes, kept):

* Chinese needs the most tokens: confirmed.
* English needs the fewest: true at 2k, essentially the same as Turkish at 10k.
* Biggest improvement for Chinese: only partly right in Part 1 (largest absolute saving of tokens, −115.6k, but the smallest relative one, −28.2% vs −30.0% and −32.3%); the final BPC results confirm the same mixed picture.

**What changed my mind about prediction 4:** I expected rare tokens in a 10,000 vocabulary to be too poorly trained on about 9M characters. A possible explanation is that each prediction step of BPE-10k covers much more text, and a 256-token window holds about 800 English characters instead of 256. It is not just more computation: BPE-10k made only 2,620 updates against 5,480 for the character model and has almost the same number of parameters (6.77M vs 6.48M).

## 3. Parameter counts of the three models ✅ (answered)

Same architecture for all three: 2 layers, d_model 256, 4 heads, d_ff 1024, dropout 0.1, context 256 tokens. Only the vocabulary differs.

| Model | Vocab (+EOS) | Input embeddings | Output layer | Position emb. | Transformer layers | **Total** |
|---|---|---|---|---|---|---|
| char | 9,444 | 2,417,664 | 2,417,664 | 65,536 | 1,580,032 | **6,480,896** |
| BPE-2k | 2,001 | 512,256 | 512,256 | 65,536 | 1,580,032 | **2,670,080** |
| BPE-10k | 10,001 | 2,560,256 | 2,560,256 | 65,536 | 1,580,032 | **6,766,080** |

The Transformer layers are identical. About 75% (char), 38% (BPE-2k) and 76% (BPE-10k) of all parameters are in the input and output vocabulary matrices, so BPE-2k is also 59% smaller than the other two. I take this into account when interpreting the results.

## 4. One sentence per language under all three tokenizers ✅

`|` separates tokens; `▁` marks a space; `�` is a fragment of a multi-byte character (byte-level BPE split one character into bytes). I replaced the Chinese sentence of the first checkpoint with a cleaner one (no stray English word).

**English:** He was First Deputy Chairman of Ways and Means from 2010 to 2013.

* Character (65 tokens): one token per character
* BPE-2k (28 tokens): He | ▁was | ▁F | ir | st | ▁D | ep | ut | y | ▁Ch | a | ir | man | ▁of | ▁W | ay | s | ▁and | ▁M | e | ans | ▁from | ▁20 | 10 | ▁to | ▁20 | 13 | .
* BPE-10k (21 tokens): He | ▁was | ▁F | irst | ▁Dep | ut | y | ▁Ch | air | man | ▁of | ▁W | ays | ▁and | ▁Me | ans | ▁from | ▁2010 | ▁to | ▁2013 | .

**Turkish:** Selim'in hükümranlığı sırasında önemli bir değişiklik daha olmuştur.

* Character (68 tokens): one token per character
* BPE-2k (25 tokens): S | el | im | ' | in | ▁h | ük | üm | ran | lı | ğı | ▁sır | asında | ▁ön | em | li | ▁bir | ▁değ | iş | ik | lik | ▁daha | ▁olm | uştur | .
* BPE-10k (15 tokens): S | elim | ' | in | ▁hüküm | ran | lığı | ▁sırasında | ▁önemli | ▁bir | ▁değişik | lik | ▁daha | ▁olmuştur | .

**Chinese:** 列車運用方面，初期服務以10輛編組為主，完全取代103系後才大幅增加以15輛編組服務的班次。

* Character (46 tokens): one token per character
* BPE-2k (48 tokens): 列 | 車 | 運 | 用 | 方 | 面 | ， | 初 | 期 | 服 | 務 | 以 | 10 | � | � | � | � | 組 | 為 | 主 | ， | 完 | 全 | 取 | 代 | 10 | 3 | 系 | 後 | 才 | 大 | � | � | 增 | 加 | 以 | 15 | � | � | � | � | 組 | 服 | 務 | 的 | 班 | 次 | 。
* BPE-10k (33 tokens): 列車 | 運 | 用 | 方面 | ， | 初期 | 服務 | 以 | 10 | 輛 | 編 | 組 | 為主 | ， | 完全 | 取代 | 10 | 3 | 系 | 後 | 才 | 大幅 | 增加 | 以 | 15 | 輛 | 編 | 組 | 服務 | 的 | 班 | 次 | 。

Short notes on these examples:

* English: frequent words (`▁was`, `▁of`, `▁and`) are single tokens at both sizes; the years `▁2010` and `▁2013` only become single tokens at 10k.
* Turkish: suffix-like pieces (`lik`, `ran`, `lığı`) are learned. At 10k whole words such as `▁sırasında`, `▁önemli` and `▁olmuştur` appear as single tokens.
* Chinese: BPE-2k needs more tokens (48) than characters (46) because rarer characters (輛, 編, 幅) break into `�` byte fragments. At 10k they become whole tokens, and two-character words such as 列車, 服務, 取代 appear.

## 5. Preliminary tokenizer statistics (validation set) ✅

![Average characters per token by tokenizer and language](outputs_part1/chars_per_token1.png)

*Figure 1: Average number of Unicode characters per token on the validation set. Chinese stays below 1.0 under BPE-2k because many characters are split into byte fragments.*

| Tokenizer | Language | Vocab size | Total tokens | Avg tokens / sentence | Avg chars / token |
|---|---|---|---|---|---|
| char | en | 9,443 | 368,269 | 81.12 | 1.000 |
| char | tr | 9,443 | 368,383 | 98.63 | 1.000 |
| char | zh | 9,443 | 368,266 | 38.55 | 1.000 |
| BPE-2k | en | 2,000 | 163,777 | 36.07 | 2.249 |
| BPE-2k | tr | 2,000 | 168,812 | 45.20 | 2.182 |
| BPE-2k | zh | 2,000 | 410,248 | 42.94 | 0.898 |
| BPE-10k | en | 10,000 | 114,709 | 25.27 | 3.210 |
| BPE-10k | tr | 10,000 | 114,292 | 30.60 | 3.223 |
| BPE-10k | zh | 10,000 | 294,656 | 30.84 | 1.250 |

![Total tokens needed to encode the validation set](outputs_part1/total_tokens.png)

*Figure 2: Total tokens per tokenizer and language. The Chinese bar under BPE-2k is taller than under the character tokenizer.*

Extra measurements:

* Character tokenizer `<unk>` rate on validation: en 0.0133%, tr 0.0041%, zh 0.0701%. BPE has 0% (byte-level).
* Share of tokens that are partial-byte fragments: BPE-2k: en 0.4%, tr 0.8%, zh 35.4%. BPE-10k: en 0.3%, tr 0.4%, zh 7.6%.
* Tokens per word: English 2.57 (2k) → 1.80 (10k); Turkish 3.47 (2k) → 2.35 (10k). Turkish words are longer (6.6 vs 4.9 characters).
* Chinese vocabulary content: BPE-2k has 662 single Chinese characters and 49 multi-character tokens; BPE-10k has 2,391 and 2,056.

## 6. Training / validation curves ✅ (answered)

![Learning curves](outputs_part2/learning_curves.png)

*Figure 3: training (dashed) and validation (solid) loss per token (left, not comparable across tokenizers) and validation bits per character (right).*

Policy: every model gets the same 10 epochs over the same training text.

| Model | Train tokens | Steps | Time | Final train loss | Final val loss | Final val BPC |
|---|---|---|---|---|---|---|
| char | 8,980,877 | 5,480 | 559 s | 2.510 | 2.490 | 3.593 |
| BPE-2k | 6,072,092 | 3,700 | 262 s | 3.736 | 3.678 | 3.595 |
| BPE-10k | 4,305,879 | 2,620 | 285 s | 4.778 | 4.913 | 3.418 |

What the curves show:

* The character model leads during the first two epochs (4.034 vs 4.041 for BPE-10k after epoch 2); BPE-10k overtakes it between epoch 2 and 2.5 (3.876 vs 3.940) and keeps its lead.
* Towards the end all curves flatten (about 0.004 BPC per half epoch), partly because the learning rate has decayed.
* BPE-10k ends with a slightly higher validation than training loss, so it may start to overfit, but its validation loss was still decreasing at the last evaluation.
* A short 2-epoch test run I made before the final run had the character model ahead (about 4.13, against 4.22 for BPE-10k and 4.27 for BPE-2k). The longer run reversed this, so early curves were misleading.
* Loss per token is not comparable across tokenizers, so the comparison uses bits per character.

**Final test results** (bits per character, lower is better):

| Model | en | tr | zh | overall |
|---|---|---|---|---|
| char | 2.226 | 2.260 | 6.245 | 3.589 |
| BPE-2k | 2.054 | 2.107 | 6.575 | 3.593 |
| BPE-10k | **1.972** | **2.033** | **6.186** | **3.410** |

## 7. An interesting observation so far ✅ and what the final results showed

Chinese under the small BPE is worse than plain characters. BPE-2k needs 410,248 tokens for Chinese, which is more than the 368,266 characters, and 35.4% of its Chinese tokens are byte fragments. The phrase 经济发展 is split as 经 | � | � | 发 | 展 by BPE-2k but as 经济 | 发展 by BPE-10k.

My explanation: Chinese characters take 3 bytes. With only about 1,750 merges shared among three languages, only the most frequent Chinese characters get merged into a single token. The rest stay as 2 to 3 byte pieces.

**The final results support this and carry it into the language models** (✅): Chinese BPC is 6.575 under BPE-2k, against 6.245 for characters and 6.186 for BPE-10k. Within every language, the order of the tokenizers by tokens per 100 characters equals their order by BPC (for Chinese: BPE-10k 80, char 100, BPE-2k 111). This is an association, not proof of cause: it is confounded with how much text fits into the 256-token window (230 Chinese characters for BPE-2k, 318 for BPE-10k, 256 for characters) and with model size.

Other observations from the checkpoint and what happened to them:

* Turkish *evlerimizden* → ev | ler | im | iz | den lines up with real suffixes, even though BPE knows nothing about morphology. ✅ Still true, but the units follow frequency, not linguistics: *kitaplar* is cut k | it | ap | lar.
* English and Turkish have almost identical characters per token at 10k (3.210 vs 3.223). **Follow-up:** Turkish is still worse in BPC (the gap is 0.034 for characters, 0.053 for BPE-2k, 0.061 for BPE-10k), so sequence length is not the whole story. The English data is Simple English Wikipedia, which may play a role; the experiment cannot separate this from Turkish morphology.
* The Chinese data mixes traditional and simplified characters (一个 and 一個). ⏳ I did not test this directly.
* New observation: in the shared vocabulary, Chinese receives 54% (2k) and 49% (10k) of the entries, but needs about twice as many different tokens as English or Turkish to cover 90% of its text.

## 8. A question I want the final experiments to answer ✅ (chosen after the checkpoint)

**Question:** Chinese needs more tokens than characters under BPE-2k. Does this extra cost also appear as worse language-model performance (BPC) for Chinese than under BPE-10k?

**Answer: yes.** Chinese BPC is 6.575 under BPE-2k and 6.186 under BPE-10k (0.389 bits per character, 5.9%, worse), and BPE-2k is also worse than the character model (6.245) for Chinese. In the same models, English and Turkish are better under BPE-2k than under characters, so the effect is specific to Chinese. The error analysis agrees: for sentences with rare Chinese characters BPE-2k is up to 3.9 bits per character worse than the character model because most of those characters are split into byte pieces.

**Caution:** the experiment shows the association. It does not show which factor (more tokens per character, byte fragments or the shorter effective context) causes it and every model was trained once (one seed).

---

## Overall: what was supported and what was not

* ✅ **Supported:** Chinese needs the most tokens; a larger BPE vocabulary compresses all languages more and gives better language models; the small BPE hurts Chinese; English learns frequent whole words and endings.
* ⚠️ **Partly supported:** the larger vocabulary helps Chinese most (true from 2k to 10k, but relative to characters Chinese gains almost nothing); Turkish "in between" (only at 2k).
* ❌ **Not supported:** BPE-2k is the best tokenizer for language modeling. BPE-10k is best in every language, and BPE-2k only ties with the character model overall because its gains for English and Turkish cancel its loss for Chinese.

## Status summary

| # | Item | Status |
|---|---|---|
| 1 | BPE vocabulary sizes and why | ✅ Done |
| 2 | Initial hypotheses and verdicts | ✅ Done (no LM-performance hypothesis was written before training) |
| 3 | Parameter counts | ✅ Answered |
| 4 | One sentence per language, all tokenizers | ✅ Done |
| 5 | Tokenizer statistics | ✅ Done |
| 6 | Training/validation curve | ✅ Answered |
| 7 | Interesting observation | ✅ Done and confirmed |
| 8 | Question for the final experiments | ✅ Answered (chosen after the checkpoint) |