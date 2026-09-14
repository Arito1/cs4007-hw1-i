# HW1 submission

**Name: Arthur**
**Student ID: S23068966**
**Group: AI**
**Repository: https://github.com/Arito1/cs4007-hw1-i.git**

## AI tool disclosure

State which AI tools you used and for what. Expected and fine; undisclosed use
is not.

>

---

## Sublab Easy — the registration bot and its bill

**How I laid the catalogue out inside the system prompt, and why:**

> Every course is listed as one line with code, title, credits, prerequisites,
> full schedule (day + start-end time), instructor, and seats left out of
> seats total — everything needed to check a request without guessing. The
> student's completed courses and the credit min/max are stated up front. The
> prompt closes by explicitly calling the catalogue "the ONLY source of truth"
> and instructing the model to refuse anything not listed, which is what held
> the line on turn 4.

**My turn 5 (Kazakh or Russian):**

> Мен үшінші курс студентімін. Мен әлі қандай курстарға тіркеле аламын?

### Run 1 — Groq, `openai/gpt-oss-20b` (standing in for `gpt-5.6-luna` per instructor approval)

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 747 | 613 | 0.000240 |
| 2 | 1057 | 377 | 0.000192 |
| 3 | 1211 | 277 | 0.000174 |
| 4 | 1355 | 120 | 0.000138 |
| 5 | 1456 | 1241 | 0.000481 |
| **total** | 5826 | 2628 | 0.001225 |

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it:free`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 771 | 401 | 0.000000 |
| 2 | 1191 | 108 | 0.000000 |
| 3 | 1317 | 107 | 0.000000 |
| 4 | 1440 | 22 | 0.000000 |
| 5 | 1487 | 515 | 0.000000 |
| **total** | 6206 | 1153 | 0.000000 |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**Groq (`openai/gpt-oss-20b`, instructor-approved substitute for OpenAI):**

```
I'm sorry, but CSS-4090 "Quantum Machine Learning" is not in the current Narxoz University catalogue for the 2026-FALL term. Therefore I can't add it to your schedule. If you have another course in mind from the listed catalogue, let me know and I'll check its availability for you.
```

**OpenRouter (`google/gemma-4-26b-a4b-it:free`):**

```
I am sorry, but CSS-4090 "Quantum Machine Learning" is not in the catalogue.
```

### Written answers

**1. The two providers used almost identical code. What actually changed, and
what did not?**

>

**2. Why did the input token count climb on every turn when your questions
stayed roughly the same length? Use the numbers from your own table. What
happens to the bill at fifty turns?**

>

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

> Both bots refused. Groq: "CSS-4090 'Quantum Machine Learning' is not in the
> current Narxoz University catalogue for the 2026-FALL term. Therefore I can't
> add it to your schedule." Gemma (OpenRouter): "CSS-4090 'Quantum Machine
> Learning' is not in the catalogue." Neither invented credits, a room, or an
> instructor. What held the line was the explicit instruction in the system
> prompt to refuse anything not in the catalogue, combined with listing the
> catalogue as a closed set ("the ONLY source of truth") rather than leaving the
> model to guess whether an unlisted course might exist elsewhere.

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

> Checked against `courses.json` directly: both bots got every trap right.
> Seat counts matched exactly (CSS-4007: 2/40, CSS-4102: 15/35, FIN-3300: 1/30),
> both caught the Tuesday 09:00-10:50 collision on turn 2, and both correctly
> excluded CSS-4400 (full, 0/25 seats) and CSS-3011 (full AND already
> completed) from the eligible list on turn 1. No hallucinated numbers found.
> The one real error I found was outside what the question asks about: in
> turn 5, Groq's Kazakh reply used Turkish day names ("Salı", "Perşembe",
> "Cuma") instead of Kazakh ("Сейсенбі", "Бейсенбі", "Жұма") — the courses and
> numbers were still correct, but the language mixed in a related-but-wrong
> Turkic language mid-sentence.
---

## Sublab Medium — one task, six models

Paste the per-model summary printed by `correct_kazakh.py`:

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-4-26b-a4b-it:free | 4 | 0 | 1534 | 0.00000 |
| qwen/qwen3.8-27b | 0 | 8 | 0 | 0.00000 |
| deepseek/deepseek-v4-flash-0731 | 7 | 0 | 6354 | 0.00159 |
| openai/gpt-oss-20b (groq) | 2 | 1 | 7725 | 0.00204 |
| openai/gpt-oss-120b (groq) | 4 | 0 | 6993 | 0.00355 |
| qwen/qwen3.6-27b (groq) | 0 | 7 | 2180 | 0.00622 |

### Which error types did each model repair?

Rows are error labels, columns are models. Write "yes", "no" or "partial".

| Error type | gemma | qwen3.8 (or) | deepseek | gpt-oss-20b | gpt-oss-120b | qwen3.6 (groq) |
|---|---|---|---|---|---|---|
| kaz_to_rus | yes | N/A | yes | yes | partial | insufficient data |
| latin_homoglyph | yes | N/A | yes | yes | yes | N/A |
| drop_hyphen | partial | N/A | yes | partial | partial | N/A |
| join_words | yes | N/A | yes | yes | yes | N/A |
| double_letter | yes | N/A | yes | no | yes | N/A |

*(N/A = model produced zero usable output for every sentence with this error
type; "insufficient data" = only one sentence out of eight came back at all.)*

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

> Better than the raw `exact` column suggests. `char_diff` for this row was
> originally computed with a buggy positional comparison (`zip` over the two
> strings) that reports a huge diff whenever one earlier character shifts the
> alignment of everything after it - I found and fixed this (now uses real
> Levenshtein edit distance) after noticing a 71-character "diff" between two
> strings that were identical except for one missing character. With the fix,
> gemma's `latin_homoglyph` sentence (KZ-03) is not exact only because it also
> added a comma and a question mark - the homoglyph substitution itself was
> fixed correctly (edit distance 2, both punctuation). Same story for
> gpt-oss-120b (exact on KZ-03) and gpt-oss-20b (edit distance 1). Every model
> that returned output at all repaired the homoglyph correctly - the two
> models that show "N/A" here (qwen3.8-27b, qwen3.6-27b) never got the chance,
> they failed on every sentence for unrelated reasons (see the model-level
> notes below).

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

> gemma on KZ-01 (`kaz_to_rus`): the published original is singular
> ("мемлекеттің елшісінен" - "of the state's ambassador"), gemma returned a
> grammatically valid plural ("мемлекеттердің елшілерінен" - "of the states'
> ambassadors") - different, equally correct Kazakh, not an error.
> deepseek on KZ-01: identical in substance, edit distance 2 from just an
> added trailing period plus one incidental letter choice - not a language
> mistake. On the other hand, gpt-oss-120b's KZ-01 output swapped the
> idiomatic diplomatic term "сенім грамоталарын" ("letters of credence") for
> the literal "сенім хаттарын" ("trust letters") - grammatically fine but not
> the term an official news report would use, which is a real (if minor)
> lexical error, not just a stylistic variant.

**Model-level notes (why the two Qwen rows have gaps):**

> `qwen/qwen3.8-27b` (OpenRouter) failed all 8 sentences with HTTP 402, not a
> rate limit - the request had no `max_tokens` cap, so the client reserved up
> to 131,072 output tokens against the account's OpenRouter balance and got
> rejected for insufficient credits before a single token was generated.
> `qwen/qwen3.6-27b` (Groq) is a reasoning/preview model: it emits a full
> `<think>...</think>` block before its JSON answer, and Groq enforces a hard
> 1000-output-token-per-minute cap on this model that the request (2048
> requested) blew past on 7 of 8 calls; the one that got through (KZ-01) also
> broke the original `parse_response`, because its `<think>` block contained
> several draft JSON snippets and the naive "first `{` to last `}`" regex
> swallowed all of them into one invalid blob. I fixed both: `correct_with`
> now caps `max_tokens` per model (500 for plain instruct models, 1200 for
> gpt-oss, exactly 1000 for qwen3.6-27b to respect its Groq limit), and
> `parse_response` now strips `<think>` blocks and scans for balanced
> `{...}` spans instead of a greedy regex, taking the last one that parses.
> These fixes are in the code but a full rerun of the two Qwen rows was not
> repeated for this submission - the reasons for the original failures are
> reported here since a model failing outright is a legitimate result too.
**Cheapest model that was good enough, and why:**

> `deepseek/deepseek-v4-flash-0731` — 7/8 exact matches for $0.00159 total. The
> only model with a comparable hit rate is `openai/gpt-oss-120b` (4/8) at more
> than double the cost ($0.00355). `google/gemma-4-26b-a4b-it:free` is
> technically cheaper (free), but at 4/8 exact it's a materially worse hit
> rate for a task where correctness is the whole point. If "good enough"
> means free and roughly half-right, gemma; if it means actually reliable,
> deepseek is the best value by a wide margin.
---

## Sublab Harder — open the tokenizer

### A. What a language costs

**`cl100k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 200 | 263 | 0.760 | 3.75 | 0.01999 |
| ru | 129 | 277 | 0.466 | 2.30 | 0.01291 |
| en | 59 | 291 | 0.203 | 1.00 | 0.00591 |

