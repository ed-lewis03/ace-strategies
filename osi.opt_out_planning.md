# Strategy: osi.opt_out_planning

**Agent source:** osi
**Result type:** opt_out_planning

## Synthesis instructions for ACE

When an `osi` row with `result_type=opt_out_planning` is present, it enumerates the subject's exposure across consumer data brokers and the takedown method for each broker. The point of this pillar is *operator action*, not adverse-content monitoring (use reputational for that) and not professional context (use social_presence for that).

- Lead with the count of actionable broker listings: `payload.findings[]` filtered to `identity_confidence in ("high", "medium")`.
- Render each broker finding as a takedown row, NOT a paragraph. Suggested shape: `{broker}: {record_summary} → {opt_out_method} at {opt_out_url} ({opt_out_notes})`.
- Group findings by `payload.findings[].broker`; within a broker, list highest `identity_score` first.
- Treat `identity_confidence="high"` as ground truth for "this is your subject's record." Treat `medium` as actionable with operator review (location-only matches are common). Treat `low` candidates as audit-only, they live in `payload.unclassified[]`, do not propose takedowns for them.
- Surface `payload.brokers_skipped[broker]` reasons when a registry broker was queried but produced no record. Common reasons: `no_search_results`, `no_record_pages_in_results`, `all_below_medium_confidence`. Do not invent a takedown for a skipped broker.
- Recommend `opt_out_method=phone` brokers be handled first when latency matters, since they confirm faster than `form` (which require email confirmation round-trips).
- Note that opt-out is ongoing, many brokers re-list within 3-12 months. Recommend a quarterly re-scan cadence.
- If a concurrent `physical_safety` row exists with severity `high`/`critical`, escalate the takedown plan's tone (lead with "URGENT") and recommend the operator prioritize home-address-bearing brokers (Spokeo, WhitePages, BeenVerified, Radaris) first.
- Do NOT auto-submit or pretend to submit takedown forms. The pillar reports targets; the operator (or a human-in-the-loop pass) executes them.

## Broker-grounded evidence (phase 6, OSI Evidence Broker 0.7.0)

Starting with calendar version `osi-2026.04.26-2`, opt_out_planning rows always carry broker-fetched listings in `payload`:

- `payload.findings[]`: high-and-medium-confidence broker records the broker resolved against the subject's identity_anchors (employer, location, domain_hint). Each item: `{broker, url, source_domain, record_summary, opt_out_url, opt_out_method, opt_out_notes, identity_score, identity_confidence, matched_anchors[], record_metadata, snippet, title, fetched_at, extras}`.
- `payload.unclassified[]`: same shape, items the broker found but could not anchor at medium-or-better confidence. Audit-only, do not propose takedowns unless the analyst explicitly asks for low-confidence candidates.
- `payload.brokers_searched[]`: which registry brokers the broker actually queried this run. Default registry covers Spokeo, WhitePages, BeenVerified, Intelius, MyLife, TruePeopleSearch, FastPeopleSearch, FamilyTreeNow, Radaris, PeekYou.
- `payload.brokers_skipped[]`: dict of `{broker: skip_reason}` when a broker was queried but produced no qualifying record. Common reasons: `no_search_results`, `no_record_pages_in_results`, `all_below_medium_confidence`.
- `payload.summary`: broker-generated one-paragraph plan across all brokers, including a count of actionable listings.
- `payload.broker_cache_hit`: true if the broker served from its 24h subject cache.
- `payload.broker_status_code`: HTTP status from the broker call. None means the OSI Evidence Broker was unreachable from Tracecat.
- `payload.broker_skip_reason`: present when the broker call could not run, e.g. `no_identity_anchors_available` (wf01 produced no employer, location, or domain hint to anchor against).
- Top-level `evidence_urls[]` on the `agent_results` row mirrors `payload.findings[].url` for SQL-side filtering.

## Per-broker takedown notes (registry as of phase 6)

- **Spokeo** (`spokeo.com`): form at `spokeo.com/optout`. ~7 day SLA. Email confirmation round-trip.
- **WhitePages** (`whitepages.com`): form at `whitepages.com/suppression_requests`. ~24 hours. Phone verification (call the listed number).
- **BeenVerified** (`beenverified.com`): form at `beenverified.com/app/optout/search`. ~24 hours. Email confirmation round-trip.
- **Intelius** (`intelius.com`): form at `intelius.com/opt-out`. ~72 hours. Requires email + last-4 SSN OR uploaded ID.
- **MyLife** (`mylife.com`): phone (1-888-704-1900) is fastest; CCPA form for CA residents only.
- **TruePeopleSearch** (`truepeoplesearch.com`): form at `truepeoplesearch.com/removal`. ~24 hours.
- **FastPeopleSearch** (`fastpeoplesearch.com`): form at `fastpeoplesearch.com/removal`. ~24 hours. Same operator as TruePeopleSearch.
- **FamilyTreeNow** (`familytreenow.com`): form at `familytreenow.com/optout`. CAPTCHA. Re-listing common, recheck quarterly.
- **Radaris** (`radaris.com`): form at `radaris.com/control/privacy`. Account creation OR phone verification. Re-listing reported within 6 months.
- **PeekYou** (`peekyou.com`): form at `peekyou.com/about/contact/optout/`. ~7 days.

## What the pillar deliberately does NOT do

- Auto-submit takedown requests. CAPTCHAs and per-broker friction make automation brittle, and a wrongly-submitted form can delete a real (non-subject) person's record.
- Cover the long tail beyond 10 brokers. Phase 6 ships a 10-broker high-traffic registry. Onerep and similar commercial services cover ~190; if a case needs that depth, escalate to Onerep manual handoff.
- Track takedown completion. The pillar reports targets at scan time. A separate operator workflow tracks "submitted → confirmed → re-listed" status; do not claim records have been removed.
