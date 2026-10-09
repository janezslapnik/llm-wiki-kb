---
name: kb-intake
description: "Finds and triages candidate sources for the knowledge base, writing a curation file in `raw/curation/` that kb-ingest consumes. Use for: find, discover or 'what's out there on' a topic; search from a prospect, or 'what should we search next'; a refresh sweep of the trusted publishers ('refresh the KB', 'freshness sweep', 'what's new from our publishers'); process my inbox or saved links; triage or approve the candidates in a curation file. Not for answering from the KB (kb-query), or for ingesting or re-fetching a known source (kb-ingest)."
---

# kb-intake

Turns a topic, a prospect, a refresh sweep or the inbox into a curation file under `raw/curation/`, then runs its triage. It never clips or writes wiki pages; kb-ingest does. Light gist fetches for authority and the citation graph are fine. Routes, tools and the identity key come from CLAUDE.md `## Instance`; the authority rubric, tiers and reject list from `wiki/source-registry.md`.

## Modes

The mode decides which steps run and when anything is committed.

| Mode | Cue | Runs | Commits |
|---|---|---|---|
| discover | a topic | steps 1–8, triage | none |
| prospect | names a `raw/prospects/` file | steps 1–8 seeded by the prospect, `from_prospect: <slug>`, triage | after triage, one commit: curation file + `git mv` of the prospect to `past-searched/` + log `prospect \| resolved <slug>`. Abandoned triage → none |
| refresh | refresh or freshness sweep, what's new from our publishers | Instance refresh route over registry T1/T2 rows, steps 5–8 with the refresh rules, `from_refresh: true`, `since: <last-refreshed>`, triage | one commit **before** triage: the file, plus the `last-refreshed:` bump if every T1/T2 row was searched (a row that returned nothing counts; an unchecked row or a subset sweep doesn't; note thin rows in the file). A bump without its file would silently skip a window; log `refresh \| <since> → <today>` |
| inbox | the inbox, saved links | inbox rules, steps 5–8, `from: inbox`, triage | none |
| triage | names or implies an existing curation file | triage only | none |

"What should we search next" → list the open prospects and the unactioned `### Candidate follow-up` items of recent Ingest reports, and ask which to run.

## Steps

1. **Gap.** One sentence on what the wiki lacks; check `wiki/index.md` and `wiki/sources/` with `rg`. No gap, or one a single-source ingest covers → say so and stop.
2. **Search** per the Instance discover route; operator-given URLs are starting points.
3. **Authority.** Apply the registry's `## Tier rubric` and drop its reject list. A domain you hard-reject as a whole site, not for one weak page, goes under `## Hard rejects` (`domain — reason`); kb-ingest adds it to the reject list.
4. **Citation graph**, a distinct pass rather than another search: follow the top few candidates' references by the Instance route; put what you find through step 3.
5. **Dedup** on the Instance identity key against one `rg -N --no-filename '^url:' wiki/sources/` pass. Matches go under `## Already in KB` with their slug.
6. **Tier** from the publisher's registry row. Unknown publisher → tier it yourself and add "new publisher: add a provisional registry row at ingest" to the Why.
7. **Sort** into Candidates; Exemplars (artefacts better pointed at than ingested; `Kind: example-file | example-folder | example-repo`); and Borderline: items that pass the hard rejects but carry a named concern (vendor-aligned, single author, polemic, paywalled), listed with the concern rather than dropped.
8. **Write** `raw/curation/YYYY-MM-DD-<topic-slug>.md` from this template:

```markdown
---
type: curation
slug: curation-YYYY-MM-DD-<topic-slug>
title: Curation wave — <topic> (YYYY-MM-DD)
topic: <one line>
gap: <one sentence>
status: pending
updated: YYYY-MM-DD
---

## Gap
<one sentence>

## Already in KB (auto-skipped)
- `<slug>` — <title> (matched URL: <url>)

## Hard rejects
- <domain> — <reason>

## Candidates

### 1. <Title>
- **URL:** <url>
- **Publisher:** <publisher>
- **Published:** YYYY-MM-DD | living-doc | unknown
- **Tier:** 1 | 2 | 3
- **Status:** pending
- **Why:** <one paragraph>

## Exemplars

### E1. <Title>
- **URL:** <url>
- **Publisher:** <publisher>
- **Tier:** 1 | 2 | 3
- **Kind:** example-repo
- **Status:** pending
- **Why pointer-only:** <one sentence>

## Borderline

### B1. <Title>
- **URL:** <url>
- **Publisher:** <publisher>
- **Tier:** 1 | 2 | 3
- **Status:** pending
- **Concern:** <one sentence>
- **Why include anyway:** <one sentence>

## Triage
(written at triage when pre-authorised)

## Ingest report
(written by kb-ingest)
```

## Prospects

- Read the prospect first: `## Why it matters` shapes the gap, `## What to search for` the queries. `## Resolved when` is framing, not a test: a searched prospect closes even with zero approved.
- To open one, write `raw/prospects/<slug>.md` (`type: prospect`, `slug`, `title`, `updated`; `## Why it matters`, `## What to search for`, `## Resolved when`, `## Notes`; no `status:`, the folder is the state) and log `prospect | open <slug>`.

## Refresh rules

- Reject only on topic or for non-articles (tag pages, pagination, product pages); the tier is the row's.
- Collapse locale and tracking variants via the identity key. Don't let one prolific publisher dominate the wave: keep its best few and note how many were held back.
- Living-doc rows: their held stubs past the Instance staleness go under Candidates with a Why that begins `refresh <slug>:`; kb-ingest re-fetches them.

## Inbox rules

- Run the Instance inbox command. On error, show it verbatim and stop; never run an interactive step (such as a login) on the operator's behalf.
- Work only `raw/inbox/` items with `status: new`; none → say the inbox is empty and stop, no file. Each gets its saved note or one light gist fetch, no citation graph; put the capture date in its Why.
- A reject-list domain goes to Borderline: the operator saved it, so they decide.
- The file is `raw/curation/YYYY-MM-DD-inbox.md`; a same-day file is appended to, numbering continued.
- After triage, set every presented item `status: done`.

## Triage

- Summarise once: counts, then every item by its file number (N, EN, BN) with publisher and tier. Ask one question, end the turn and change nothing until the reply.
- A bare "yes" approves Candidates and Exemplars, not Borderline ("accept B2" promotes one). "approve 1 3" rejects the unlisted numbers; "all except 5" approves the rest; "reject all" rejects everything. An ambiguous reply gets one question, never a guess.
- Set each `- **Status:**` to `approved` or `rejected`, then the wave's `status: approved`, or `ingested` when nothing was approved.
- Triage by your own judgment only if the operator pre-authorised it in this session; record it in the file's `## Triage` section, quoting the authorisation.
- Triage ends the skill: report counts and offer kb-ingest; don't start it unasked.
