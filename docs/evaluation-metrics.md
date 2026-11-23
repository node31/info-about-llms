# Evaluating retrieval tools and LLM applications

Evaluate the retriever, the answer generator, and the complete application
separately. A good answer alone does not establish that retrieval worked;
relevant search results alone do not establish that the answer is correct.
This separation also appears in public [RAG evaluator guidance][rag-evaluators].

The scorecard and experiment design below are recommendations, not universal
industry acceptance thresholds.

## 1. Established retrieval metrics

For a query, let R be the labeled relevant items and T_k the top k results.
Choose an evaluation unit, such as a file, function, or source passage, before
calculating scores. Deduplicate results at that unit.

| Metric | Meaning | Best use |
| --- | --- | --- |
| Recall@k | Number of relevant items in T_k divided by number in R | Finding all needed evidence |
| Precision@k | Number of relevant items in T_k divided by k | Limiting irrelevant results |
| Hit rate@k | Fraction of queries with at least one relevant top-k result | Single-target lookup |
| MRR@k | Mean reciprocal rank of the first relevant result; zero if none appears within k | Getting the first useful result early |
| nDCG@k | Discounted graded relevance normalized against the ideal ranking | Ranking results with different usefulness |
| MAP | Mean of per-query average precision | Ranking multiple relevant results |

These are established information-retrieval metrics; see the
[Stanford IR textbook][ir] and [Sentence Transformers evaluator][retrieval].
Choose a few that reflect the task rather than optimizing all of them at once.
For truncated MAP, specify the cutoff and implementation.

Fictional example: a query needs four functions. The retriever returns ten
results, of which three are relevant. Recall@10 is 3/4 = 75%; Precision@10 is
3/10 = 30%. A hit occurred, but one required function was missed.

Record how you handle fewer than k results. Evaluate unanswerable questions
separately: recall has no denominator when R is empty. Recall against an
incomplete label set measures known-relevant coverage, not exhaustive coverage.

For multi-file tasks, I recommend also measuring the fraction of queries whose
retrieved evidence contains every required component. This custom
complete-evidence rate is stricter than a hit rate.

## 2. Answer-generation metrics

| Metric | Question to evaluate |
| --- | --- |
| Correctness | Is the answer right according to independently checked facts or tests? |
| Faithfulness / groundedness | Are its factual claims supported by the supplied evidence? |
| Completeness | Does it cover the required parts of the expected answer? |
| Relevance | Does it address the question rather than merely the topic? |

Groundedness, relevance, and completeness appear in
[public RAG evaluation tooling][rag-evaluators].
[Ragas faithfulness][faithfulness] evaluates supported claims divided by the
claims assessed in the response. Grounding is not independent verification of
the source's truth or freshness.

Framework definitions differ. For example, Ragas offers
[context recall][context-recall] based on supported reference-answer claims as
well as document-ID-based recall. Do not compare these as if they were the same
measurement. Pin the evaluator, version, rubric, and judge model.

For a source-backed assistant, I recommend additionally defining:

- Citation support precision: supported claim-citation links divided by the
  links assessed. A link existing is not proof of support.
- Citation coverage: factual claims requiring evidence that have a supporting
  citation, divided by all claims requiring evidence.
- Unsupported-answer rate: unanswerable test cases where the system nevertheless
  gives an unsupported substantive answer, divided by unanswerable cases.
- Answerable-task success rate: answerable cases correctly answered, divided by
  answerable cases. Track this alongside abstention to prevent "always refuse"
  from looking successful.

These are explicit scorecard definitions, not claims that all frameworks use
identical names or formulas. Define treatment of empty answers and responses
without factual claims rather than silently giving them a perfect score.

## 3. End-to-end coding outcomes

For code changes, prioritize whether the requested behavior works and existing
behavior is preserved. Build success alone is not functional correctness.
SWE-bench's evaluation similarly distinguishes resolved instances by required
test outcomes; see the [evaluation harness guide][swebench].

My recommended coding scorecard is:

- Task success on the first attempt under a fixed tool/token/time budget.
- Targeted behavioral tests passed and regression tests preserved.
- Build, type-check, and lint outcomes where applicable.
- Correct source locations and verifiable explanations for navigation tasks.
- Human intervention needed, alongside final success.

Allow valid alternative solutions. Do not require an exact tool-call sequence
unless the sequence itself is a product requirement.

## 4. Efficiency and reliability

Record retrieval and full-request latency separately, plus time to first token
for streaming and input/output token usage. OpenTelemetry's
[GenAI conventions][telemetry] describe operation and inference measurements.

