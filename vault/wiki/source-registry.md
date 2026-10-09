---
type: source-registry
slug: source-registry
title: Source registry — trusted publishers, tiered
updated: <YYYY-MM-DD>
last-refreshed: <YYYY-MM-DD, set at instantiation>
---

# Source registry

The KB's authority map: **which publishers we trust, how much, and where to look for their latest.** A per-*publisher* view (the [[index]] lists per-*URL*). Three jobs:

1. **Drive search** — start discovery from Tier-1/Tier-2 publishers before open web search.
2. **Rank candidates** — score a freshly-found source by the tier of its publisher; suppress reject-list domains.
3. **Refresh** — walk the registry periodically, pull what each trusted publisher has shipped since `last-refreshed:`, run it through normal curation/ingest.

## Tier rubric

Tier reflects a publisher's *typical* signal. Individual articles can be tiered up or down (a first-hand interview with the originators on an otherwise-T2 outlet is T1 for that piece; a paraphrase post on a T1 outlet isn't).

- **Tier 1 — Primary / originator.** As close to the source as you can get: the originating organisation or author, official standards bodies, a maker's own documentation *for its own product or method*, and high-fidelity interviews/talks *with the people who originated the thing*. **Cite, don't paraphrase.**
- **Tier 2 — Trusted practitioner / named outlet.** A named person or outlet with a track record, offering original analysis or first-hand experience. Quote sparingly, synthesise freely.
- **Tier 3 — Provisional / community.** One-off authors, multi-author platforms, aggregator repos, vendor blogs with a sales angle, benchmark/SEO-adjacent sites. Keep, but flag. **Promote** to T2 on a second high-signal contribution; **demote/remove** if it adds no signal beyond what T1 already says.
- **Reject.** SEO content farms, listicles that paraphrase primary sources, low-effort generated content. Named in the Reject list below.

**Authority criteria** for a candidate found in discovery; reject one that fails more than two:

- **First-party for the topic.** The entity that built, authored or standardised the thing, not a commentator.
- **Track record.** The author has a body of work in the domain and is cited by others in it.
- **Specificity.** Names things, cites numbers, describes mechanisms; not vague generalities.
- **Cites its sources.** Claims are traceable.
- **Dated and current.** A clear publish date; how recent is enough depends on how fast the field moves.
- **Acknowledges alternatives.** Reasons through trade-offs rather than presenting one approach as the only path.

**Hard rejects**, dropped rather than listed as Borderline: SEO content farms; AI-generated content without editorial review; listicles that aggregate without analysis; vendor marketing without engineering substance; sites with no track record; any domain on the reject list.

## How the search pipeline uses this (operational contract)

For `kb-intake` triage (discover, prospect and inbox modes):

- A candidate whose domain matches a **T1** row → recommend auto-approve.
- **T2** → recommend approve, light triage.
- **T3** → manual triage, default skeptical.
- **Reject-list** domain → drop, don't surface (inbox mode: Borderline, since the operator saved it).
- **Unknown** domain → triage normally; if it earns ingest, kb-ingest adds a provisional row at the tier given at triage.

For `kb-intake` refresh mode: sweep the T1+T2 rows by the Instance refresh route since `last-refreshed:`, dedup against `wiki/sources/`, and write new URLs into a curation file for normal triage; held stubs of living-doc rows past the staleness threshold become re-fetch candidates. Bump `last-refreshed:` only after a full sweep, not a single-publisher one.

Columns: **Domain(s)** are the hosts that identify the publisher; **Stubs** is the number of source stubs from it in `wiki/sources/`, bumped by kb-ingest at the end of each wave; **Listing to refresh** is a page or feed that lists its new material (empty when domain search is enough); **Verdict** (T3) is, for example, `keep`, `promote` or `demote`, with a short reason.

## Tier 1 — primary / originator

| Publisher | Domain(s) | Who / what | Stubs | Listing to refresh |
|---|---|---|---|---|

## Tier 2 — trusted practitioner / named outlet

| Publisher | Domain(s) | Who | Stubs | Listing to refresh |
|---|---|---|---|---|

## Tier 3 — provisional / validate

| Publisher | Domain(s) | Contribution (citations) | Verdict |
|---|---|---|---|

## Reject list

Criteria: SEO content farms, paraphrase listicles, low-effort generated content that adds no signal beyond primary sources. kb-ingest appends a wave's hard-rejected domains at close. Each entry: `domain — reason — wave`.
