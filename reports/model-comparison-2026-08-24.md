# Two models on one gold set, 2026-08-24

Build plan phase 21 C; decision 55(e). The question this was built to answer:
**does the cheap model, grounded, beat the dear model's raw knowledge — and at
what multiple of the price?**

The rules for reading it were written before the numbers existed, in the
proposal's §6, precisely so the numbers could not bend them. They are applied
below without exception, including where they cost the story.

## The two reports

| | mini | full |
| --- | --- | --- |
| Report | [`eval-20260824-012947`](eval-20260824-012947.md) | [`eval-20260824-015436`](eval-20260824-015436.md) |
| `chat_deployment` | `gpt-5.4-mini` | `gpt-5.4` |
| `judge_deployment` | `gpt-5.4-mini` | `gpt-5.4-mini` |
| Price (in / out, per Mtok) | $0.75 / $4.50 | $2.50 / $15.00 |

Both over `eval/gold-set.jsonl` (43 records: 13 core, 13 lift, 17 matter),
`gold_set_sha256_12` `285391c54cc8`, corpus snapshot 2026-06-30 at 13,318
documents / 113,164 chunks, `arm_temperature` 0 and `arm_seed` 20260811.

## What makes these comparable

Checked mechanically rather than by eye, because a header is forty-one rows and
the one that matters is the one nobody reads. Diffing the two headers:

**`chat_deployment` is the only fingerprinting field that differs.** Everything
else — every prompt fingerprint, the agent's tool fingerprint, the ranking
rules hash, the retrieval settings hash, the corpus definition, the seed — is
identical.

Two independent confirmations that the two runs saw the same system:

- **The retrieval tables are identical to three decimal places**, every stratum,
  every metric. They should be: retrieval does not consult a chat model.
- **`usage.baseline.input_tokens` is identical** — 845,185 on both. The baseline
  arm's input is the question plus the retrieved passages, so an exact match is
  proof the two runs were handed the same passages in the same order.

**The judge did not move.** `judge_deployment` reads `gpt-5.4-mini` in both.
This is the rake decision 55(e) exists to keep anyone off: the judge shares the
one `IChatClient` with the answer arms, so comparing models by editing
`AzureOpenAI__ChatDeployment` would have swapped the instrument along with the
subject and invalidated the human agreement `judge-validation.md` records.
`--model` moves the arms by passing them a client and never touches
configuration, so it cannot reach the judge.

## What it cost

Arms only. The judge's completions are excluded on purpose — folding the
instrument into the measurement is how a comparison stops comparing.

| Arm | mini | full | |
| --- | --- | --- | --- |
| closed-book | $0.0745 | $0.4046 | |
| baseline | $0.6772 | $2.3870 | |
| agent | $0.8589 | $2.5166 | |
| **total** | **$1.6106** | **$5.3081** | **3.30x** |

**The cost ratio needs no statistics, and 3.30x is the honest number** — not
the 9x the proposal used illustratively. Two things drive that, and they pull
against each other:

- gpt-5.4's list prices are exactly **3.33x** the mini's on both sides
  ($2.50/$0.75, $15.00/$4.50), so on identical token volumes the ratio would be
  3.33x and nothing else.
- Token volumes did *not* stay identical. `full` sent **fewer** input tokens
  (1.65M against 1.87M) because its agent arm reformulated less — 101
  completions against 127 — and generated **more** output (78.7K against 45.6K),
  because it writes longer answers. Those roughly cancel.

So the price ratio is essentially the whole cost story here. A tenth of the
price was never available between these two models.

**These dollar figures are upper bounds.** gpt-5.4 lists a cached input rate of
$0.25 against $2.50 uncached, and neither `UsageDetails` nor `model_spend`
separates cached from uncached input — two price fields, one input total — so
every input token is priced at the uncached rate. Where the provider served a
cache hit, the real bill is lower, and disproportionately so for `full`. The
mini's own prices come from `.env.example`, which records them with a "verify
against the Azure pricing page" caveat; tokens are in both headers, so if that
figure is wrong the dollars can be recomputed without re-running anything.

## What it bought — the headline

Right behaviour over all 43 records. (`Right behaviour` is a correct answer on
an answerable question and an abstention on an unanswerable one.)

| Arm | mini | full | gap |
| --- | --- | --- | --- |
| closed-book | 26% | 35% | +9 to full |
| baseline | 49% | 60% | +11 to full |
| agent | 49% | 53% | +4 to full |

**The noise floor binds all three of these.** It is 8 points
(`MeasureAggregate.NoiseFloor`), measured as the closed-book arm moving 46% to
54% between two runs of the same records with `arm_seed` unset
(`eval-20260810-121621.md`). It was measured on **13** records and this gold set
has 43, so it is plausibly conservative here — but nobody has measured that, so
8 points is what gets applied.

Under that rule:

- **closed-book, +9: adjudicated, barely.** The dear model knows more unaided.
- **baseline, +11: adjudicated. The dear model is genuinely better, grounded.**
- **agent, +4: not adjudicated.** Inside the floor. No claim either way.

## The selling point, and whether it survives

The claim worth money was *mini grounded beats full's raw knowledge.*

**mini baseline 49% against full closed-book 35% — +14 points to the cheap
model, grounded.** That is the largest gap in this study and comfortably outside
the floor. **The selling point holds at this gold-set size.**

It costs $0.68 against $0.40 — grounding is not free, because the retrieved
passages are input tokens. So the honest sentence is *retrieval buys the cheap
model more than three times its price bought the dear one*, not *the cheap model
is free*.

