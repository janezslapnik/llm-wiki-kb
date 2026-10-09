# Examples

Sample files from a small knowledge base on home sourdough baking: one intake wave, three sources ingested, a lint pass. Use them to see what the skills write before you run them yourself.

| File | Path in the vault | Written by | What it shows |
|---|---|---|---|
| `curation-wave.md` | `raw/curation/2026-10-08-sourdough-starter-maintenance.md` | `kb-intake` (candidates, triage), `kb-ingest` (statuses, ingest report) | One wave: the gap, the candidates with tier and reasons, the triage decisions and what the ingest found. |
| `source-stub.md` | `wiki/sources/perfect-loaf-store-sourdough-starter.md` | `kb-ingest` | The frontmatter-only record of one source. |
| `technique-page.md` | `wiki/techniques/starter-maintenance.md` | `kb-ingest` | A page built from three sources, with section locators and a disagreement between them. |
| `log-excerpt.md` | `wiki/log.md` | `kb-ingest`, `kb-lint` | One entry per operation. |

In this wave the operator let Claude triage on its own, so the curation file quotes that permission in its `## Triage` section. Normally you approve the candidates yourself.

## Not included

- Clips (`raw/clips/`). They hold the full text of third-party pages, so they are not redistributed. The `clip:` path in the stub and the clip paths in the log point to files that are not here.
- The rest of the vault. Some `[[links]]` point to pages that are not copied, such as the other two stubs and `sluggish-starter`.

The files sit flat rather than in vault layout, so `scan.sh` never reads this folder. Quotes are short and cite their source and section; quoted material belongs to its publishers.