**`o200k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 84 | 263 | 0.319 | 1.58 | 0.00839 |
| ru | 74 | 277 | 0.267 | 1.32 | 0.00740 |
| en | 59 | 291 | 0.203 | 1.00 | 0.00591 |

*($ figures use `qwen/qwen3.6-27b`'s $0.60/M input rate on Groq, the instructor-approved substitute for `gpt-5.6-sol`.)*

### B. What a homoglyph does

One row per `latin_homoglyph` sentence in the dataset. Paste the actual decoded
token strings around the divergence point, not a description of them.

| Sentence id | Foreign char (index, name) | Tokens correct | Tokens corrupted | Δ | Diverges at |
|---|---|---|---|---|---|
| KZ-03 | idx 0 'A' LATIN CAPITAL LETTER A; idx 2 'a' LATIN SMALL LETTER A; idx 5 't' LATIN SMALL LETTER T | 16 | 20 | +4 | 0 |
| KZ-08 | idx 1 'o' LATIN SMALL LETTER O; idx 3 'a' LATIN SMALL LETTER A; idx 9 'T' LATIN CAPITAL LETTER T | 21 | 24 | +3 | 1 |

**Token pieces around the divergence:**

```
KZ-03
correct  : ['А', 'лая', 'қ', 'тарға', ' ақша']
corrupted: ['A', 'л', 'a', 'я', 'қ']

