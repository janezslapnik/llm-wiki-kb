---
name: kb-query
description: "Answers questions from the <SUBJECT> knowledge base, a wiki on <one-line scope>, with a cited synthesis. Use for how-do-I, what-is, what-does-<source>-say, compare-X-vs-Y, and 'what does the KB have on X' questions in that domain. Not for adding material (kb-intake) or auditing (kb-lint)."
---

# kb-query

Answers from the wiki as a synthesis, not a retrieval.

1. **Find.** Search `wiki/index.md` with `rg`, read the relevant pages, and follow their links where the question crosses concepts. For detail beyond a page, read the source's local clip (`clip:` on its stub) rather than the network.
2. **Register.** The page is already the better reference, so the answer adds value by interpretation: synthesise, don't reformat a page. Pick one framing as the spine and use the others in support. Prose by default; bullets only for genuinely enumerated content, tables only for real comparisons. Length is functional: the smallest answer that covers the question.
3. **Attribute** in prose by author or name ("<Author> argues…"), then list the pages once at the end:
   `**Sources:** [[<source-slug>]] (<author>), [[<page-slug>]] (wiki synthesis).`
   Inline `[[slug]]` only where the prose names a concept worth linking.
4. **Recap.** A substantive answer ends with an `**In short:**` block of 3–6 one-line bullets, before the Sources line, stated once; short answers skip it.
5. **Honesty.** Answer from the KB, and label anything from your own knowledge as such. When the KB lacks the answer or covers it partly, say what is missing and where it might be found (the upstream URLs of related stubs).
