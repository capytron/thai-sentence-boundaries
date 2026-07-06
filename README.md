---
license: cc-by-2.0
language:
- th
task_categories:
- token-classification
tags:
- thai
- thai-nlp
- sentence-segmentation
- sentence-boundary-detection
pretty_name: Thai Sentence Boundaries (permissive gold set)
size_categories:
- n<1K
configs:
- config_name: default
  data_files:
  - split: test
    path: data/all.jsonl
---

# Thai Sentence Boundaries — a small permissive gold set

> **Licensing note (repo label vs. per-record).** This card is labeled
> `cc-by-2.0` as the single most-binding license across the set so downstream use
> is always safe; **most records are actually CC0** (see the `license` field on
> each row and the [License & attribution](#license--attribution) section). Only
> the `tatoeba_synth` split carries CC-BY (attribute Tatoeba); everything else is
> public-domain / CC0.

A small, **fully permissive** gold set of **Thai sentence boundaries** — 905
hand-verified sentences across **8 registers**, released so anyone can evaluate
Thai sentence segmenters without a licensing tangle. Every record is **CC0** or
**CC-BY** (Tatoeba); there are no non-commercial, no-derivatives, share-alike, or
gated parts.

> **Why.** The established Thai sentence-boundary corpora — ORCHID (CC-BY-**NC**-SA),
> LST20 / Blackboard Treebank (NECTEC, gated, redistribution restricted) — are
> non-commercial or gated. We wanted something small but **fully permissive
> (CC0 / CC-BY)** to evaluate on and share freely, so we made and released this.
> It's a starting point, not a comprehensive corpus — use it, fork it, add to it.

---

## Table of contents
- [What's in it](#whats-in-it)
- [The registers, in detail](#the-registers-in-detail)
- [How it was made (provenance & method)](#how-it-was-made-provenance--method)
- [Data format](#data-format)
- [How to use it](#how-to-use-it)
- [Reference results](#reference-results)
- [Honest limitations](#honest-limitations)
- [License & attribution](#license--attribution)
- [Contributing](#contributing)
- [Citation](#citation)

---

## What's in it

| register | docs | sentences | license | one-line |
|---|---:|---:|---|---|
| `handgold_prose` | 16 | 182 | CC0-1.0 | hand-annotated free prose (public-domain literature + PD statutes) |
| `tatoeba_synth` | 150 | 592 | CC-BY-2.0-FR | native Thai Tatoeba sentences concatenated → exact boundaries |
| `formal_news` | 1 | 22 | CC0-1.0 | formal news register |
| `academic_legal` | 1 | 21 | CC0-1.0 | academic / legal-statute register |
| `novel_dialogue` | 1 | 23 | CC0-1.0 | narrative + nested dialogue quotes |
| `social_chat` | 1 | 22 | CC0-1.0 | informal chat, no terminal punctuation |
| `poetry_lyrics` | 5 | 21 | CC0-1.0 | poems / lyrics with line breaks |
| `tricky_tokens` | 1 | 22 | CC0-1.0 | abbreviation / date / decimal / URL false-boundary traps |
| **total** | **176** | **905** | | 26 docs CC0 · 150 docs CC-BY |

Each register lives in `data/<register>.jsonl`; `data/all.jsonl` concatenates all
of them (the `register` field lets you split back out).

## The registers, in detail

Thai has no obligatory sentence-terminating punctuation and uses spaces both
between words/phrases *and* between sentences, so sentence segmentation is a real
disambiguation task. These registers were chosen to stress the parts that make it
hard, from different angles:

- **`handgold_prose` (182)** — the flagship set: real, connected prose, not
  synthetic. Two sub-sources, both public domain:
  - *Classical literature*: นิทานเวตาล (translated by Prince Bidyalongkorn,
    d. 1945) and ราชาธิราช (translated by Chaophraya Phrakhlang (Hon), d. 1805) —
    out of copyright by author-death term. Narrative prose with embedded dialogue.
  - *Statutes*: the preamble of the Civil and Commercial Code (B.E. 2466 / 1923)
    and the Interim Constitution (B.E. 2557 / 2014) — statutes, regulations and
    gazette text are **not** subject to copyright under the Thai Copyright Act
    B.E. 2537, Section 7. Long, clause-dense formal sentences.
- **`tatoeba_synth` (592)** — each document is several independent, *native* Thai
  [Tatoeba](https://tatoeba.org) sentences joined with single spaces. Because we
  control the join, the boundaries are **exact (100% correct)**. This gives clean,
  high-volume boundary signal and, crucially, natural intra-sentence spaces (e.g.
  a space before ๆ) that must **not** be cut — a good false-positive test.
- **`formal_news` (22)** — long noun-phrase sentences with few lexical end cues;
  where "cut every space" over-segments and models must learn restraint.
- **`academic_legal` (21)** — มาตรา / พ.ศ. / ordinal section numbers, whose
  periods are candidate false boundaries.
- **`novel_dialogue` (23)** — narration interleaved with dialogue in nested
  quotation marks (“ ” « » 「 」), including a quote-inside-a-quote case.
- **`social_chat` (22)** — colloquial, run-on, **no** terminal punctuation —
  boundaries carried only by particles/context.
- **`poetry_lyrics` (21, 5 poems)** — line breaks that are *sometimes* sentence
  ends and sometimes enjambment; deliberately mixes both.
- **`tricky_tokens` (22)** — deliberate traps that must **not** trigger a
  boundary: ด.ช. / พ.ศ. / ค.ศ. / ม.ค. / น.ส. / กทม. / ดร., decimals (3.14),
  dates (5 ม.ค. 2567), URLs, and mixed Thai-English-Chinese.

## How it was made (provenance & method)

- **Hand-written sets** (`formal_news`, `academic_legal`, `novel_dialogue`,
  `social_chat`, `poetry_lyrics`, `tricky_tokens`): authored by hand for
  evaluation, each targeting the phenomenon described above. Original work,
  released CC0.
- **`handgold_prose`**: sentences were split by reading the public-domain source
  and then **reconstruction-verified programmatically** against the exact source
  string (byte-for-byte) to catch transcription slips. The annotation rulebook
  covers: dialogue with/without quotation marks; question/imperative-final
  particles; enumerated clauses kept as one sentence vs. split; multi-token dates
  and eras; and ฯลฯ (et cetera — sentence-internal) vs. a mid-clause honorific ฯ
  such as โปรดเกล้าฯ / กรุงเทพฯ (never a boundary). One passage with source OCR
  corruption was excluded rather than silently fixed.
- **`tatoeba_synth`**: built by concatenating distinct native Thai Tatoeba
  sentences with single spaces. The underlying sentences are **held out** from any
  model training we did, so it is safe as an evaluation split. Boundaries are
  exact by construction.
- **Validation**: every record's boundaries are strictly ascending and the final
  boundary equals the document's rune length; the provided loader reconstructs
  all 176 documents without error.

We deliberately used **no** NC / ND / SA or gated corpora (no ORCHID, LST20, TED,
FLORES) at any point, so this set is free to redistribute and build on.

## Data format

One JSON object per line:

```json
{
  "text": "<full document text, exactly as it should be segmented>",
  "sentences": ["<sentence 1>", "<sentence 2>", "..."],
  "register": "handgold_prose",
  "license": "CC0-1.0",
  "source": "hand-annotated PD/CC0 prose"
}
```

`sentences` are the ordered gold sentences that make up `text`. We ship the raw
`text` **and** the `sentences` list (rather than pre-computed integer offsets)
because some registers contain intra-document line breaks and non-space joins
(poetry, dialogue); this way you derive boundary positions under *your own*
tokenization and metric, with no lossy normalization on our side. A gold boundary
sits at the end of each sentence.

## How to use it

Derive boundary positions (rune indices) from `text` + `sentences`:

```python
import json

def gold_cuts(rec):
    """Rune index after which each gold sentence ends (last == len(runes))."""
    text, cuts, pos = list(rec["text"]), [], 0
    for s in rec["sentences"]:
        # skip any leading whitespace/newline, then consume the sentence
        while pos < len(text) and "".join(text[pos:pos+len(s)]) != s and text[pos].isspace():
            pos += 1
        pos += len(s)
        cuts.append(pos)
    return cuts

for line in open("data/all.jsonl", encoding="utf-8"):
    rec = json.loads(line)
    text, gold = rec["text"], set(gold_cuts(rec))
    # compare your segmenter's predicted cut positions against `gold`
```

**Metrics.** The two standard measures for Thai sentence segmentation:

- **Space-correct accuracy** — accuracy over inter-word-space decision points
  (did we correctly keep vs. cut each space?). Dominated by non-boundaries, so it
  rewards precision on the majority-negative spaces.
- **Boundary-class F1** — precision/recall/F1 over sentence-end positions. More
  sensitive to recall on the (minority) true boundaries.

Report both — they can disagree, and a segmenter tuned for one may trade the other.

## Reference results

Measured with [thai-nlp-go](https://github.com/sukitss/thai-nlp-go)'s
`sentence/charseg` (character-level, pure-Go, no tokenizer) and `sentence/crf`
(a port of PyThaiNLP's crfcut) engines. Macro-averaged boundary-F1, equal weight
per register, over the permissive sets here:

| engine | macro boundary-F1 | note |
|---|---:|---|
| whitespace (cut every space) | 0.666 | recall 1.0, low precision |
| crf.Default (crfcut port) | 0.559 | word-level CRF, needs a tokenizer |
| charseg.Default | 0.663 | char-level, no tokenizer, ~24× faster than the CRF |
| charseg, recall-lean τ | **0.698** | operating-point tuned on held-out data |

These are honest and register-dependent — **no single engine wins everywhere**.
Formal news favors "cut every space"; dialogue, tricky-tokens and the free-prose
gold favor the learned model. On the out-of-domain academic ORCHID corpus (which
this set does not include, being NC), a same-data head-to-head has the char-level
model within ~0.01–0.02 boundary-F1 of the word-CRF at ~24× the speed. Treat the
numbers above as a starting reference, not a leaderboard.

## Honest limitations

- **Small.** 905 sentences total — a seed/eval set, not a training corpus or a
  comprehensive benchmark. ORCHID (~30k sentences) and LST20 (~74k) are far larger
  (but NC/gated).
- **Register imbalance.** The six hand-written registers are ~20 sentences each
  (one document apiece, except poetry with five); `tatoeba_synth` and
  `handgold_prose` are larger. Macro-averaging treats each register equally to
  compensate, but small per-register counts mean per-register numbers are noisy.
- **`tatoeba_synth` is synthetic** — real sentences, but the *document* boundaries
  come from our concatenation, not from naturally occurring multi-sentence text.
- **Single-source annotation.** Boundaries reflect one annotation pass with the
  documented rulebook; there is no multi-annotator agreement study. Sentence
  segmentation has genuine gray areas (clause vs. sentence, enumerations) — we
  aimed for consistency, not a claim of ground truth.

## License & attribution

Per-record, via the `license` field. No NC / ND / SA / gated components.

- **CC0-1.0** — all hand-written sets and `handgold_prose`. Public-domain
  dedication: copy, modify, distribute, use (even commercially), no attribution
  required. Underlying `handgold_prose` texts are themselves public domain
  (classical literature by age; Thai statutes are non-copyright under the Thai
  Copyright Act B.E. 2537 §7); the annotation is our original CC0 work.
  <https://creativecommons.org/publicdomain/zero/1.0/>
- **CC-BY-2.0-FR** — the `tatoeba_synth` split. The sentence *text* is from
  [Tatoeba](https://tatoeba.org) (CC-BY 2.0 FR): **attribute Tatoeba** if you use
  it. The concatenation + boundary annotation are ours (CC0), but the underlying
  sentences remain Tatoeba's under CC-BY.
  <https://creativecommons.org/licenses/by/2.0/fr/>

Filter on the `license` field if you need a strictly CC0-only subset.

## Contributing

More permissive registers are very welcome — especially larger natural
free-prose from public-domain or CC0/CC-BY sources, or additional annotators on
the existing docs. Open an issue or PR. Please keep everything CC0 / CC-BY / PD;
no NC / ND / SA / gated material.

## Citation

```
Thai Sentence Boundaries — a small permissive gold set.
https://github.com/capytron/thai-sentence-boundaries
Companion to the thai-nlp-go project: https://github.com/sukitss/thai-nlp-go
```
