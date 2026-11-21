# What does a million-token context window mean?

A context window is the amount of information a model can work with during
generation. Think of a working desk, not a permanent archive.
See [context windows][windows].

## Tokens are not words

A token can represent a word, part of a word, or punctuation. As a rough English
estimate, one million tokens can represent around 600,000 to 800,000 words.
The actual count depends on the tokenizer, language, and content. Code and
non-text inputs need their own accounting. Use the provider's token-counting
tools rather than this estimate for a real request.
See [Google's token guide][tokens].

## What takes up the space?

Instructions, selected conversation history, the current question, document
excerpts, tool definitions, and tool results can all consume context. Generated
output and reasoning tokens also consume budget according to the particular
model's rules. Providers may describe input and combined limits differently.
See [provider accounting details][windows].

This is an illustrative budget, not a real model configuration:

```text
Instructions                  5,000 tokens
Conversation history         95,000 tokens
Retrieved documents         850,000 tokens
Current question              1,000 tokens
-----------------------------------------
Input total                 951,000 tokens
```

If a model's combined input-and-output limit were exactly 1,000,000 tokens,
that example would leave at most 49,000 tokens for output and any other counted
material. A separate output limit could make the actual allowance smaller.

## What it does not mean

A large window does not mean the model permanently learns the supplied documents,
can produce equally large outputs, or will reliably use every detail. More
context can make finding and combining relevant information harder.
See [context management and retrieval][engineering].

An application can persist messages or summaries outside the model and make them
available in later requests. That is application-managed memory, not a weight
update.

For knowledge-base applications, use the larger window as capacity for useful
evidence, not a reason to include every document indiscriminately.

[windows]: https://platform.claude.com/docs/en/build-with-claude/context-windows
[tokens]: https://ai.google.dev/gemini-api/docs/tokens
[engineering]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