For this application, I would report:

| Measurement | Reporting convention |
| --- | --- |
| Retrieval latency | p50 and p95, split by cache condition |
| Total latency | p50 and p95, including tool calls and retries |
| Token usage | Retrieved-context, full-input, and output tokens separately |
| Cost per task | Retrieval, reranking, model, and retry costs |
| Cost per successful task | Total workload cost divided by successful tasks |
| Errors and timeouts | Fractions of attempted tasks, reported separately |
| Index freshness | Source-change-to-searchable delay and stale-result rate |
| Access-control violations | Unauthorized results or answers in dedicated tests |

Count failed attempts in cost and reliability reporting. If no tasks succeed,
cost per successful task is undefined, not zero.

## 5. Code-repo-specific checks

For the [tags, documentation, and call-map design](code-knowledge-base-strategy.md),
I would add these custom diagnostics:

- Tag candidate recall: labeled relevant items surviving the tag filter divided
  by labeled relevant items. This checks whether early filtering loses evidence.
- Call-edge precision and recall on a manually verified graph sample, split
  between resolved, inferred, and unresolved edges.
- Summary fidelity and freshness, checked against the indexed source revision.
- Final evidence recall within a fixed retrieved-token budget, not only at k.

For graph recall, specify the truth set and whether you are measuring static
relationships or runtime-observed calls. An unobserved runtime edge is not
automatically nonexistent.

## 6. Build an evaluation set

As an initial pilot, I recommend 100-200 human-reviewed questions or tasks, then
expanding based on failures. This is a practical starting suggestion, not a
statistically sufficient sample size for every use case.

Include symbol lookups, conceptual queries, multi-file reasoning, bug fixes,
similar distractor names, missing information, stale summaries, and
access-restricted content. Use fictional or authorized data only.

For each case, store:

```text
Task ID and query
Repository revision and access scope
Answerable or unanswerable
Relevant file/function/passage IDs and relevance grades
Required answer claims or behavioral tests
Permitted evidence alternatives
Task category and difficulty
```

Keep a development set separate from a held-out test set. Preserve task
environment, index revision, prompts, model IDs, generation settings, and tool
budgets so comparisons are interpretable.

Use deterministic checks where possible, rubric-based model judges where useful,
and human review to calibrate judges. Repeat stochastic trials and report
uncertainty instead of treating a small score change as definitive.
These practices align with [agent evaluation guidance][agent-evals].

## 7. Diagnose where improvements are needed

My recommended controlled comparisons:

| Experiment | Hold fixed | Change |
| --- | --- | --- |
| Retriever comparison | Corpus, queries, labels, and evidence budget | Keyword, vector, hybrid, tags, reranking |
| Generator comparison | Exact evidence and tasks | Answering model or prompt |
| Oracle-evidence run | Model, task, and budget | Supply verified sufficient evidence |
| No-retrieval baseline | Model and tasks | Remove retrieved evidence |
| Component ablation | Rest of pipeline | Remove tags, summaries, or graph expansion |

An oracle-evidence gap suggests retrieval/context assembly is worth investigating;
it is not proof of a single cause. Failure even with sufficient evidence suggests
generation, task, or evaluation issues. Inspect the traces and graders.

## Recommended first dashboard

Start with Recall@10, nDCG@10, task correctness, faithfulness, complete-evidence
rate, unsupported-answer rate on unanswerable cases, p95 latency, context tokens,
and cost per successful task. Add MRR for symbol lookup.

Tune thresholds to the actual risk, workload, and baseline. Keep per-category
results and failure counts visible; do not hide them in a single blended score.

## Public tools and references

- [Stanford: evaluating ranked retrieval][ir]
- [Sentence Transformers: retrieval evaluator][retrieval]
- [Ragas: available metrics][ragas]
- [RAG evaluator definitions][rag-evaluators]
- [SWE-bench: evaluating patches][swebench]
- [Anthropic: evaluating agents][agent-evals]
- [OpenTelemetry GenAI metrics][telemetry]

[ir]: https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-ranked-retrieval-results-1.html
[retrieval]: https://www.sbert.net/docs/package_reference/sentence_transformer/evaluation.html
[ragas]: https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/
[faithfulness]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/faithfulness/
[context-recall]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/context_recall/
[rag-evaluators]: https://learn.microsoft.com/en-us/azure/foundry/concepts/evaluation-evaluators/rag-evaluators
[swebench]: https://www.swebench.com/SWE-bench/guides/evaluation/
[agent-evals]: https://www.anthropic.com/research/demystifying-evals-for-ai-agents
[telemetry]: https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md