KZ-08
correct  : ['Д', 'он', 'аль', 'д', ' Т', 'рамп']
corrupted: ['Д', 'o', 'н', 'a', 'л', 'ль']
```

### C. Did it get better?

| Language | cl100k_base | o200k_base | Change |
|---|---|---|---|
| kk | 0.760 | 0.319 | -0.441 (-58%) |
| ru | 0.466 | 0.267 | -0.199 (-43%) |
| en | 0.203 | 0.203 | 0.000 (0%) |

### Written answers

**1. What is the Kazakh tax?** The ratio against English in both encodings, the
dollar figure from A, and how much it changed between the two tokenizers.

> On `cl100k_base`, Kazakh costs 3.75x English (200 tokens vs 59 for the same
> six meanings) — $0.01999 vs $0.00591 per 1,000 sentences. On `o200k_base`
> the ratio drops to 1.58x (84 vs 59 tokens) — $0.00839 vs $0.00591. The newer
> tokenizer cut the Kazakh tax by more than half (3.75x -> 1.58x, a 58%
> reduction in the surcharge), but did not eliminate it: Kazakh still costs
> noticeably more per sentence than English even on the model with the
> larger, more multilingual vocabulary.

**2. Why did the models repair `kaz_to_rus` but struggle with
`latin_homoglyph`?** Both are single-letter substitutions and both look almost
identical on screen. Use your token streams from B as the evidence. Say what the
model actually received in each case.

> `kaz_to_rus` stays inside the Cyrillic block — it swaps a Kazakh-specific
> Cyrillic letter for a visually similar Russian Cyrillic one, so the
> tokenizer still segments the sentence into Cyrillic subword pieces the
> model has seen thousands of times in Russian text, and it can pattern-match
> its way back to the intended Kazakh word. `latin_homoglyph` is a different
> story at the byte level: swapping Cyrillic 'А' for Latin 'A' moves that
> character into a completely different Unicode block. The KZ-03 token
> stream shows this directly — the correct sentence tokenizes into
> recognizable multi-character pieces (`'лая'`, `'тарға'`, `' ақша'`), while
> the corrupted one immediately fragments into single, mostly meaningless
> characters (`'A'`, `'л'`, `'a'`, `'я'`, `'қ'`) starting from the very first
> token. The model isn't looking at "a word with one weird letter" — it is
> looking at a sequence of near-random single-character tokens with no
> subword structure to recognize, which is a much harder pattern to repair.

**3. Name one thing this measurement does not explain about your Sublab Medium
results.** You measured OpenAI's tokenizers; three of your six models were not
OpenAI's. What follows, and what would you have to do to close the gap?

> This only measured `cl100k_base` and `o200k_base`, which are OpenAI's
> tokenizers. Half of the Sublab Medium models (gemma, the two Qwen models,
> deepseek) use entirely different tokenizers with their own vocabularies —
> there's no guarantee they fragment the same homoglyph the same way `o200k_base`
> does. A model with a Kazakh- or Cyrillic-heavy training/tokenizer vocabulary
> could conceivably handle `latin_homoglyph` better (or worse) than what these
> two encodings predict. To actually close the gap, I'd need to load each
> model's own tokenizer (e.g. via its Hugging Face `tokenizer.json` for the
> open-weight ones) and rerun measurement B per model, instead of assuming
> OpenAI's tokenizer behavior generalizes to models that never used it.