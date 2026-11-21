# Why do LLMs hallucinate?

Generating a convincing answer and establishing that it is true are different
tasks. Here, a hallucination means a fabricated or unsupported claim presented
as fact.

## Several mechanisms contribute

**Prediction is not verification.** Pretraining teaches models to predict tokens
from context. This develops useful factual and reasoning capabilities, but does
not make every generated claim correct.

**Incentives can favor guessing.** If evaluations award no credit both for a
wrong answer and for admitting uncertainty, guessing can improve a model's
measured accuracy while increasing false answers. Training and evaluation
incentives are one contributor, not a complete explanation of every error.
See [research on hallucination incentives][incentives].

**Uncertainty recognition can fail.** In a study of one model, recognizing a
familiar entity could activate answering behavior without the specific requested
fact being available. This is an observed mechanism, not a universal account of
all models.

**Reasoning can go wrong too.** The same research documents cases of plausible
reasoning being generated in response to misleading hints. Missing knowledge
is therefore not the only failure mode.
See [Anthropic's interpretability research][tracing].

## A fictional example

Available document:

> Project Cedar is owned by Team Blue.

Question:

> Who approved Cedar's budget?

Supported answer:

> This document identifies the owner, but not the budget approver.

Unsupported answer:

> Team Blue's director approved the budget.

Ownership does not establish approval. The second answer fills an evidence gap
with a plausible story.

## Designing a more reliable knowledge assistant

Recommended checks:

- Retrieve evidence relevant to the specific question.
- Require citations and check that they actually support the claims.
- Explicitly allow "the sources do not establish this."
- Test missing-information and misleading-question cases.
- Measure unsupported claims as well as correct answers.

These are engineering recommendations, not guarantees. Retrieval supplies
evidence, but a system still needs evaluation of both retrieval quality and
answer quality. See [RAG design and evaluation][evaluation].

A larger context window or a fine-tuned model should not be treated as proof
that an answer is true.

[incentives]: https://openai.com/index/why-language-models-hallucinate/
[tracing]: https://www.anthropic.com/research/tracing-thoughts-language-model
[evaluation]: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide
