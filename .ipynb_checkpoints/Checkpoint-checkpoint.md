# Checkpoint: Multilingual tokenization and small Transformer LMs

**Languages:** English (en), Turkish (tr), Mandarin Chinese (zh)
**Status at checkpoint:** Part 1 (tokenizers and analysis) complete. Part 2 (language models) set up, training not yet finished.

---

## 1. BPE vocabulary sizes and why  ✅

I trained two byte-level BPE tokenizers (Hugging Face `tokenizers`) on the shared multilingual `balanced.txt` training data plus a character level tokenizer built from the training text only.

| Tokenizer | Vocabulary size |
|---|---|
| Character-level | 9,443 (+ `<unk>` for unseen characters) |
| BPE-small | 2,000 |
| BPE-large | 10,000 |

**Why these sizes:**
- **2,000** is small: 256 byte symbols plus about 1,750 merges shared by three languages. It stays close to the raw characters, so I expected it to work well for English and Turkish but to struggle with Chinese, which has thousands of different characters.
- **10,000** is 5x larger, enough for many whole words and common Chinese characters, so the difference between the two BPE tokenizers should be visible.
- The embedding table stays small compared with large models, which suits a small Transformer trained on about 9M characters.
- I used **byte-level** BPE so that no text is ever unknown. A normal character-based BPE with only 2,000 tokens could not even contain all Chinese characters.

---

## 2. Initial hypotheses (written before running anything, one thats also in my ipnyb file at start)  ✅

1. **Most tokens:** Chinese will need the most tokens, English the fewest, Turkish in between. Reason: a Chinese character is 3 bytes in UTF-8, so in a small byte-level vocabulary rare characters stay in pieces.
2. **Larger BPE vocabulary:** fewer tokens and more characters per token for every language, with the biggest improvement for Chinese, since 10,000 slots can hold most common characters.
3. **Units learned:** English will get frequent whole words (`the`, `of`, `and`) and endings (`ing`, `tion`, `ed`). I did not have a prediction for Turkish and Chinese units, so I decided to explore them experimentally.
4. **Best tokenizer for language modeling:** BPE-2k or BPE-10k, leaning to BPE-2k on this small dataset (shorter sequences than characters, but few rare tokens). I expect the character model to be the slowest and probably the worst, and I am least sure about BPE-10k.

**Additional hypothesis about LM performance (written before training):**
None so far

### What I expected vs. what Part 1 showed
- Chinese needs the most tokens: **confirmed** (410k and 295k tokens vs. about 164k and 114k for English).
- English needs the fewest: **true at 2k** (163.8k vs. 168.8k Turkish) but essentially **not same at 10k** (114.7k vs. 114.3k).
- Biggest improvement for Chinese: **only partly right.** Chinese saved the most tokens in absolute terms (−115.6k), but the least in relative terms (−28.2%, compared with −30.0% English and −32.3% Turkish).

---

## 3. Parameter counts of the three models  ⏳ still to be explored

---

## 4. One sentence per language under all three tokenizers  ✅

`|` separates tokens; `▁` marks a space; `�` is a fragment of a multi-byte character (byte-level BPE split one character into bytes).

**English:** *Oranges are sometimes found in Renaissance paintings of married couples.*
- Character (72 tokens): `O | r | a | n | g | e | s | ▁ | a | r | e | ...` (one token per character)
- BPE-2k (33 tokens): `O | r | ang | es | ▁are | ▁s | om | etim | es | ▁f | ound | ▁in | ▁R | en | a | is | s | an | ce | ▁p | ain | t | ings | ▁of | ▁m | ar | ri | ed | ▁c | o | up | les | .`
- BPE-10k (20 tokens): `Or | ang | es | ▁are | ▁sometimes | ▁found | ▁in | ▁R | ena | iss | ance | ▁pain | t | ings | ▁of | ▁married | ▁co | up | les | .`

**Turkish:** *Gölgem Hanım'dan Mahmut adında bir oğlu vardır.*
- Character (47 tokens): one token per character
- BPE-2k (23 tokens): `G | öl | g | em | ▁H | an | ım | 'd | an | ▁M | ah | m | ut | ▁ad | ında | ▁bir | ▁o | ğ | lu | ▁v | ard | ır | .`
- BPE-10k (17 tokens): `G | öl | g | em | ▁Han | ım | 'd | an | ▁Mah | m | ut | ▁ad | ında | ▁bir | ▁oğlu | ▁vardır | .`

