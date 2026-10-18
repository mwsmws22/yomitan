# Bunpro deck membership — TODO summary

**Branch:** `rd/bunpro-deck-membership` (local R&D; not pushed)  
**Parent PR:** [#2498](https://github.com/yomidevs/yomitan/pull/2498) `bunpro-integration`  
**Status:** Research only — no product code yet

## Goal

Show optional deck badges on Bunpro matches (e.g. 音 for オノマトペ) alongside
existing level labels (N1 / A2), for many decks, without `decks × search-results`
API fan-out.

## Findings (done)

- [x] Vocab and search payloads do **not** include deck membership
- [x] Vocab UI `?deck_id=` pill comes from `prev_next_by_deck_id` (needs known deck)
- [x] Membership source: `POST /api/frontend/decks/{deck_code}/contents`
  - Omit `deck_unit_id` → whole deck (オノマトペ = 811; one unit was only 23)
  - Cap **1000 items/page** (`pagy`); Core 4k needs 4 pages
  - Rows expose `contentable_id` (vocab id) for O(1) set membership
- [x] No usable reverse endpoint (“decks containing this vocab”)
- [x] Explore listing: `GET /decks/explore` (official decks by category)
- [x] Per-term probes (`prev_next` × N decks) scale poorly — reject for ~20 decks
- [x] Notes dump: `.tmp/deck-membership-rd.md`

## Recommended approach

**Prefetch + cache vocab-id sets per enabled `deck_code`.**  
Existing Bunpro match already has `id` → `set.has(id)` → badge.  
Warm cache once (or on TTL); **+0 API calls per search** afterward.

Alternatives parked:

| Option | When |
|---|---|
| Ship static id dumps in extension | Want zero runtime Bunpro traffic |
| Per-term `prev_next` probes | Never for multi-deck |

## Implementation TODOs (when we pick this up)

1. [ ] Design: curated deck list vs user-picked explore decks; badge glyph per deck
2. [ ] Protocol: `contents` request/response parse → `Set<ReviewableId>`
3. [ ] Client: background sync + cache (memory / storage) with refresh policy
4. [ ] Wire badges into existing Bunpro control UI next to level chips
5. [ ] Start with オノマトペ (`onoma`) only; prove path before N decks
6. [ ] Tests: parse fixtures, membership hit/miss, pagination
7. [ ] Permissions / rate limits: don’t stampede on every popup open
8. [ ] Decide: merge into #2498 later vs follow-up PR

## Cost sketch (cache warm)

| Deck | code | vocab | pages@1000 |
|---|---|---:|---:|
| オノマトペ | onoma | 811 | 1 |
| Core 2k | u9t7g7 | 1945 | 2 |
| Core 4k | ephdqm | 3881 | 4 |
| Bunpro N1 Vocab | qqovik | 2974 | 3 |

~20 mid/large decks ≈ tens of requests total to warm, not per lookup.

## Out of scope for this branch

- Product UI / settings
- Pushing this branch
- Changing #2498 behavior until design is approved