**And the finding that cuts the other way, stated with equal weight:** the dear
model grounded (60%) beats the cheap model grounded (49%) by 11 points, which is
also outside the floor. **Nothing in this product may claim the two models are
equally good.** What it may claim is what the row above says: grounding a cheap
model beats not grounding an expensive one, by more than grounding an expensive
model beats grounding a cheap one (+14 against +11).

## The case for retrieval got *stronger* on the dear model

The most interesting result here, and not one anybody predicted.

| Closed book, all 43 | mini | full |
| --- | --- | --- |
| Right behaviour | 26% | 35% |
| **Incorrect** | **33%** | **40%** |
| Abstained | 26% | 16% |
| Ungrounded citations emitted | 67 | **123** |

The dear model unaided is right more often **and wrong more often**, because it
abstains far less (16% against 26%) and cites nearly twice as much with nothing
in front of it. Its 123 ungrounded citations are the fabrication tally, and it
is the larger of the two.

So retrieval is not a crutch that a better model outgrows. On this gold set the
better model is the more confident fabricator, and grounding removes more
absolute error from it (40% -> 14% incorrect) than from the mini (33% -> 9%).

## Lift, per stratum

| Stratum | Arm | mini closed book | mini grounded | full closed book | full grounded |
| --- | --- | --- | --- | --- | --- |
| core | baseline | 46% | 77% (+31) | 62% | 77% (+15) |
| core | agent | 46% | 77% (+31) | 62% | 77% (+15) |
| lift | baseline | 15% | 62% (+46) | 31% | 77% (+46) |
| lift | agent | 15% | 54% (+38) | 31% | 69% (+38) |
| matter | baseline | 18% | 18% (+0) | 18% | 35% (+18) |
| matter | agent | 18% | 24% (+6) | 18% | 24% (+6) |

**On `core`, the two models grounded are identical — 77% and 77%.** The entire
gap between them on that stratum is closed-book knowledge that retrieval then
supplies to both. That is the cleanest illustration in the study of what the
product is for, and it is worth more than the headline average.

The gap lives in `lift` (62% against 77%) and `matter` (18% against 35%) — the
questions the model cannot answer from memory and has to *read* an answer out
of retrieved text. That is a reading-comprehension difference, not a knowledge
one, and it is the honest place to say the dear model earns its price.

`matter`'s mini baseline at +0 lift is the weakest cell in the study and was
weak before this comparison; it is a known standing problem, not a new finding.

## The agent arm does not beat the one-shot baseline

On `full`, the agent arm (53%) is *below* the baseline (60%); on mini they tie
at 49%. This predates the comparison and is visible in earlier reports. It is
noted here because a reader looking at two reports side by side will see it
twice and might mistake it for something the model choice caused. It is not.

One genuine model effect on retrieval, though: **agent trace recall rose 0.510
-> 0.616.** The retriever is identical, so the whole of that is the dear model
writing better search queries. It found more of the expected authority and still
scored lower — worth its own look, separately.

## Deadline extraction

`deadline-proposals`, both models, same fixture (2 documents, 19 expected
dates), prompt `7bdee2da6e84`, checks `proposal-v1`. There is no report file for
this verb — it prints — so the numbers are transcribed here.

| | mini | full |
| --- | --- | --- |
| Recall | 100% (19/19) | 100% (19/19) |
| Precision | 95% (20/21) | 90% (19/21) |
| **Drop rate** | **0% (0/23)** | **0% (0/29)** |
| Obligations reported | 23 | 29 |
| Listed, undated | 2 | 8 |

**The drop rate is the number decision 55(d) sends to this phase**, on the
theory that a cheaper extractor might quote less faithfully. **It does not: both
models are at zero.** No proposal from either model quoted a sentence that was
not in the document.

The mini is one spurious date more precise; `full` reports more obligations
overall and lists six more undated ones. On 21 proposed dates across two
documents, **none of that is adjudicable** and no claim is made from it. What
the run does establish is the negative: there is no fidelity penalty visible
here for using the cheap extractor, which is the thing that would have argued
against offering the picker at all.

## What this study does not cover

- **The matter metrics were skipped** (`--no-matter-metrics`, both runs).
  `MatterMetricsRunner` takes no model, so they cannot vary with `--model`;
  including them would have produced two identical sections at the price of a
  cold record read. Both reports therefore lack a section older committed
  reports have.
- **The text lane was running looser than its header claims, in both runs.**
  `lexeme_stats` on the Azure corpus describes a ~82,000-chunk corpus against
  the 113,164 actually there, so decision 28's selective text arm admitted more
  lexemes than `TextQueryMatchBudget` intends. It applied equally to both runs —
  the retrieval tables are identical — so the *comparison* is unaffected, but
  the absolute retrieval numbers describe a configuration the system was not
  quite in. Deferred deliberately; `docs/ideas.md` entry 21. It must not be
  refreshed between two reports being compared, which is why it was not
  refreshed tonight.
- **43 records is still small.** The gold set remains the binding constraint,
  and growing it remains the standing answer.

## The four sentences a reader should leave with

1. **Grounding a cheap model beats not grounding an expensive one**, by 14
   points, which is outside the noise floor.
2. **The expensive model is still better when grounded**, by 11 points, which is
   also outside the noise floor. Parity may not be claimed.
3. **The price difference is 3.3x, not 10x**, and it is almost entirely list
   price rather than token volume.
4. **Retrieval matters more for the expensive model, not less** — it fabricates
   nearly twice as many citations unaided.
