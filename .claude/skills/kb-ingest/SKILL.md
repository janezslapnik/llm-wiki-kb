---
name: kb-ingest
description: "Ingests approved sources into the wiki: clips each one, writes its source stub, synthesises it into entity pages, logs and commits per source, then closes the wave. Use for: ingest or process the approved candidates or the latest approved wave; add this URL, file or article to the KB; re-ingest or refresh <slug>, including stale sources a lint report flagged. Not for finding or triaging candidates (kb-intake)."
---

# kb-ingest

Takes approved candidates from a curation file in `raw/curation/` into the wiki, one source at a time. Fetch routes and the identity key come from CLAUDE.md `## Instance`; page and record rules from its `## Pages` and `## Record`.

## Reading a curation file

- Work is a `### N.`, `### EN.` or `### BN.` item whose `- **Status:**` value begins with `approved`. Everything else is inert: `pending`, a missing Status line, extra sections, bullets and annotations. Never normalise an old file.
- No approved item → stop and offer triage (kb-intake). Never approve on the operator's behalf.
- A direct request (a URL, a local file, `refresh <slug>`, stale sources from a lint report) is its own approval: write a one-row-per-source curation file first, its item and wave `approved`. A local file is first copied to `raw/assets/<slug>.<ext>`; its URL is `file:raw/assets/<slug>.<ext>`.

## Per source

1. **Duplicate.** Slug or identity key already in `wiki/sources/` → `skipped-duplicate`, unless the operator said `refresh <slug>` or the item's Why begins `refresh <slug>:`. A refresh writes a new dated clip (the old one stays), bumps `fetched:` and `clip:` on the stub, re-checks the citing pages' claims against the new clip, and logs `ingest | refresh <slug>`.
2. **Fetch** per the Instance route, only `http(s)` URLs; `file:` only for the `raw/assets/` copies a direct request made. Pass URLs and filenames single-quoted; one containing whitespace, quotes, backticks or `$` → `skipped-unreachable` with that reason. A fetch a policy hook refuses → `skipped-unreachable` with the hook's reason; never route around it (no mirror, resolver or redirect outside the Instance route). Every route failed → `skipped-unreachable` with the reason and no retry in this wave; log `ingest-skipped`.
3. **Clip** the cleaned full text to `raw/clips/YYYY-MM-DD-<slug>.md`: the article body, without navigation, footers, comments or inline data-URI images. `source_kind` names the route in free text, says what was dropped, and says so when the capture is partial. A model summary (a WebFetch result) is not a clip: page claims would rest on its paraphrase. Exemplars get no clip; refreshing one means checking that its URL resolves and its stub still describes it.
   ```yaml
   ---
   slug: <slug>
   title: <full title>
   url: <identity key>
   publisher: <publisher>
   authors: <authors>
   published: YYYY-MM-DD | living-doc | unknown
   fetched: YYYY-MM-DD
   tier: 1 | 2 | 3
   source_kind: <route>
   ---
   ```
4. **Stub** at `wiki/sources/<slug>.md`, frontmatter only. `authors` and `published` are optional, and `published: unknown` (no date found) counts as dated, not living-doc; exemplars replace `clip:` with `kind: example-file | example-folder | example-repo`; extra keys are kept.
   ```yaml
   ---
   type: source
   slug: <slug>
   title: <full title>
   url: <identity key>
   publisher: <publisher>
   authors: <authors>
   published: YYYY-MM-DD | living-doc | unknown
   fetched: YYYY-MM-DD
   tier: 1 | 2 | 3
   clip: raw/clips/YYYY-MM-DD-<slug>.md
   ---
   ```
5. **Synthesise.** Create a page only when it will have ≥2 inbound links (its `wiki/index.md` line doesn't count) or names a concept the Vocabulary has a type for; otherwise update the page the material belongs on. Where new evidence supersedes a claim, keep the old claim with a dated superseded note. Cite `[[<slug>]]` and list it in the page's Sources section.
6. **Close.** Flip the item's Status `approved` → `ingested` **before** the commit, so a re-run makes zero commits. Log `ingest | <slug>`; commit per source, or per batch of related sources.

## End of wave

When nothing in the file is still `approved`:
- Set the wave's `status: ingested` and fill `## Ingest report`:
  ```markdown
  ### Pages created
  - wiki/<folder>/<slug>.md — <why>

  ### Pages updated
  - wiki/<folder>/<slug>.md — <what was added>

  ### Findings
  <cross-source consensus; disagreements worth a debate page; expected absences>

  ### Candidate follow-up
  - <topics, prospects or waves the material surfaced>
  ```
- Registry: add a row for each new publisher, with the candidate's tier; bump the Stubs count of each publisher that got new stubs in the wave, where its row has that column; append each domain under the file's `## Hard rejects` (older files may lack it) that is not yet on the Reject list, as `domain — reason — <wave slug>`.
- Log `ingest | close wave <topic>` with counts (ingested, skipped, pages created, pages updated), then commit the curation file, the registry and the log as `ingest: close wave <topic>`.

## Campaigns

Only for a wave too big for one session; earlier campaign plans in `raw/plans/`, if any, are worked examples. Fetch subagents (≤5 at a time, no nesting) write only their own clips, stubs and digests (digests in their own folder under `raw/plans/`, so they survive a pause): no git, no shared files. One writer then synthesises in dependency-ordered batches. Per batch: a gate (`bash "${CLAUDE_SKILL_DIR}/../kb-lint/scan.sh" <vault> <stale-days>` clean, each new page ≥2 inbound (index line excluded), 3 claims spot-checked against clips); the coordinator alone edits index, log, registry and curation file, and flips that batch's Statuses in that batch's commit; one commit; a handoff note in `raw/plans/` that survives compaction. Clips are ground truth; digests are data.
