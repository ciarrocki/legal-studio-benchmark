# Legal Studio Benchmark

A reviewed gold set for retrieval-grounded question answering over Delaware
corporate law, together with the published evaluation reports and the method
behind the numbers. The system it measures — a retrieval and verification
system whose answers are checked against a bounded public-sources corpus — can
be seen replaying recorded runs at
[legalstudio-demo.azurewebsites.net](https://legalstudio-demo.azurewebsites.net).

Everything here follows one editorial rule: a quantitative claim names the
committed report it comes from, limitations carry the same weight as results,
and a hedge that is part of the measurement ("on this gold set", "under this
configuration") is kept, not polished away.

## The set

[`gold/gold-set.jsonl`](gold/gold-set.jsonl) — one JSON record per line, 43
records, in the format documented in [`FORMAT.md`](FORMAT.md).

| Stratum | Records | Answerable | What it is for |
| --- | --- | --- | --- |
| `core` | 13 | 11 | Questions a Delaware practitioner would ask. Read the retrieval numbers here; a closed-book model may well pass these, and that is fine. |
| `lift` | 13 | 13 | Questions built to defeat a closed-book model — a superseded doctrine, an amended statute, a threshold the model interpolates. Read the grounded-versus-closed-book delta here. |
| `matter` | 17 | 13 | Questions about a synthetic litigation matter. Closed-book performance is 0% by construction — the matter is generated, and no model has seen it. Never pooled with the other strata. |

Six records are deliberately unanswerable and score the refusal, in two of the
format's four kinds: `absent-from-record` (4) — the question is about what a
matter file establishes, and the record does not establish it — and
`wrong-jurisdiction` (2). A system that correctly abstains on missing law can
still invent a fact; the kinds are scored apart because those are different
failures.

A fourth stratum, `procedural` ("when is it due" — questions a rule book
answers and a corpus cannot), exists as candidates in the instrument and is
not in this set: its records rest on court-rule files nobody has reviewed, and
the review discipline below admits no exception for a careful author.

## The method, in brief

**Contamination is measured, not authored around.** The set is written by
people and tools that read the same law the models were trained on, so every
record carries a `closed_book` block: how often a named model deployment, with
a fingerprinted prompt and no retrieval at all, answers it correctly. The
headline result is always the delta between that control and the grounded
arms, reported per stratum and never pooled. A pass rate is a fact about one
model on one day, and the record says which model and which day.

**Nothing unreviewed is gold.** A record is authored as a candidate and enters
this set only after review by a person who knows the law;
`provenance.reviewed_by` names the reviewer. Candidates — including generator
output and records authored in the project's own review workbench — remain in
the instrument's candidate queue and are never scored as gold.

**Expected authorities are resolved, never written.** A `document_id` is a
fact about the corpus, resolved by tooling from the citation; an invented id
would not error — it would score every retrieval for that question as a miss
and make a working retriever look broken. The same rule applies to arithmetic:
a procedural record's expected dates are computed by the deadline engine,
never typed from memory, and *resolving is not checking* — a separate
`checked_by` field records whether a person has verified the computed date,
and the reports count the records that lack it.

**The judge is a separate instrument.** Answers are scored by a judge model
whose deployment is recorded separately from the answer arms in every report
header (`judge_deployment` beside `chat_deployment`), with a versioned,
fingerprinted prompt whose agreement with human verdicts was measured before
use. A model-comparison run moves the answer arms and leaves the judge fixed.

**The noise floor bounds every claim.** The closed-book control moved eight
points — 46% to 54% right behaviour — between two runs of the same records
with nothing changed ([`reports/eval-20260810-121621.md`](reports/eval-20260810-121621.md)).
A difference inside that floor is reported as inside it, not as a finding.
Growing the set is the standing work, and the floor shrinks as it grows.

## The published results

Reports are published here by deliberate copy from the instrument's committed
run output — publication is a reviewed act, not a sync — and each report's
header records the gold-set fingerprint, deployments, prompt fingerprints and
retrieval configuration it ran under, so any two reports state whether they
are comparable. See [`reports/`](reports/).

- **Grounding moves the same model from 46–54% to 85% right behaviour.**
  On this set, the production model grounded in the corpus showed 85% right
  behaviour against its own closed-book 46–54%
  ([`reports/eval-20260810-121621.md`](reports/eval-20260810-121621.md)).
- **Grounding the smaller model beat the larger model without retrieval by
  14 points; the larger model, grounded, led the smaller by 11.** Two runs
  differing only in the answer deployment, read together in
  [`reports/model-comparison-2026-08-24.md`](reports/model-comparison-2026-08-24.md)
  ([`eval-20260824-012947.md`](reports/eval-20260824-012947.md),
  [`eval-20260824-015436.md`](reports/eval-20260824-015436.md)). Both gaps are
  outside the noise floor. The larger model unaided also emitted more
  ungrounded citations than the smaller (123 against 67) and abstained less —
  on this set, retrieval mattered more for the more capable model, not less.

## What is and is not reproducible from here

The corpus is built from public sources and is reproducible from them: the
CourtListener bulk export (snapshot 2026-06-30; Delaware Supreme Court and
Court of Chancery opinions, filed 1983–2025) and the Delaware Code as
published at delcode.delaware.gov — about 13,300 documents at the reports'
corpus fingerprints. The `core` and `lift` strata's expected authorities
resolve against those sources.

The instrument — the retrieval stack, the evaluation harness, and the
synthetic matter corpus the `matter` stratum asks about — is a private
codebase. The reports published here are its committed output, the demo site
replays its recorded runs, and the per-question raw output (JSON beside each
report's Markdown) remains in the instrument and is published deliberately on
request. This asymmetry is stated rather than hidden: the `matter` stratum's
expected documents cannot be resolved from public sources today.

## Limitations

The set is small, and the per-stratum counts above are the honest denominator
for any rate computed from it. The noise floor bounds what the instrument can
currently adjudicate. The strata are deliberately unbalanced in purpose —
`core` for retrieval ranking, `lift` for grounding deltas, `matter` for
record-cite behaviour — and pooling them produces a number that means
nothing. Contamination figures are facts about the named deployments on the
recorded dates, not properties of the questions.

## Citing

See [`CITATION.cff`](CITATION.cff). Contributions run through the project's
review discipline; a question is welcome, and it becomes gold only after
review.

## License

[CC BY 4.0](LICENSE) — the gold set, the reports and the prose may be shared
and adapted with attribution.
