# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not a software project**. It is a personal Obsidian vault implementing the "LLM Wiki" pattern for strength & conditioning / sport performance knowledge. There is no build, test, lint, or run step — work is done entirely by reading and editing markdown files.

The vault lives inside an iCloud-synced Obsidian directory, so the working-directory path contains spaces and special characters (`S&C`, `iCloud~md~obsidian`). Always quote paths in shell commands.

## Authoritative operating instructions

**Read `AGENTS.md` first.** It is the operating schema for this vault and defines the layer model, directory conventions, ingest/query/lint workflows, linking style, page style, and the log format. Treat it as the source of truth; this file only adds higher-level orientation. If conventions need to change, update `AGENTS.md` (not just this file).

## Layer model

Two layers, with a hard boundary between them:

- `raw/` — **immutable** source material (podcast transcripts, presentation transcripts, articles). Read but do not edit unless the human explicitly asks. New sources are added here by the human.
- `wiki/` — **LLM-owned** synthesis layer. Create, update, cross-link, and maintain pages here. Structured into `sources/` (one page per ingested raw file), `entities/` (people, organizations, products), `concepts/` (topics, mechanisms, frameworks), and `questions/` (durable answers to user queries). Three top-level files anchor it: `index.md` (catalog), `overview.md` (synthesis), `log.md` (append-only activity record).

The cardinal rule: every claim in `wiki/` should be traceable to a source page, and every source page should link out to the entities/concepts it touches. Avoid orphan pages — link new pages from `index.md` and at least one relevant topic page.

## Domain context

The wiki's subject matter is sport performance training — speed, acceleration, agility, strength development, conditioning, and field/court transfer for sports like basketball, soccer, and handball. The wiki is meant to compare coaching models and extract programming principles, not to be a generic exercise library or transcript archive. When synthesizing, preserve disagreement between coaches rather than flattening them into one universal system, and distinguish claims, interpretations, and open questions.

## Linking and naming

- Obsidian-style wiki links only: `[[Page Name]]` (Title Case, descriptive, stable filenames with `.md`).
- When a concept or entity recurs, give it a dedicated page rather than inlining the description.

## Logging

Any ingest, substantive query that changes the wiki, lint pass, or schema/maintenance edit should append one entry to `wiki/log.md` using the heading format defined in `AGENTS.md` (`## [YYYY-MM-DD] type | Title`). Today's date is available in the conversation context — use it as the absolute date.
