---
name: semantic_scholar_search
description: Search scientific literature to refine variables and dataset candidates for CMR search
affordances: [http]
parameters:
  - name: query
    required: true
    type: string
    description: Science question keywords (5-10 words recommended)
  - name: limit
    type: integer
    default: 5
    description: Number of results (5-10 recommended)
  - name: fields
    type: string
    default: "title,abstract,year,doi,topics,citationCount"
    description: Comma-separated field list
---

# Semantic Scholar Search

Queries the Semantic Scholar Graph API to find peer-reviewed papers related to the researcher's science question. Triggered **only as a fallback** when the initial CMR search returns insufficient results.

The agent uses returned abstracts and keywords to infer additional variables, resolution requirements, and processing levels — then feeds these into a refined CMR search.

## Trigger condition
Invoke only when initial CMR search returns 0 results or results are clearly not relevant to the science question.

## Endpoints

**Paper search:** `GET https://api.semanticscholar.org/graph/v1/paper/search`
- `query` (required) — science question keywords
- `limit` — number of results (default 5)
- `fields` — comma-separated: title,abstract,year,doi,topics,citationCount

**Paper detail:** `GET https://api.semanticscholar.org/graph/v1/paper/{paperId}`
- `paperId` (required) — ID from paper search result
- `fields` — same field list

## Authentication
API key from environment variable `SEMANTIC_SCHOLAR_API_KEY`. Pass it in header `x-api-key`. If no key is set, proceed without the header (public tier, more rate-limited).

## Rate limit and error handling
1 request per second — must be respected. If a request fails (HTTP 429, 500, timeout, etc.), retry once. If the second attempt also fails, return a human-readable message like "Literature search unavailable: {reason}" — do NOT raise exceptions. Max 2 attempts per request.

## Fields to extract from results
- title, abstract, year, doi
- topics / keywords
- citationCount (proxy for paper quality/relevance)
- Dataset short names (inferred from abstract text — no structured "datasets used" field)

## Output format
For each paper:
```
- {title} ({year}) [citations: {citationCount}]
  DOI: {doi}
  Variables mentioned: {extracted variables}
  Datasets mentioned: {inferred dataset names}
  Abstract: {first 200 chars}
```

## Notes
- Dataset mentions must be inferred from abstract text; there is no structured field
- To resolve inferred dataset names to CMR short_names, use cmr_collection_search with the inferred name as keyword
- Prefer recent papers (last 5-10 years) unless the science question is historical

## Response patterns
See `responses/` directory for handling of success, no papers found, and rate limit scenarios.

## When to use
- Only when initial CMR search returns insufficient results
- To extract additional variables, resolution requirements, and processing level signals from literature
- Results feed back into refined CMR search — not surfaced directly to user
