# Published reports

Reports arrive here by deliberate copy from the instrument's committed run
output — publication is a reviewed act, not a sync, and nothing automates the
copy. A report may stand alone; an essay may accompany one in
[`../essays/`](../essays/), beside its evidence.

Every report's header records the gold-set fingerprint, the answer and judge
deployments, the prompt fingerprints and the retrieval configuration it ran
under. Two reports are comparable when their headers say so, and not
otherwise.

Each Markdown report summarises a run whose per-question raw output (a JSON
file beside it in the instrument) records every answer, verdict and retrieval
trace. The raw files remain in the instrument and are published deliberately
on request.

| Report | What it carries |
| --- | --- |
| [`eval-20260810-121621.md`](eval-20260810-121621.md) | The post-migration baseline: grounded 85% right behaviour against 46–54% closed-book on the same model — and the noise floor's own measurement, the control moving eight points between two runs of the same records. |
| [`eval-20260824-012947.md`](eval-20260824-012947.md) | The model-comparison run on the larger deployment (answer arms moved; judge fixed). |
| [`eval-20260824-015436.md`](eval-20260824-015436.md) | The same run on the production deployment. |
| [`model-comparison-2026-08-24.md`](model-comparison-2026-08-24.md) | The two runs read together: grounding the smaller model beat the larger unaided by 14 points; the larger grounded led by 11; both outside the floor. |
