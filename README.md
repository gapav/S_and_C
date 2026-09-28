# LLM Wiki

A starter personal knowledge base scaffold for the LLM Wiki pattern: raw sources are kept immutable, while an LLM-maintained markdown wiki accumulates summaries, entities, concepts, questions, and synthesis over time.

## Structure

- `raw/` — source documents you add. The LLM reads these but should not modify them.
- `raw/assets/` — downloaded images and attachments referenced by sources.
- `wiki/` — LLM-generated and LLM-maintained markdown pages.
- `wiki/index.md` — content catalog and navigation entry point.
- `wiki/log.md` — chronological append-only activity log.
- `AGENTS.md` — operating instructions for LLM agents maintaining this wiki.

## Basic workflow

1. Add a source to `raw/`.
2. Ask the LLM agent to ingest it.
3. Review the generated source summary and updates to related pages.
4. Ask questions against `wiki/`; valuable answers can be filed back into `wiki/questions/`.
5. Periodically ask the LLM to lint the wiki for stale claims, missing links, orphan pages, and contradictions.

