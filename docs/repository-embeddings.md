# Repository embeddings and semantic search

## The useful idea

Precompute a searchable representation of repository content, persist it, and
reuse it across agent sessions. This can help locate relevant code without
repeatedly loading the entire repository.

The important distinction: give the agent access to a search tool backed by the
index, not a prompt containing all of the numerical vectors.

Embedding-based retrieval compares vectors to find relevant content. The
answering model receives the retrieved text, not the entire vector index.
See the [embeddings guide][embeddings].

## Indexing is not pretraining

An embedding model converts input into a numerical representation used for
similarity comparisons. Running code through an existing embedding model does
not train the answering model on that repository.

An embedding is not a portable, compressed repository that any language model
can directly read from a list of numbers. Keep the original text or a reliable
way to retrieve it alongside the vectors.
See [embeddings and retrieval][embeddings].

## Similarity is not a call relationship

Two retry implementations may be semantically similar without calling one
another. Conversely, a caller and callee may perform very different operations.

Use semantic retrieval to discover candidates, and a resolved call/dependency
graph to investigate structural connections. A similarity score should not be
treated as proof that one function calls another.

See [semantic search and language intelligence][workspace] and the proposed
[tags-and-call-maps strategy](code-knowledge-base-strategy.md).

## Proposed architecture

```text
BUILD OR UPDATE
Source + file summaries
  -> function/class-aware chunks
  -> embedding model
  -> vector index + source metadata

ON AGENT STARTUP
Open the saved index through a search tool

FOR EACH QUESTION
Tags + exact symbols + semantic search
  -> candidate summaries and symbols
  -> relevant call-graph neighbors
  -> targeted source excerpts
  -> evidence in the answering model's context
```

This is an architecture proposal, not an implemented indexer in this repository.
Semantic repository indexing is an established way to support code discovery;
see [workspace indexing][workspace].

## Recommended index design

- Start with function/class chunks and file summaries, not only whole-file
  vectors. Preserve enough surrounding context to interpret each chunk.
- Store source paths, qualified symbol identities, line ranges, content hashes,
  and the source revision with each entry.
- Record the embedding model, version, dimensions, and chunking scheme.
  Keep document and query encodings compatible; rebuild when required.
- Update changed entries and remove deleted entries. Handle uncommitted changes
  separately from a committed index.
- Evaluate retrieval on representative questions instead of selecting a model
  solely because it is large or expensive.

Treat these as starting recommendations to validate on the actual codebase.
Measure relevance, missed dependencies, freshness, latency, and context usage.

## Storage and Git

One possible local layout:

```text
repo\
  .code-index\      Generated vectors, metadata, and graph
  docs\            Human-readable documentation
  .gitignore       Ignore patterns
```

Put this pattern inside `.gitignore`:

```gitignore
.code-index/
```

The vectors live in the directory, not in `.gitignore` itself. Ignored untracked
files are not included in ordinary Git adds, and existing tracked files are not
untracked by adding an ignore pattern. A local ignored index is not automatically
distributed to other clones.
See the [Git ignore documentation][gitignore].

For a team deployment, consider an access-controlled shared index keyed by
repository and revision instead of committing generated vector data.

## Privacy and context boundaries

Treat vectors, summaries, metadata, and graphs as potentially sensitive derived
artifacts. Ignoring files in Git is not an access-control mechanism.
For private code, use only approved embedding services or approved local models.

The model still needs relevant evidence in its context to answer. The index
improves discovery; it does not permanently teach the answering model the
repository or guarantee correct conclusions.
See [retrieval-based question answering][embeddings].

[embeddings]: https://developers.openai.com/api/docs/guides/embeddings
[workspace]: https://code.visualstudio.com/docs/agents/reference/workspace-context
[gitignore]: https://git-scm.com/docs/gitignore
