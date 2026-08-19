# VoltRide — AI Product Management Case Studies

Two hands-on case studies on building and **evaluating** LLM features, written from a product
manager's perspective rather than an engineer's. The focus is not "can this be built" — it can —
but "how do I know whether it is good enough to ship, and what does it cost to run".

> ### All data here is fictional
> VoltRide is an invented e-bike sharing company. The feedback corpus, the internal documents and
> the evaluation questions were purpose-built for this portfolio. **No employer data is used.**
>
> That constraint turned into an advantage: a purpose-built dataset carries ground-truth labels,
> which makes quality measurable rather than asserted.

## The case studies

| # | Question | Outcome |
|---|---|---|
| [**1 — Feedback to roadmap**](case-study-1-feedback-to-roadmap/) | Can an LLM turn 2,500 monthly free-text responses into a prioritized backlog? | 73 % topic accuracy. Precision on safety alerts was fixable by prompting; recall was not — one report in three stayed undetected in every variant. Not shippable, and the write-up explains why. |
| [**2 — RAG support assistant**](case-study-2-rag-support/) | Can a retrieval system answer from six internal documents *and* admit when the answer isn't there? | In progress |

## What's in here

```
data/
  feedback_voltride.csv     60 labelled feedback items (ground truth for case study 1)
  eval_questions_rag.csv    25 evaluation questions with known answers (case study 2)
  docs/                     six fictional internal documents (knowledge base for case study 2)
case-study-1-feedback-to-roadmap/
  README.md                          the full write-up
  notebook.ipynb                     the complete run, with outputs
  error_analysis_v1.csv              the hand-coded error analysis
  fig1_metrics_by_run.png            all metrics across the four runs
  fig2_safety_precision_recall.png   the precision/recall trade-off
```

## A note on language

The work was carried out in German. The feedback corpus deliberately mixes German and English,
prompts were written in German, and category names were English. **The write-ups are in English;
the data artefacts are not translated** — retranslating them would invalidate the measurements,
and one of the findings in case study 1 depends on the German/English collision.

## Method

Each case study follows the same discipline:

- a baseline run, measured before anything is changed
- **one change per run**, with the expectation written down beforehand
- error analysis by hand — open coding, then axial coding — never delegated to a model
- failed calls are never disguised as predictions; a success-rate column runs alongside every metric
- precision and recall reported together, never a single blended accuracy figure
- a cost model built from measured token counts, not estimates

## Stack

Python · Groq (`openai/gpt-oss-20b`) · `sentence-transformers` · pandas · matplotlib
Everything runs on free API tiers. Total inference cost across all runs: well under one euro.

## Author

Michael Seidou — product manager, working towards AI product roles.
Website: [michaelseidou.com](https://michaelseidou.com)
