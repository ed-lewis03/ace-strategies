# Strategy: osi.social_presence

**Agent source:** osi
**Result type:** social_presence

## Synthesis instructions for ACE

When an `osi` row with `result_type=social_presence` is present, it captures the subject's discoverable social and professional presence across LinkedIn, X (Twitter), GitHub, personal sites, Facebook, Instagram, and Snapchat. The point of this pillar is identity verification and contemporary professional context, not adverse-content monitoring (use reputational for that).

- Lead with the subject's current professional anchor: `payload.findings[].current_role` plus `current_employer` when both present.
- Group findings by `payload.findings[].platform`. Cite each cited profile by `url` and identify the matched anchors via `matched_anchors`.
- Treat `identity_confidence="high"` findings as ground truth for current role/employer/location. Treat `medium` as supportive. Treat `low` as candidate-only and route to `payload.unclassified[]`.
- When two findings on different platforms agree on `current_employer`, that is high-confidence corroboration. When they disagree, flag the conflict explicitly and prefer the more recent `fetched_at`.
- Surface `payload.platforms_skipped[platform]` reasons when a major platform was searched but yielded no qualifying profile (e.g. `all_below_medium_confidence`, `no_results`). Do not invent presence on a skipped platform.
- Do NOT speculate on personal life from social_presence alone. Stick to the literal `headline`, `bio`, `current_role`, `current_employer`, `location`, `recent_activity` strings the broker extracted.

## Broker-grounded evidence (phase 5, OSI Evidence Broker 0.6.0)

Starting with calendar version `osi-2026.04.26-1`, social_presence rows always carry broker-fetched profiles in `payload`:

- `payload.findings[]`: high-confidence profile matches the broker resolved against the subject's identity_anchors (employer, location, domain_hint). Each item: `{platform, handle, headline, current_role, current_employer, location, bio, recent_activity[], profile_metadata, identity_score, identity_confidence, matched_anchors[], extraction_method, fetched_at, url, source_domain, extras}`.
- `payload.unclassified[]`: same shape, items the broker found but could not anchor at medium-or-better confidence. Audit-only, do not surface unless the analyst asks for low-confidence candidates.
- `payload.platforms_searched[]`: which platforms the broker actually queried this run.
- `payload.platforms_skipped[]`: dict of `{platform: skip_reason}` when a platform was queried but produced no qualifying finding. Common reasons: `all_below_medium_confidence`, `no_results`, `platform_disabled`.
- `payload.summary`: broker-generated one-paragraph human summary across all platforms.
- `payload.broker_cache_hit`: true if the broker served from its 24h subject cache.
- `payload.broker_status_code`: HTTP status from the broker call. None means the broker was unreachable.
- `payload.broker_skip_reason`: present when the broker call could not run, e.g. `no_identity_anchors_available` (wf01 produced no employer, location, or domain hint to anchor against).
- Top-level `evidence_urls[]` on the `agent_results` row mirrors `payload.findings[].url` for SQL-side filtering.

When composing the social_presence section:

- Cite specific URLs from `payload.findings[]` for each statement about the subject's current role or platform presence. No anchor-less professional claims.
- If `payload.findings` is empty AND `payload.broker_skip_reason="no_identity_anchors_available"`, say "social presence skipped: intake provided no employer, location, or domain to anchor disambiguation" and do NOT fall back to unanchored search results.
- If `status="partial"` with `skip_reason="broker_unreachable"`, treat as no-data and say so. Do not fall back to training-data prose about the subject's known social presence.
- If `status="partial"` with `skip_reason="no_adverse_media_found"` (carried over from generic broker tri-state, here means "no qualifying profiles across any searched platform"), say "no platform profiles met the identity-anchor confidence threshold" and surface `payload.platforms_searched[]` so the analyst sees what was tried.
- For LinkedIn specifically: `extras.platform=linkedin` profiles often show truncated public-snippet data because LinkedIn rate-limits unauthenticated reads. The `bio` field may be partial; do not infer absence of detail from a short bio.
