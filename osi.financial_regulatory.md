# Strategy: osi.financial_regulatory

**Agent source:** osi
**Result type:** financial_regulatory

## Synthesis instructions for ACE

When an `osi` row with `result_type=financial_regulatory` is present, it captures regulatory / financial-exposure findings (sanctions hits, adverse media, litigation, corporate filings, PEP status, AML flags).

- Lead with `payload.severity` and name the regime(s) involved (OFAC, EU, UK, UN, etc.) when available.
- Any sanctions/PEP hit is lead material — do not bury it.
- Quote `payload.findings[]` with source and date; stale findings (>24 months) should be noted as "historical."
- `payload.recommendations[]` are compliance actions — render as a checklist, not prose.
- If severity is `high` or `critical`, include a "do not proceed without compliance review" banner.
- Do not render legal opinion — cite findings, let the operator's compliance team interpret.

## Broker-grounded evidence (phase 3, OSI Evidence Broker)

Starting with calendar version `osi-2026.04.25-2`, every financial_regulatory row also carries broker-fetched, cited evidence in `payload`:

- `payload.evidence[]`: array of `{url, source_domain, title, snippet, content_excerpt, published_at, fetched_at, pre_filter_score, pre_filter_label, extraction_method}`. These are real adverse financial/regulatory hits (SEC, OFAC, FINRA, CFPB, court records, enforcement coverage) the broker found and pre-classified using the financial-regulatory keyword profile.
- `payload.unclassified[]`: items filtered as noise (social, recipe pages, off-subject results). Audit trail, do not surface unless asked.
- `payload.broker_query`: literal search query the broker used.
- `payload.broker_skip_reason`: present when the broker found nothing actionable (`no_search_results`, `no_adverse_media_found`, `subject_not_found_in_results`).
- `payload.broker_candidate_count`: how many raw search hits the broker considered.
- `payload.broker_cache_hit`: true if served from the broker's 24h subject cache (cache is pillar-scoped, so a reputational hit on the same subject does NOT satisfy this).
- `payload.broker_status_code`: HTTP status from the broker call. None means the broker was unreachable.
- Top-level `evidence_urls[]` on the `agent_results` row mirrors `payload.evidence[].url` for SQL-side filtering.

When composing the financial/regulatory narrative:

- Cite specific URLs from `payload.evidence[]` rather than asserting unsourced claims. Each adverse statement should map to at least one entry.
- Weight `.gov` and primary-source domains (sec.gov, ofac.treasury.gov, finra.org, justice.gov, courtlistener.com) as the strongest evidence; news outlets are corroborating, not primary.
- Prefer items with `pre_filter_label="adverse"` over `borderline`. Borderline items are subject-mention pages without strong financial-regulatory signal; useful as context, not for sanctions/enforcement claims.
- Distinguish enforcement actions (SEC/OFAC/FINRA/CFPB) from civil litigation (private suits, class actions) and from criminal proceedings (DOJ/USAO indictments). Cite which when surfacing each finding.
- If `payload.evidence` is empty AND `payload.broker_skip_reason` is set, say "no enforcement or financial-regulatory evidence found via OSI's search" with the skip reason, do not invent findings.
- If the row's `status="partial"` with `skip_reason="broker_unreachable"`, be explicit: the financial_regulatory pillar could not reach its evidence broker on this run, treat as no-data, do not fall back to training-data prose about the subject.
- Existing `payload.findings[]` from prior pillar logic still takes precedence when present (it represents human-curated entries); evidence is supplementary.
