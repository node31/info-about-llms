# Info about LLMs

Plain-language notes on how large language models use knowledge, context, and
evidence.

## Topics

1. [Knowledge, training, and retrieval](docs/knowledge-and-training.md):
   what persists across restarts, and when to use fine-tuning or RAG.
2. [What a million-token context window means](docs/context-windows.md):
   capacity, token budgets, and the difference between context and memory.
3. [Why LLMs hallucinate](docs/hallucinations.md):
   plausible answers versus supported answers, and practical mitigations.
4. [A documentation-first code knowledge base](docs/code-knowledge-base-strategy.md):
   a proposed tags-first retrieval strategy with call maps and progressive access
   to source code.
5. [What GPT stands for](docs/what-gpt-means.md):
   Generative Pre-trained Transformer.
6. [Repository embeddings and semantic search](docs/repository-embeddings.md):
   persistent indexes, similarity versus dependencies, and retrieving evidence
   without filling the prompt with vectors.
7. [Evaluating retrieval tools and LLM applications](docs/evaluation-metrics.md):
   established metrics, a code-repo scorecard, and controlled experiments that
   separate retrieval failures from generation failures.
8. [Multimodal models, multilingual models, and agents](docs/models-and-agents.md):
   input/output capabilities, language coverage, and the difference between a
   model and an action-taking system.

## Reading guide

Think of learned parameters as a model's acquired capabilities and knowledge,
context as its working material, and retrieval as looking up external evidence.
These mechanisms can be combined; they are not interchangeable.

The notes are introductory explanations, not promises about any particular model.
Provider limits and API behavior change: consult the linked primary sources for
the model you actually use.

## Publication boundary

This repository contains newly written educational summaries based on public
research and public documentation, plus a general architecture proposal.
Examples are fictional. It contains no
conversation transcripts, workplace documents, proprietary code, credentials,
or local environment configuration.
