# artifact-agent

An agent whose behavior is entirely defined by markdown artifacts. Swap the artifacts directory — get a completely different agent. Zero code changes.

Built with [pydantic-ai](https://ai.pydantic.dev/).

## How it works

```
artifacts/code_search/          <-- swap this directory
  index.md                      <-- root context (loaded as system prompt)
  contexts/
    index.md
    repo_search_expert.md
  guardrails/
    index.md
    search_boundaries.md
  tools/
    index.md
    github_search.md
    gitlab_search.md
```

1. **System prompt** = root `index.md` + full artifact file tree
2. Agent uses `load_artifact` tool to dynamically pull in relevant artifacts during reasoning
3. Agent reads index files first (table of contents), then loads specific artifacts based on the query
4. Artifacts define everything: persona, guardrails, tool usage, domain knowledge

The agent does its own context management — it decides what to load based on the user's query.

## Setup

```bash
# install dependencies
uv sync

# set your API key
echo "OPENAI_API_KEY=sk-..." > .env
```

## Usage

```bash
# default artifacts directory
uv run python agent.py "find python repos for RAG pipelines" --artifacts artifacts/code_search

# different artifact set = different agent
uv run python agent.py "check SPD-41a compliance for my dataset" --artifacts artifacts/compliance

# help
uv run python agent.py --help
```

## Creating your own artifacts

Create a directory with a root `index.md` and organize subdirectories however you want. There are no hardcoded categories — the agent discovers the structure from index files.

```
my_artifacts/
  index.md          <-- describes the agent's purpose, lists categories
  category_a/
    index.md        <-- describes what's in this category
    file1.md
    file2.md
  category_b/
    index.md
    file3.md
```

Then run:

```bash
uv run python agent.py "your query" --artifacts my_artifacts
```

## Logging

All agent activity is traced via [loguru](https://github.com/Delgan/loguru):

```
23:07:03 | DEBUG   | [Store] indexed: contexts/index.md
23:07:03 | INFO    | [Store] ready — 8 artifacts indexed from artifacts/code_search
23:07:03 | DEBUG   | [Artifact] system prompt assembled (1243 chars)
23:07:04 | INFO    | [ToolCall] load_artifact('contexts/repo_search_expert.md') → 542 chars | # Repository Search Expert...
23:07:05 | INFO    | [ToolCall] search_github(query='RAG pipeline language:python', sort='stars', per_page=5)
23:07:06 | DEBUG   | [ToolCall] search_github → 1832 total results
```

- `[Store]` — artifact store initialization (reading files from disk)
- `[Artifact]` — system prompt assembly
- `[ToolCall]` — agent invoking tools during reasoning
