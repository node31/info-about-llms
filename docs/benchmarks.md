# SWE-bench, MTEB, and RTEB

These are evaluation benchmarks, not models or agents. They test different
parts of a system.

| Benchmark | Name / scope | Main question |
| --- | --- | --- |
| SWE-bench | Software engineering benchmark | Can the coding system resolve repository issues? |
| MTEB | Massive Text Embedding Benchmark | How useful are embeddings across different tasks? |
| RTEB | Retrieval Embedding Benchmark | How well do embeddings support retrieval? |

See the [SWE-bench paper][swe-paper], [MTEB paper][mteb-paper], and
[RTEB introduction][rteb].

## SWE-bench: fixing real software issues

The benchmark supplies a repository at a particular revision and a real GitHub
issue. The evaluated system generates a code patch, which is checked using an
execution-based evaluation harness.

```text
Repository + issue description
  -> coding model or agent setup
  -> proposed patch
  -> required tests
  -> resolved or unresolved
```

The central outcome is the fraction of benchmark instances resolved under the
chosen evaluation protocol. Record the evaluated instance set and treatment of
errors or unattempted tasks. See the [official FAQ][swe-faq].

SWE-bench is a benchmark, while SWE-agent is an example of a system that can
attempt such tasks. Do not confuse the exam with the participant.

The benchmark has several variants, including Verified and Lite. Verified uses
human-validated problems. Scores need a variant, split, model, tools, retry
policy, and resource budget to be interpretable. A system-level score is not
necessarily an isolated measurement of the base model.
See the [official project][swe-project].

Tests establish performance under the harness, not a proof of correctness for
all possible inputs. Nor does an issue-resolution score isolate retrieval
quality from reasoning, patch generation, and tool use.

## MTEB: evaluating embedding representations

MTEB evaluates embeddings across tasks such as retrieval, classification,
clustering, semantic similarity, and reranking. It is not primarily an exam
asking a generative model to produce a correct prose answer.
See the [original paper][mteb-paper] and [benchmark catalog][mteb-catalog].

Fictional example:

```text
Query: "Where are expired credentials rejected?"
Candidate functions:
  reject_expired_credentials
  render_login_page

Evaluate whether the relevant function ranks above the distractor.
```

That illustrates retrieval, only one of the capabilities MTEB evaluates.
Different task types use different measurements. An overall embedding benchmark
score should not be read as a percentage of arbitrary questions answered
correctly.

## RTEB: a retrieval-focused benchmark

RTEB focuses on finding relevant documents or passages for queries. Its
introduction describes open and private evaluation data intended to assess
generalization beyond repeatedly reused public test sets.
See the [maintainers' announcement][rteb].

The official catalog includes code, law, and healthcare subsets, alongside
language-specific choices. Check the exact benchmark definition and version:
an embedding model can perform differently across domains and languages.
See the [current benchmark catalog][mteb-catalog].

RTEB is presented within the MTEB evaluation ecosystem. It is a more
retrieval-focused lens, not a replacement name for the entire multi-task suite.

## Choosing for a repository assistant

My recommended sequence:

1. Use relevant MTEB and RTEB results to shortlist embedding models.
2. Compare them on held-out questions and relevance labels from the actual
   authorized repository, including exact identifiers and paraphrased questions.
3. Measure the complete retriever, including chunking, tags, filters, and
   reranking, rather than assigning its quality the embedding model's score.
4. Evaluate code-changing behavior separately with SWE-bench or an appropriately
   controlled repository-specific issue suite.

These are engineering recommendations, not guaranteed model-selection rules.
Never infer that a high general embedding score guarantees strong code search,
or that a high coding score guarantees reliable retrieval in a new codebase.

For score definitions and experiment design, see
[evaluation metrics](evaluation-metrics.md). For the independent dimensions of
domain, modality, language, and agent behavior, see
[models and agents](models-and-agents.md).

[swe-paper]: https://arxiv.org/abs/2310.06770
[swe-faq]: https://www.swebench.com/SWE-bench/faq/
[swe-project]: https://github.com/SWE-bench/SWE-bench
[mteb-paper]: https://arxiv.org/abs/2210.07316
[mteb-catalog]: https://docs.mteb.org/overview/available_benchmarks/
[rteb]: https://huggingface.co/blog/rteb
