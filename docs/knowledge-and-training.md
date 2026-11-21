# Knowledge, training, and retrieval

## Does every request contain the entire conversation?

Not necessarily. The application or service assembles the context available to
the model. It can include selected messages, summaries, retrieved documents, and
tool results. Keeping information in context is not the same as training on it.
See [context engineering][context].

## Option 1: Change what the model has learned

Training adjusts learned parameters. Fine-tuning adapts a previously trained
model to a narrower task or domain. Some approaches train additional adapter
parameters rather than changing all of the base model's weights.

```text
Base model + training examples
    -> training
    -> saved customized model or adapter

Later:
Load base model and saved customization + question
    -> answer
```

Saving and reloading the trained artifacts preserves the customization; merely
restarting a chat does not create it. Fine-tuning can improve task behavior, but
is not a guarantee of exact factual recall. See [fine-tuning and RAG][augmentation]
and [LoRA adapters][lora].

## Option 2: Supply knowledge when answering

Instead of changing model parameters, store documents outside the model and
retrieve useful passages for each question. This is retrieval-augmented
generation, or RAG.

```text
Documents -> passages -> searchable index

Question -> search -> relevant passages
         -> model receives question and evidence
         -> answer
```

You generally do not need to send the entire knowledge base. The document store
persists separately, while selected evidence enters the current context.
Retrieval can use keyword search, vector search, or a combination.
See the [original RAG paper][rag] and [public implementation overview][augmentation].

## Choosing an approach

As a starting design heuristic:

| Need | Starting point |
| --- | --- |
| Current facts with inspectable sources | Retrieval |
| Specialized response patterns or task behavior | Prompting, then evaluate fine-tuning |
| Both specialized behavior and current facts | Combine customization and retrieval |

Instructions are a legitimate way to specify a task, but they do not themselves
update learned parameters. Conversely, training does not remove the need to
specify the current task.

A practical starting architecture is a standard model plus retrieval, with
evaluation questions covering missing evidence as well as answerable questions.
Add fine-tuning only when measured failures justify it.

[context]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
[augmentation]: https://learn.microsoft.com/en-us/azure/developer/ai/augment-llm-rag-fine-tuning
[lora]: https://huggingface.co/docs/peft/main/en/conceptual_guides/lora
[rag]: https://arxiv.org/abs/2005.11401
