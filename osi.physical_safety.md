# Strategy: osi.physical_safety

**Agent source:** osi
**Result type:** physical_safety

## Synthesis instructions for ACE

When an `osi` row with `result_type=physical_safety` is present, it captures physical-safety risk findings (location exposure, doxxing, direct threats, travel/residence risk).

- Lead with `payload.severity` (expected values: `critical` | `high` | `medium` | `low` | `informational`).
- If severity is `critical` or `high`, preface the response with an explicit action-required framing.
- Quote the most severe item from `payload.findings[]` verbatim before summarizing.
- Always surface `payload.recommendations[]` — these are operator actions, not advisory.
- If any finding references a location or travel pattern, do NOT repeat the raw location in the headline; reference it as "a specific location" and defer detail to the drill-down.
- If a concurrent `reputational` or `financial_regulatory` row has overlapping findings, note the correlation.

## Broker-grounded evidence (phase 3, OSI Evidence Broker)

Starting with calendar version `osi-2026.04.25-2`, every physical_safety row also carries broker-fetched, cited evidence in `payload`:

- `payload.evidence[]`: array of `{url, source_domain, title, snippet, content_excerpt, published_at, fetched_at, pre_filter_score, pre_filter_label, extraction_method}`. These are real adverse safety hits (criminal records coverage, arrest reports, threat / incident news, restraining orders, violence-adjacent reporting) the broker found and pre-classified using the physical-safety keyword profile.
- `payload.unclassified[]`: items filtered as noise. Audit trail, do not surface unless asked.
- `payload.broker_query`: literal search query the broker used.
- `payload.broker_skip_reason`: present when the broker found nothing actionable (`no_search_results`, `no_adverse_media_found`, `subject_not_found_in_results`).
- `payload.broker_candidate_count`: raw search-hit count considered.
- `payload.broker_cache_hit`: true if served from the broker's 24h pillar-scoped subject cache.
- `payload.broker_status_code`: HTTP status from the broker call. None means the broker was unreachable.
- Top-level `evidence_urls[]` on the `agent_results` row mirrors `payload.evidence[].url` for SQL-side filtering.

When composing the physical-safety narrative:

- Cite specific URLs from `payload.evidence[]` rather than asserting unsourced claims. Each safety claim should map to at least one entry.
- Weight criminal records, court documents, and law-enforcement releases as primary; news coverage is corroborating.
- Prefer items with `pre_filter_label="adverse"` over `borderline`. Borderline items are subject-mention pages with weak safety signal; useful as context, not for incident or threat claims.
- Continue to apply the location-redaction rule above to any location surfaced by an evidence item: do not echo a raw address or precise location in the headline; defer to drill-down.
- If `payload.evidence` is empty AND `payload.broker_skip_reason` is set, say "no physical-safety evidence found via OSI's search" with the skip reason, do not invent findings.
- If the row's `status="partial"` with `skip_reason="broker_unreachable"`, be explicit: the physical_safety pillar could not reach its evidence broker on this run, treat as no-data, do not fall back to training-data prose about the subject.
- Existing `payload.findings[]` from prior pillar logic still takes precedence when present (it represents human-curated entries); evidence is supplementary.
