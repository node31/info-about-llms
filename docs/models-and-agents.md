# Multimodal models, multilingual models, and agents

These terms describe different dimensions: data types, natural languages, and
how a system controls actions. Domain specialization is another dimension, not
a subset of modality or language coverage.

## Multimodal: different kinds of information

A multimodal model can process or generate more than one modality, such as text,
images, audio, or video. A language-centered model with these capabilities is
often called a multimodal LLM.

Supported inputs and outputs need not match. Accepting an image and returning a
text explanation does not imply the model can generate images.
See [Google's multimodal overview][multimodal] and the
[Gemini technical report][gemini].

Fictional example:

```text
Input:  A chart image + "Explain the trend."
Output: A text explanation.
```

Multimodal does not mean several models are necessarily being called. It
describes the information handled, not the number of models in the application.
The Gemini report describes a model family with capabilities across modalities.

## Multilingual: different natural languages

A multilingual LLM works with multiple human languages, such as English, Hindi,
and Japanese. Translation is one use, not the whole definition; multilingual
instruction following is another.

Fictional example:

```text
Input:  An English document + a request for a Hindi summary.
Output: A Hindi text summary.
```

This example remains text-only. Different natural languages are not different
modalities. Language coverage also does not imply equal quality in every
language. See the [Aya multilingual model paper][multilingual].

## Domain-specific: specialized subject matter or tasks

A domain-specific model is developed or adapted for an area such as medicine,
law, finance, or software engineering. This does not by itself specify its
modalities, supported languages, or whether an application uses it as an agent.

Hypothetical combinations:

| Specialization | Modalities | Natural languages |
| --- | --- | --- |
| Legal document analysis | Text only | English only |
| Financial document search | Text only | English and Hindi |
| Medical image explanation | Text and images | English only |
| Medical image explanation | Text and images | Multiple languages |

These are illustrative possibilities, not claims about particular products.
A concrete example of specialization with different modality options is
MedGemma, whose public description includes both text and multimodal medical
variants. See [Google's MedGemma overview][domain-model].

Model role is also separate: a domain-specific embedding model produces
representations for search, while a domain-specific generative model produces
answers. Neither role automatically implies agentic execution.
See [embedding benchmarks](benchmarks.md).

## Agent: a system that selects actions

In the LLM-based setting, a useful architectural distinction is between a
predefined workflow and an agent whose model dynamically directs steps and tool
use. Terminology varies; a model plus retrieval is not automatically an agent.
See [Building effective agents][agents].

Fictional code-fixing example:

```text
Task: Fix a failing test.
  -> choose which code to inspect
  -> edit the relevant function
  -> run a test tool
  -> inspect the result
  -> choose whether to revise or finish
```

The model proposes actions; application code executes permitted tool calls.
Tools and a control loop make the actions possible, not multilingual or
multimodal capability alone.

## Combining the dimensions

These are illustrative combinations:

| System | Multimodal? | Multilingual? | Agentic? |
| --- | --- | --- | --- |
| Text-only translator | No | Yes | Not necessarily |
| Image question-answering model used once | Yes | Depends on language support | Not necessarily |
| Text-only code-fixing agent | Not required | Not required | Yes |
| Agent that inspects screenshots and answers in several languages | Yes | Yes | Yes |

For the [repository knowledge-base proposal](code-knowledge-base-strategy.md),
semantic retrieval can supply evidence to either a fixed answer pipeline or an
agent. Image understanding could support diagram or screenshot questions;
multilingual capability could support queries and explanations across languages.
Treat these as independent design choices and evaluate each on the actual task.

[multimodal]: https://cloud.google.com/use-cases/multimodal-ai
[gemini]: https://arxiv.org/abs/2312.11805
[multilingual]: https://arxiv.org/abs/2402.07827
[agents]: https://www.anthropic.com/research/building-effective-agents
[domain-model]: https://research.google/blog/medgemma-our-most-capable-open-models-for-health-ai-development/
