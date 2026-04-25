# Strategy: osi.reputational

**Agent source:** osi
**Result type:** reputational

## Synthesis instructions for ACE

When an `osi` row with `result_type=reputational` is present, it captures reputational risk (negative press, social sentiment, adverse mentions, brand-linked controversy).

- Lead with `payload.severity` and overall narrative direction (e.g. "coordinated negative sentiment" vs "isolated incident").
- Summarize `payload.findings[]` by source cluster, do not list every hit.
- Surface `payload.recommendations[]` as PR/comms actions, not legal actions (route those to `financial_regulatory`).
- Distinguish subject-authored vs third-party content when deciding tone.
- If Fusion's `risk_brief` for the same case shows `zignal_*` KPIs, align this narrative with those KPIs; flag contradictions.
- Never recommend deletion / takedown unless a finding explicitly warrants it.

## Broker-grounded evidence (phase 2, OSI Evidence Broker)

Starting with calendar version `osi-2026.04.25-1`, every reputational row also carries broker-fetched, cited evidence in `payload`:

- `payload.evidence[]`: array of `{url, source_domain, title, snippet, content_excerpt, published_at, fetched_at, pre_filter_score, pre_filter_label, extraction_method}`. These are real adverse-media hits the broker found and pre-classified.
- `payload.unclassified[]`: same shape, items that were filtered as noise (linkedin, recipe pages, off-subject results). Audit trail, do not surface unless the analyst asks.
- `payload.broker_query`: the literal search query the broker used.
- `payload.broker_skip_reason`: present when the broker found nothing actionable (`no_search_results`, `no_adverse_media_found`, `subject_not_found_in_results`).
- `payload.broker_candidate_count`: how many raw search hits the broker considered.
- `payload.broker_cache_hit`: true if the broker served from its 24h subject cache.
- `payload.broker_status_code`: HTTP status from the broker call. None means the broker was unreachable.
- Top-level `evidence_urls[]` on the `agent_results` row mirrors `payload.evidence[].url` for SQL-side filtering.

When composing the reputational narrative:

- Cite specific URLs from `payload.evidence[]` rather than asserting unsourced claims. Each adverse statement should map to at least one entry.
- Prefer items with `pre_filter_label="adverse"` over `borderline`. Borderline items are subject-mention pages without strong adverse signal, fine for context but not for accusations.
- If `payload.evidence` is empty AND `payload.broker_skip_reason` is set, say "no adverse media found via OSI's search" with the skip reason, do not invent findings.
- If the row's `status="partial"` with `skip_reason="broker_unreachable"`, be explicit: the reputational pillar could not reach its evidence broker on this run, treat as no-data, do not fall back to training-data prose about the subject.
- Existing `payload.findings[]` from prior pillar logic still takes precedence when present (it represents human-curated entries); evidence is supplementary.
