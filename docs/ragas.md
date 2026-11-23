# What is Ragas?

The original paper expands RAGAs as **Retrieval Augmented Generation
Assessment**. Ragas is an open-source evaluation framework, initially focused
on RAG pipelines, with metrics now covering additional LLM application and
agent tasks. See the [original paper][paper] and [metric catalog][metrics].

It is not a language model, embedding model, or a single fixed test dataset.
It supplies evaluation machinery that can be used on application-specific data.

## What goes into an evaluation?

A RAG evaluation typically records:

```text
Question
Retrieved passages
Generated answer
Reference answer or reference passages, where required
```

Different metrics need different fields. Some do not require a reference answer;
others do. Reference-free evaluation still needs evaluation inputs and a scoring
method. It does not create independent ground truth.
See [faithfulness][faithfulness] and [context recall][recall].

## Common metrics

| Metric | What it examines |
| --- | --- |
| Context precision | Whether relevant retrieved chunks rank ahead of irrelevant chunks |
| Context recall | Whether retrieved evidence covers the information expected in a reference |
| Faithfulness | Whether answer claims are supported by the retrieved context |
| Response relevancy | Whether the response addresses the question |
| Factual correctness | Whether the answer's claims match a reference answer |

The [catalog][metrics] lists these metrics. Their implementations are not all
the same: [context precision][precision] has ranking-based and ID-based options;
[context recall][recall] can assess reference-answer claims or reference IDs.
Specify the exact variant instead of assuming equivalence to ordinary
Precision@k or Recall@k.

[Factual correctness][correctness] compares a response with a reference through
claim analysis. It is not an independent fact-check of the reference itself.

## A fictional code example

```text
Question: Where are expired sessions rejected?

Retrieved evidence:
  reject_expired_session checks expires_at.

Generated answer:
  reject_expired_session checks expires_at and sends an email.
```

The first answer claim is supported; the email claim is not. A faithfulness
evaluation should flag the unsupported addition. Exact scores depend on claim
decomposition and evaluator behavior, not a guarantee of this example.

## How scoring works

Many Ragas metrics use an LLM as a judge, and some use embeddings or deterministic
comparisons. It also supports custom metrics.
See the [metric catalog][metrics].

For example, faithfulness identifies answer claims and estimates the fraction
supported by retrieved context. A high score means evidence consistency, not
proof that the source is correct or current.
See [faithfulness calculation][faithfulness].

My recommended safeguards are to calibrate automated judgments against human
reviews, pin evaluator versions and judge settings, record evaluation costs,
and keep behavioral tests for code changes. A judge-model score is not a
substitute for executing the relevant tests.

## Relationship to other evaluation tools

- MTEB and RTEB provide benchmark tasks for evaluating embeddings.
- SWE-bench tests repository issue resolution using generated patches.
- Ragas provides metrics and evaluation tooling for an application's runs.

See [benchmark comparison](benchmarks.md) and
[evaluation metrics](evaluation-metrics.md).

For this repository-assistant design, I recommend using embedding benchmarks
to shortlist models, labeled repository queries to test retrieval, Ragas to
diagnose evidence use and answer quality, and execution tests to assess patches.

[paper]: https://aclanthology.org/2024.eacl-demo.16/
[metrics]: https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/
[precision]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/context_precision/
[recall]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/context_recall/
[faithfulness]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/faithfulness/
[correctness]: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/factual_correctness/
