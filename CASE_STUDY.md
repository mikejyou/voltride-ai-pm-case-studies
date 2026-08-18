# Support answers from six internal documents — with a source, and the nerve to stay silent

For VoltRide, a fictional e-bike sharing service, I built an assistant that answers Level-1 support questions from six internal documents, cites a source for every claim, and refuses when the answer is not documented — and I measured what each of the two failure directions costs.

`Duration: ~10 hours` · `Role: concept, build, evaluation` · `Stack: Python, sentence-transformers (local), Groq / gpt-oss-20b, pandas`

> **A note on the data**
> This case study runs on a fictional company and synthetic data. That was deliberate:
> portfolio work has to be public, my employer's data is not. The upside of a synthetic
> corpus is ground-truth labels — quality becomes measurable rather than asserted.
>
> The corpus, the prompts and the system's answers are in German. That is part of the
> design: English retrieval is the easy case, and one finding below only surfaces on
> non-English text.

---

## 1. The problem

Level-1 support answers questions about prices, deadlines, refund limits and service areas all day. The answers live scattered across six internal documents, in places contradictory.

The two failure directions are not equally expensive. A fabricated answer — "you can refund up to €60 yourself" — reaches a paying customer and becomes a commitment. An unnecessary refusal costs an escalation. At 200 requests a day, the first kind is the expensive one.

## 2. Why conventional software doesn't solve this

Full-text search finds documents, not answers. "What does a 20-minute ride cost on the Basic plan?" appears nowhere — the number only exists once you add the unlock fee to the per-minute rate. A language model can do that. What it cannot do on its own is tell *stated in the document* from *sounds plausible*. That distinction is the actual product work.

## 3. The approach

Split documents into sections at their headings → embed each section locally as a vector → retrieve the three most similar sections per question → let the model answer **only** from those sections, in a fixed format with fields for status, answer, sources and conflict.

**Rejected, with reasons:**

- **A model as the grader (LLM-as-a-judge).** More convenient, but the grader is itself a source of error I would have had to validate against the same labels. Instead: a visible table of check patterns, one per question, stating what counts as correct.
- **A bigger model.** The obvious move, and the wrong one — the model was never the bottleneck.
- **A single combined quality metric.** A system at 95 % accuracy and 0 % refusal rate is dangerous, not successful. The two numbers are never merged.

## 4. How I measured quality

**The test set:** 25 questions with known answers — 14 factual, 3 multi-hop, **5 genuine gaps that must be refused**, 2 documented contradictions, 1 trap question that sounds answerable but isn't.

**Three metrics, kept apart:** accuracy on the 17 answerable questions. Refusal rate on the 5 gaps. And — the turning point — **evidence recall**: was the supporting passage even present in the retrieved context?

Three runs, one intervention each, changing nothing but the refusal rule in the prompt:

| Run | Accuracy (n=17) | Refusal rate (n=5) | False refusals | Evidence recall |
|---|---|---|---|---|
| `baseline` | 76 % | 80 % | 18 % | 74 % |
| `strict` | 71 % | 80 % | 18 % | 74 % |
| `very strict` | 71 % | **100 %** | 24 % | 76 % |

**The central finding:** all three runs fail on the same four questions. Retrieval does not depend on the prompt, so it was identical across all runs — I had turned a dial three times that was never the problem.

**Of the 13 questions where the evidence was demonstrably in context, 12 were answered correctly.** The one exception is a false positive in my evidence check, not a model error. Every genuine failure happened one step earlier, during retrieval.

Tightening the rule bought exactly one thing: the project's only hallucination became a refusal. I paid with a useful partial answer ("Aarburg, price not stated") that turned into a flat "not in the documents". **The refusal rule is insurance against retrieval failures, not a quality lever.**

### Error classes

| Class | Frequency | Cause | Countermeasure | Effect |
|---|---|---|---|---|
| `evidence_not_retrieved` | 4 of 17 | wrong section returned; the city table is **one** vector in which each individual city disappears | split tables row by row | **regression, see §5** |
| `checker_too_strict` | 4 of 17 | patterns demanded the wording of the reference answer, not its substance | separated "wrong" from "incomplete" | 53 % → 76 % |
| `metric_distorted` | 1 metric | refusals counted as incorrect source attribution | count answered rows only | 68 % → 100 % |
| `hallucination` | 1 of 25 | evidence missing, the model filled the gap | stricter refusal rule | works, but expensive |

**Before/after:** my automated checker first reported 76 % accuracy as 53 %. **24 percentage points of measurement error — produced by the instrument, not the system.** Example: "up to €25.00 independently, €60.00 requires team lead approval" was scored wrong because the word "no" was missing.

## 5. What didn't work

**The intervention that netted nothing.** Splitting the city table row by row fixed three questions and broke three others. Evidence recall before and after: 68 %. The cause was mine: I repeat the table header in every row so "620" isn't context-free — which put the words "service area" into six short sections, where they outranked the one correct section for "what does parking outside the service area cost?". Visible only because I compared the *composition* of failures, not the percentage.

**The gap between 76 % and 71 % was not a model difference at all.** The model gave substantively the same answer to Q07 in all three runs. Only the first run had my manual verdict attached. A label that exists for one of three runs produces a difference that looks like a result.

**The most expensive item was the tooling, not the model.** The notebook had to be rebuilt six times — missing line breaks in the file format, a folder upload that never completed, the wrong CSV delimiter, and twice, cells that referenced variables from earlier cells and crashed after a runtime restart. Only two of those rounds produced anything substantive. I worked with an AI assistant as a sparring partner, and a sizeable share of the effort went into correction loops unrelated to the actual problem.

**Rate limits.** The free tier aborted four calls mid-run, and my own code truncated the error message to 110 characters — cutting off exactly the part that mattered. I had to build a diagnostic cell that reads the counters straight from the response headers. It revealed that the limit counts the **reserved** output budget: a test call with 16 tokens passes while the same call with 1,600 tokens is rejected. A test call that is too small reports "all clear" when nothing is clear.

Across several sessions, two things saved the work: a cache that stores **successful calls only** — cached failures would have silently produced wrong measurements — and one line that archives the state before every disconnect:

```python
import shutil
shutil.make_archive('cs2_arbeit', 'zip', '/content/cs2_arbeit')
```

## 6. Cost and operations

Three complete runs over 25 questions: **$0.0094** (60,122 input, 6,773 output tokens). Extrapolated to 200 support questions a day: **roughly $0.75 a month**. Embeddings run locally and cost nothing.

A bigger model would be the wrong investment: the model answered correctly everything it could see. The operating cost sits elsewhere — maintaining the documents, and reworking the cases where retrieval returns the wrong section.

## 7. What I would do differently

**Measure retrieval first.** Evidence recall costs zero model calls and would have saved three prompt runs. I only built it after the third.

**Take the instrument as seriously as the system.** Two of my metrics were wrong before either said anything about the system. A percentage without the distribution behind it is an assertion.

**Stick to the structure the document gives you.** My table intervention was well-intentioned and created a new problem that would have stayed invisible without a before/after comparison.

---

*Code, notebook with executed outputs, evaluation set and all raw results: https://github.com/mikejyou/voltride-rag-case-study*
