---
name: cmr_collection_search
description: Search NASA CMR for Earth science dataset collections
affordances: [http]
parameters:
  - name: keyword
    required: true
    type: string
    description: Targeted free-text term(s) derived from science question — must be specific enough that the best match ranks first
  - name: variable_name
    type: string
    description: Variable name(s) inferred from question or literature
  - name: short_name
    type: string
    description: Dataset short name if known (retrieve via CMR keyword search)
  - name: instrument
    type: string
    description: Instrument filter
  - name: temporal_start
    type: string
    description: "ISO timestamp for temporal range start (e.g. 2020-01-01T00:00:00Z)"
  - name: temporal_end
    type: string
    description: "ISO timestamp for temporal range end (e.g. 2023-12-31T23:59:59Z)"
  - name: bounding_box
    type: string
    description: "Spatial bounding box as 'west,south,east,north' (e.g. '-180,-90,180,90')"
  - name: page_size
    type: integer
    default: 1
    description: Always set to 1. Query must be targeted enough to surface the best result first.
---

# CMR Collection Search

Queries the NASA CMR Collections Search API to find Earth science datasets relevant to a research question. Returns UMM-JSON metadata for each matching collection.

## Endpoint
`GET https://cmr.earthdata.nasa.gov/search/collections.umm_json`

## Headers
- No authentication required (public API)

## Query parameters sent to API
- `keyword` (required) — free-text search
- `variable_name` — variable name filter
- `short_name` — dataset short name filter
- `instrument` — instrument filter
- `temporal[]` — ISO timestamps as `temporal[]=start,end`
- `bounding_box` — spatial extent as `west,south,east,north`; omit for global
- `page_size` — always 1 (query must be precise)

Always request UMM-JSON format. Do not paginate — if the top result is not relevant, reformulate the query rather than fetching more pages.

## Response JSON structure (UMM-JSON)
The response is a JSON object with:
- `hits` (integer) — total matching collections
- `items` (array) — list of collection objects

Each item in `items` has:
- `meta` — metadata about the record
- `umm` — the actual collection metadata, containing:
  - `ShortName`, `EntryTitle`, `Abstract`
  - `Platforms[]` (each has `ShortName`)
  - `Instruments[]` (each has `ShortName`)
  - `ProcessingLevel` (has `Id` field)
  - `ScienceKeywords[]`
  - `DataCenters[]`
  - `RelatedUrls[]`
  - `TemporalExtents[]` (each has `RangeDateTimes` with `BeginningDateTime`, `EndingDateTime`)
  - `SpatialExtent` (if present)

**Important:** Parse `items` directly — NOT `feed.entry`. This is UMM-JSON format, not the legacy JSON format.

## Output format
For each collection returned, show:
```
- {ShortName}: {EntryTitle}
  Processing Level: {ProcessingLevelId}
  Temporal: {TemporalExtents}
  Platforms: {Platforms}
  Abstract: {Abstract (first 200 chars)}
```

If any field is null or missing, display `n/a`.

## Response patterns
See `responses/` directory for handling of specific scenarios (success, no results, too many results, incomplete metadata).

## When to use
- After GCMD keyword lookup has normalized the search terms
- Primary discovery mechanism for NASA Earth science datasets
