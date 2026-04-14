# Search Rules

## Tool ordering
1. `gcmd_keyword_lookup` — always first, for every variable, before any CMR search
2. `cmr_collection_search` — after vocabulary mapping is complete
3. `semantic_scholar_search` — conditional only; triggered by insufficient CMR results

## CMR search rules
- Always search at the **collection level only** — granule-level search is out of scope
- Always confirm every returned collection actually exists and is active in CMR before surfacing it
- Never recommend a collection based solely on prior knowledge or reference documents without a live CMR query
- Use `page_size=1` per targeted query — query must be precise enough that the best match is first
- Run multiple targeted queries rather than one broad query

## GCMD rules
- Always run `gcmd_keyword_lookup` before the first `cmr_collection_search` for any variable
- Only fall back to free-text search if lookup returns no match, and flag this to the researcher
- Use only the local gcmd.json — do NOT call the live KMS API

## Clarification rules
- All clarifications are **blocking** — no searches run until answered
- Batch all questions together; maximum 5 questions per pause
- No partial or speculative searches under any circumstances
- Agent may infer defaults (spatial=Global, temporal=current year) but may NOT execute searches using inferred values without user confirmation

## Result rules
- Return 5–6 collections that together address all aspects of the science question
- Do not return fewer than 5 without explaining why
- Do not return an unranked dump of 20+ collections
- For every surfaced dataset, explain *why* it appears
- No recommendations or endorsements — present ranked options only
- Verify temporal coverage against the confirmed study period (mandatory)

## Source hierarchy
1. CMR (authoritative — gates all final recommendations)
2. gcmd.json local vocabulary (authoritative for keyword normalization)
3. Semantic Scholar (informational — feeds variable refinement)
4. Reference documents in workspace (reasoning aids — never substitute for a live CMR query)
