# A documentation-first code knowledge base

## Proposed strategy

Build a navigable map of a codebase so the model can understand the relevant
parts without loading every source file into context.

The proposed discovery order is:

```text
Task
  -> tag catalog
  -> matching file summaries
  -> relevant function call maps
  -> linked summaries and module context
  -> targeted source excerpts
  -> implementation or evidence-backed answer
```

This is a design proposal, not an implemented indexer or search tool.

## 1. Tag each code file

Maintain tags describing the responsibilities, concepts, and data handled by
each file. Tags can live in an external catalog rather than modifying source.

Fictional example:

```text
Source: src/payments/validator.py
Tags: payments, validation, currency, input-checking
Symbols: validate_payment, validate_currency
Summary: docs/files/src/payments/validator.py.md
Module: payments
```

Use a controlled vocabulary and aliases so related questions can find the same
files. A file can have several responsibilities and therefore several tags.

## 2. Maintain function call maps at multiple levels

The intended navigation covers functions, files, folders, modules, and the
codebase as a whole.

Store function-level caller-to-callee edges once. Derive higher-level views by
grouping those edges by file, folder, and module instead of maintaining separate,
potentially contradictory copies.

Each edge should include:

- Caller and callee symbol identities, paths, and source locations.
- The source revision and extraction method.
- Whether the target was resolved, inferred, or remains unknown.

Use stable, qualified symbol identities within an index revision, not just
function names. Overloads and identical names in different modules must not
collapse into one node.

Treat completeness as an explicit requirement to measure, not a promise.
Callbacks, reflection, plugins, and runtime dispatch require special handling.
Expose unresolved calls and indexing gaps rather than presenting them as absent
relationships. Static and runtime-observed edges should remain distinguishable.

## 3. Search tags before searching code

Start by finding relevant tags, then retrieve the matching file summaries and
call maps. Expand to neighboring callers, callees, folders, or modules when the
task requires more context.

Use this as the default narrowing strategy, not an irreversible exclusion rule.
If tags are missing, ambiguous, or produce no useful evidence, broaden to
symbol, lexical, or semantic search. Record when fallback was needed so the
catalog can improve.

For a complementary discovery layer, see
[repository embeddings and semantic search](repository-embeddings.md).
Use embedding similarity to find candidates and call maps to inspect their
structural relationships.

## 4. Budget the working context

Tokens are the model's working space. A million tokens corresponds very roughly
to 600,000-800,000 English words, not a guaranteed word maximum. The tokenizer,
content, and provider's accounting determine actual capacity.
See [context windows](context-windows.md) and its primary sources.

Budget for the task, instructions, conversation, retrieved evidence, tool
metadata, and output according to the model's limits. Do not spend the full
allowance on documentation.

## 5. Keep file documentation separate from source

For each source file, maintain a short companion document, for example
`docs/files/src/payments/validator.py.md`.

Recommended fields:

```text
Source path and revision:
Purpose and responsibilities:
Tags:
Key symbols:
Inputs, outputs, and side effects:
Important invariants and error behavior:
Caller/callee map reference:
Related files and summaries:
Relevant tests:
```

Use repository-relative links that resolve to real paths. Keep file paths intact
to avoid collisions between files with identical names in different folders.

## 6. Reveal code progressively, but verify against it

Use summaries to find where to look, then read the relevant functions and their
direct dependencies. Load whole files only when their surrounding context is
necessary.

Delay source access during broad discovery, not during verification. Before
editing code or making precise behavioral claims, inspect the actual source and
relevant tests. Summaries are navigation aids, not authoritative substitutes.

## Freshness, access, and evaluation

Regenerate affected summaries and graph edges when source changes. Associate
artifacts with a commit and file content hash; mark stale entries explicitly.
Handle local uncommitted changes separately from the committed index.

Apply source access controls to tags, summaries, graph edges, caches, and search
results as well as code: derived artifacts can reveal sensitive information.
Do not publish an actual codebase index without authorization.

Evaluate the design on realistic tasks. Useful measures include retrieval recall,
unsupported behavioral claims, unresolved graph coverage, stale artifacts,
context usage, latency, and correctness of resulting code changes.

The goal is not merely fewer tokens. It is enough trustworthy evidence to
complete the task while avoiding irrelevant context.