**Chinese:** *正德六年（1511年）中式辛未科會試第一百九十名，登第三甲第一百四十九名進士 citation citation 。*
- Character (58 tokens): one token per character
- BPE-2k (40 tokens): `正 | 德 | 六 | 年 | （ | 15 | 11 | 年 | ） | 中 | 式 | � | � | 未 | 科 | 會 | � | � | 第一 | 百 | 九 | 十 | 名 | ， | 登 | 第 | 三 | � | � | 第一 | 百 | 四 | 十 | 九 | 名 | 進 | 士 | ▁citation | ▁citation | ▁。`
- BPE-10k (31 tokens): `正 | 德 | 六年 | （ | 15 | 11 | 年 | ） | 中式 | 辛 | 未 | 科 | 會 | 試 | 第一 | 百 | 九 | 十 | 名 | ， | 登 | 第三 | 甲 | 第一 | 百 | 四十 | 九 | 名進士 | ▁citation | ▁citation | ▁。`

*(The stray English word "citation" is a leftover in the original Wikipedia data. [OPTIONAL: replace this Chinese sentence with a cleaner one.])*

**Short notes on these examples:**
- English: frequent words (`▁are`, `▁of`, `▁in`) are single tokens at both sizes, while `▁sometimes`, `▁found` and `▁married` only become single tokens at 10k. Rarer words like "Renaissance" stay in pieces (`▁R | ena | iss | ance`).
- Turkish: suffix-like pieces (`ım`, `ında`, `an`) are learned. At 10k, whole words such as `▁vardır` and `▁oğlu` appear as single tokens.
- Chinese: at 2k, rarer characters (`辛`, `試`, `甲`) break into `�` byte fragments. At 10k they are whole tokens, and multi-character units such as `六年`, `第一`, `中式` appear.

---

## 5. Preliminary tokenizer statistics (validation set)  ✅

![Average characters per token by tokenizer and language (validation set)](outputs_part1/chars_per_token1.png)

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

**Extra measurements:**
- Character tokenizer `<unk>` rate on validation: en 0.0133%, tr 0.0041%, zh 0.0701%. BPE has 0% (byte-level).
- Share of tokens that are partial-byte fragments: BPE-2k: en 0.4%, tr 0.8%, **zh 35.4%**. BPE-10k: en 0.3%, tr 0.4%, **zh 7.6%**.
- Tokens per word: English 2.57 (2k) → 1.80 (10k); Turkish 3.47 (2k) → 2.35 (10k). Turkish words are longer (6.6 vs 4.9 characters).
- Chinese vocabulary content: BPE-2k has 662 single Chinese characters and 49 multi-character tokens; BPE-10k has 2,391 single characters and 2,056 multi-character tokens.

---

## 6. Preliminary training / validation curve  ⏳ still to be explored

---

## 7. An interesting observation so far  ✅

**Chinese under the small BPE is worse than plain characters.**
BPE-2k needs **410,248 tokens** for Chinese, which is more than the 368,266 characters, and **35.4%** of its Chinese tokens are byte fragments. The phrase `经济发展` is split as `经 | � | � | 发 | 展` by BPE-2k but as `经济 | 发展` by BPE-10k.

My explanation: Chinese characters take 3 bytes. With only about 1,750 merges shared among three languages, only the most frequent Chinese characters get merged into a single token. The rest stay as 2-3 byte pieces, so for Chinese a "compressing" tokenizer actually stretches the sequence.

*Other observations I may follow up on:*
- Turkish `evlerimizden` → `ev | ler | im | iz | den` lines up with real suffixes (plural, possessive, ablative), even though BPE knows nothing about morphology. The suffixes are simply very frequent.
- English and Turkish have almost identical characters per token at 10k (3.210 vs 3.223), although Turkish needs more tokens per word (2.35 vs 1.80).
- The Chinese data mixes traditional and simplified characters (`一个` and `一個` are both among the first merged tokens).

---

## 8. A question I want the final experiments to answer  ⏳ to be answered by training

---

## Status summary

| # | Item | Status |
|---|---|---|
| 1 | BPE vocabulary sizes and why | ✅ Done |
| 2 | Initial hypotheses | ✅ Done (LM hypothesis line to add before training) |
| 3 | Parameter counts | ⏳ Still to be explored |
| 4 | One sentence per language, all tokenizers | ✅ Done (Chinese sentence could be cleaner) |
| 5 | Tokenizer statistics | ✅ Done |
| 6 | Training/validation curve | ⏳ Still to be explored |
| 7 | Interesting observation | ✅ Done |
| 8 | Question for final experiments | ⏳ To choose |