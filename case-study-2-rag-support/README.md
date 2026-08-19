# Case study 2 — RAG support assistant

**Status: in progress.**

Can a retrieval system answer support questions from six internal documents — with a source
reference — *and* reliably say "that isn't in here" when it isn't?

The evaluation set ([`../data/eval_questions_rag.csv`](../data/eval_questions_rag.csv)) contains
25 questions with known answers, deliberately split into five types:

| Type | n | What it tests |
|---|---|---|
| `fakt` | 14 | Correct figure, correct source |
| `luecke` | 5 | **Must refuse.** The answer is in none of the documents. |
| `mehrschritt` | 3 | Combining two documents, or arithmetic |
| `falle` | 1 | Sounds answerable, isn't |
| `widerspruch` | 2 | The documents contradict each other; a good answer names both sides |

Two metrics are tracked separately and never blended: **hit rate** on the 18 answerable questions,
and **refusal rate** on the 5 gaps. A system with 95 % hit rate and 0 % refusal rate is dangerous,
not successful. The curve between the two across runs is the case study.
