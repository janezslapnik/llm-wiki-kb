# llm-wiki-kb

[![CI](https://github.com/janezslapnik/llm-wiki-kb/actions/workflows/check.yml/badge.svg)](https://github.com/janezslapnik/llm-wiki-kb/actions/workflows/check.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A Claude Code template for a knowledge base that Claude builds and maintains, following Andrej Karpathy's [LLM-Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). You point it at a subject. It finds sources, asks you which to keep, clips them, writes linked wiki pages, audits the wiki and answers questions from it with citations. See [`examples/`](examples/) for files from a real run on home sourdough baking.

## Why this one

- **You approve every source.** Nothing is clipped until you say so.
- **Tiered source registry.** Publishers are ranked Tier 1 to 3, with a reject list, so a source's authority is explicit.
- **One git commit per operation.** Every change can be reviewed and reverted.
- **Plain Markdown in git.** No app, no server, no database. Read it in any editor or in Obsidian.
- **Lint scanner and CI.** `scan.sh` checks frontmatter, broken links, orphans and stale sources.

Four generic Claude Code project skills do the work. All per-KB values live in the `## Instance` and `## Vocabulary` sections of `.claude/CLAUDE.md`, plus one line in kb-query's description.

- `kb-intake` finds and triages candidate sources into a curation file.
- `kb-ingest` turns approved sources into clips, source stubs and wiki pages.
- `kb-lint` audits the wiki (`scan.sh` does the mechanical checks) and applies fixes on request.
- `kb-query` answers questions from the wiki with a cited synthesis.

## How it's laid out

There are two git repos, one inside the other.

- **Harness**: this outer repo, with the skills and the schema (`.claude/CLAUDE.md`). It tracks the template, so you can pull skill fixes later.
- **Vault**: a nested repo with `raw/` (clips, curation files, prospects) and `wiki/` (the pages). It is its own repo because the KB content has its own history and can stay private while the harness follows upstream. The skills call it `<vault>`, and bare `raw/…` and `wiki/…` paths in the skills are inside it.

```
.claude/CLAUDE.md          schema: Instance, Vocabulary, Pages, Record
.claude/skills/kb-*/       the four skills (kb-lint/scan.sh is the scanner)
<vault>/raw/               clips, assets, curation, prospects, plans
<vault>/wiki/              overview, index, log, source-registry, sources/, synthesis/
                           (the template ships <vault> as vault/; my-kb/ after the Quick start)
```

Terms the skills use (the full list is under `## Terms` in `.claude/CLAUDE.md`):

- **Operator**: you, the person who approves sources and schema changes.
- **Wave**: one curation file's batch of candidates, from search through triage to ingest.
- **Prospect**: a gap worth searching later, filed in `raw/prospects/`.
- **Clip**: the cleaned full text of a source in `raw/clips/`. Page claims rest on clips.
- **Stub**: a frontmatter-only page per source in `wiki/sources/`, pointing at its clip. Its `published` date may be `unknown`.

## Requirements

Claude Code, git, bash, ripgrep with PCRE2 (`rg --pcre2-version`), and fd (`fdfind` from Debian and Ubuntu's `fd-find` package is accepted). Tested on Linux.

<details>
<summary>Other tools and optional extras</summary>

- `scan.sh` also uses awk, sed, grep, sort, comm, paste, cut, basename, mktemp, and GNU or BSD date. It exits with code 2 and a message when fd, ripgrep (with PCRE2) or awk is missing.
- Optional: search or fetch tools named in your Instance block (for example a search MCP server), and `markitdown` for turning PDFs and Office files into Markdown.

</details>

## Quick start

**From "Use this template" or a clone.** The vault ships tracked in this repo, but it becomes its own repo. Untrack it first.

```bash
git clone <your-repo-url> my-kb-harness && cd my-kb-harness
git rm -r -q --cached vault
mv vault my-kb
sed -i 's#^<vault>/$#my-kb/#' .gitignore    # macOS: sed -i '' 's#^<vault>/$#my-kb/#' .gitignore
git -C my-kb init
```

**From a plain copy** (no `.git`): run the same steps, but `git init` in place of `git rm -r -q --cached vault`.

Then fill in the placeholders:

1. `.claude/CLAUDE.md`: replace each `<...>` placeholder value in `## Instance` and `## Vocabulary`. In the Vault bullet, replace `<VAULT-FOLDER>` with `my-kb`; the lowercase `<vault>` is the skills' name for that folder and stays. Replace the Inbox bullet's whole `<OPTIONAL ...>` value with your command, or delete the bullet if you don't use one.
2. `.claude/skills/kb-query/SKILL.md`: replace `<SUBJECT>` and `<one-line scope>` in the `description:` line. Keep the line in double quotes and put no `"` inside it. Leave `<source>` as it is; it is literal text. Change nothing else in the skills.
3. `my-kb/wiki/source-registry.md`: set `last-refreshed:` and seed some Tier 1, 2 and 3 publisher rows.
4. `my-kb/wiki/overview.md`: write the scope paragraph and the Sources line. Set the `updated:` dates in the seed pages and date the first entry in `wiki/log.md`.

Commit both repos:

```bash
git -C my-kb add -A && git -C my-kb commit -m "bootstrap: vault seed"
git add -A && git commit -m "chore: instantiate my-kb"
```

`git status` should now be clean in both. To run the lint scan by hand: `bash .claude/skills/kb-lint/scan.sh my-kb 90` (vault folder, stale days, optional `YYYY-MM-DD` for today).

The template's CI workflow (`.github/workflows/check.yml`) runs shellcheck and a YAML check on the skills. Its scan smoke test only runs while a `vault/` folder is tracked, so it skips itself in your instance.

## How the skills chain

```
intake -> curation file -> your triage reply -> ingest -> lint -> query
```

Start Claude Code in the harness (`cd my-kb-harness && claude`) and talk to it:

- "find sources on X" runs `kb-intake`. It writes `raw/curation/<date>-<topic>.md` and shows numbered candidates.
- "approve 1 3" marks those items approved and the rest rejected.
- "ingest the approved wave" runs `kb-ingest`: one clip, one source stub, page updates, a log entry and a commit per source.
- "add this URL to the KB" skips discovery. A direct request is its own approval.
- "lint the KB" runs `kb-lint` and writes a dated report. "apply the lint fixes" works through it, one category per commit.
- Any question about the subject runs `kb-query`.

Item statuses in a curation file are `pending`, `approved`, `rejected`, `ingested`, `skipped-duplicate` and `skipped-unreachable`. [`examples/`](examples/) shows each step's output: a curation file, a source stub, a wiki page and a log excerpt.

## Customising

In `.claude/CLAUDE.md`, fill only the `## Instance` and `## Vocabulary` sections. Leave `<vault>` in the skills as written. Notes on a few Instance values:

- **Stale** takes a number of days. A living-doc source fetched longer ago than that is stale. `0` turns the check off.
- **Paywalled sources** set the policy for paywalled candidates, and override kb-intake's Borderline default.
- New page types go into Vocabulary. The skills propose them to you before using them.

## Optional inbox

If you save links on the go, provide a command and put it in the Inbox bullet. It must write one `raw/inbox/<name>.md` per saved link, with frontmatter `type: inbox`, `url:`, `captured: YYYY-MM-DD` and `status: new`, and an optional note in the body. "process my inbox" triages them, and processed items get `status: done`. Uncomment the inbox lines in the vault's `.gitignore`. No inbox tool ships with this template.

## Updating and drift check

Pull later skill fixes from the template and compare:

```bash
git remote add template <template-repo-url>
git fetch template
git diff template/main -- .claude/skills
```

This compares file trees, so it works even though "Use this template" gives your repo an unrelated history. The only expected difference is the `description:` line of `kb-query/SKILL.md`. To take the template's skills and keep your kb-query file, then commit:

```bash
git checkout template/main -- .claude/skills
git checkout HEAD -- .claude/skills/kb-query/SKILL.md
git commit -m "chore: update skills from template"
```

If kb-query itself changed upstream, copy your `description:` line into the new file instead. The notes on the template's GitHub Releases page list every skill-file change.

## Security

- The schema tells Claude to treat fetched pages and clips as data, never as instructions. That lowers the risk of prompt injection but does not remove it.
- Keep the vault private, and give it its own private remote for backup. Clips are full copies of what you fetch, so never clip documents that carry personal data.
- Review the commits before you push. One commit per operation keeps each review small.
- Keep permission prompts on for `rg`, `fd`, `git` and `curl`. The FAQ lists narrow allow rules.

## FAQ

- **Why not WebFetch for clips?** It returns a model summary, not the page. Use it for gists only, and use `curl` plus `markitdown` (or your own tools) for clips.
- **Too many permission prompts?** Allow narrow forms like these in `.claude/settings.local.json`:

  ```json
  {"permissions": {"allow": ["Bash(git -C my-kb status:*)", "Bash(git -C my-kb add:*)", "Bash(git -C my-kb commit:*)"]}}
  ```

  Approve the lint scan when prompted; `scan.sh` only reads the vault. Write your vault folder literally after `-C`: a `*` there would also match `-c <option>`. Don't allow `rg`, `fd` or `git` broadly. `fd -x`, `rg --pre` and `git -c` would let fetched content run commands without a prompt.
- **Can I browse the wiki in Obsidian?** Yes. Open the vault folder as an Obsidian vault; its `.gitignore` already skips Obsidian's per-machine files.
- **Can I upload these skills to claude.ai?** They are Claude Code project skills. They expect the repo layout and a shell.
- **Contributing.** Issues and pull requests are welcome.

## License

MIT. See [LICENSE](LICENSE).
