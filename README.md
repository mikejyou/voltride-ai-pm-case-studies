# VoltRide — RAG support assistant over internal documents

A portfolio case study on **grounding and justified refusal**: when may a retrieval-augmented
system answer, when must it stay silent, and what does each of the two failure directions cost?

> **All data is invented.** VoltRide is a fictional company; the six documents and the 25
> evaluation questions are synthetic. No connection to any real employer. The upside: the
> dataset carries ground-truth labels, which makes quality measurable rather than asserted.
>
> The document corpus is in German, and so are the prompts and the model's answers. That is
> deliberate — English-language retrieval is the easy case, and the central finding below is
> partly a consequence of running an English-trained embedding model on German text. The
> analysis, this README and the case study are in English.

**→ [CASE_STUDY.md](CASE_STUDY.md)** — the write-up, 3–4 minutes.

## The result in three lines

- Of 13 questions where the supporting passage was demonstrably in context, **12 were
  answered correctly**. Every genuine failure happened one step earlier, during retrieval.
- Three increasingly strict refusal rules moved accuracy **not at all**. The refusal rule is
  insurance against retrieval failures, not a quality lever.
- My own automated grader was wrong on **4 of 17** questions — 24 percentage points of
  measurement error, produced by the instrument rather than the system.

![Grounding vs. refusal](images/tradeoff_grounding_vs_refusal.png)

## Repository layout

| Path | Contents |
|---|---|
| `notebook/` | `CaseStudy2_RAG_Support.ipynb` — runs top to bottom, 11 cells |
| `daten/docs/` | the six fictional company documents (German) |
| `daten/eval_fragen_rag.csv` | 25 evaluation questions with known answers |
| `ergebnisse/` | raw results of the three runs, metrics, retrieval diagnostics, manual coding |
| `images/` | the figure used in the case study |

Folder and column names inside the notebook and the result files are German, matching the
corpus. `CASE_STUDY.md` explains every metric in English.

## Running it yourself

1. Open the notebook in Google Colab (or locally with Jupyter).
2. Get a free API key at [console.groq.com/keys](https://console.groq.com/keys) and store it
   as the Colab secret `GROQ_API_KEY`.
3. Upload the seven files from `daten/` — folder structure does not matter, cell 2 finds them.
4. **Cells 1–5 make no API calls at all.** They contain the complete retrieval diagnostic,
   including a regression warning that compares against the previous configuration. Only
   cell 9 talks to the model.

Three full runs over 25 questions cost about $0.01. Embeddings run locally and are free.

## How quality is measured

Two metrics, never merged into one:

- **Accuracy** on the 17 answerable questions (14 factual + 3 multi-hop)
- **Refusal rate** on the 5 genuine gaps

Measured separately: **evidence recall** — was the supporting passage present in the retrieved
context at all? If it wasn't, a wrong answer is not a model error. Measuring this at document
level rather than section level flattered the system by 12 percentage points.

The automated grade is a **proposal, not a verdict**. `ergebnisse/handcodierung_basis.csv`
holds the manual coding in `korrekt_manuell`; `korrekt_final` combines both. The cache stores
successful calls only — cached failures would silently corrupt the measurements.

## Stack

Python · sentence-transformers (`all-MiniLM-L6-v2`, local) · Groq `openai/gpt-oss-20b` · pandas · matplotlib
