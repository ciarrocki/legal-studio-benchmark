# The gold-record format

One JSON object per line of [`gold/gold-set.jsonl`](gold/gold-set.jsonl).
This document is the standalone statement of the format the instrument's
`GoldRecord` type defines; the two move together.

## Fields

| Field | Notes |
| --- | --- |
| `id` | Unique. Keys screening results back onto the record. |
| `question` | What gets asked. Must not contain its own answer. |
| `stratum` | `core`, `lift`, `matter`, or `procedural`. Strata are never pooled in a report. |
| `generator` | Where the question came from: `hand`, or a named generator (`obscurity`, `statute-diff`, `recent-opinion`, `novelty`, `matter`). |
| `authority_type` | `opinion`, `statute`, `both`, `none`. |
| `expected[]` | The expected authorities: `document_id`, `citation`, and `weight` (3 controlling, 2 supporting, 1 relevant). |
| `reference_answer` | What a correct answer *says*. It states the law; it never addresses the grader. |
| `judge_notes` | Optional scoring guidance sent **to the judge** as a labelled block: alternative acceptable shapes, and plausible-looking answers that are not correct. |
| `notes` | Why the expected authority controls; reviewer instructions. For the human; never sent to a model. |
| `answerable` | False for the abstention records. |
| `unanswerable_kind` | Only when `answerable` is false; one of the four kinds below. |
| `predicted_closed_book` | The author's guess — `pass` / `fail` / `unsure` — recorded **before** the closed-book screen, so the first screen scores the premise of the exercise and not only the questions. |
| `closed_book` | Written by the screening run: the deployment, the prompt fingerprint, the date, the sample count and the measured pass rate. Null until measured. |
| `expected_computation` | Only on a `procedural` record: the jurisdiction, court, trigger event and date, the service method — and the dates that follow, computed and never typed. |
| `provenance` | Where the record came from and **who reviewed it**. A record without a named reviewer is a candidate, not gold. |

**Three fields, three audiences.** `reference_answer` is prose a correct
answer resembles; the moment it starts saying "the correct response is to…"
it is addressing the grader instead. Grading guidance belongs in
`judge_notes`, which the judge receives as a separate labelled block; `notes`
is for the human reviewer and reaches no model at all.

`weight` exists so a rank-sensitive metric is possible: a flat list of
expected ids scores a result set identically whether the controlling holding
or a case discussing it ranked first.

## The four unanswerable kinds

Unanswerable records live in whichever stratum they belong to and carry
`answerable: false` plus a kind. The kinds fail for different reasons and are
scored apart, because a system can get one right and the others wrong:

- **`wrong-jurisdiction`** — another sovereign's law. Deliberately "sovereign,
  not geographic": a question about federal law is neither Delaware's nor
  another state's, and it is still the corpus being asked about the wrong body
  of law.
- **`absent-authority`** — Delaware, but a subject the corpus does not cover.
- **`outside-date-window`** — right jurisdiction, right subject, authority
  filed after the corpus's pinned snapshot.
- **`absent-from-record`** — nothing to do with the law. The question is about
  what a matter file establishes, and the evidentiary record does not
  establish it: witnesses disagree, the only person who knew was never
  deposed, no document was ever created. A system that correctly abstains on
  missing law can still invent a fact, and this kind exists to catch that
  failure.

## Two resolution rules

**Document ids are resolved, never written.** A `document_id` is a fact about
the corpus. Records are authored with the citation and the literal string
`"unresolved"`, then resolved by the instrument's tooling; the loader refuses
to score a set that still contains one. An invented id would not error — it
would score every retrieval for that question as a miss and make a working
retriever look broken. Statute ids are the exception and are derivable:
`8 Del. C. § 102(b)(7)` lives in the document for its section, because the
ingest emits one document per section and subsection lettering is inside the
text.

**Expected dates are computed, never typed.** The same rule applied to
arithmetic: a due date typed from memory does not error either — it scores
every arm wrong on a question the calculator answers correctly. Procedural
records are authored with the ask and the rule ids, every `due` left
`"unresolved"`, and resolved by the deadline engine the product runs.
**Resolving is not checking**: `checked_by` is a separate field the resolver
never writes, and the reports count the records that lack it — until a person
has read the computed date against the rule book, these rows measure
agreement with the calculator, not correctness.

## Contamination is a measured field

Every record's `closed_book` block records how often a named model deployment,
under a fingerprinted prompt and no retrieval, answers the question correctly.
Two consequences worth stating plainly: retrieval metrics do not care about
contamination (they only need the controlling authority to be unambiguous, and
optimising the whole set for closed-book difficulty would make them *less*
representative — which is why the strata exist); and a pass rate is a fact
about one model on one day, re-measured when the deployment changes. A set
that resisted last year's model is not evidence about this year's.

The best `lift` records are ones where the closed-book prior is *actively
wrong*, not merely absent — a superseded doctrine, an amended statute, a
threshold the model interpolates. A confidently wrong control produces a clean
signal and a legally meaningful one.
