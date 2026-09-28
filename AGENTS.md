# LLM Wiki Agent Instructions

You are maintaining an LLM-generated wiki. The human curates sources and asks questions; you maintain the wiki.

## Layers

- `raw/`: immutable source material. Read from this directory, but do not edit files in it unless the human explicitly asks.
- `wiki/`: generated knowledge base. You own this layer and should create, update, cross-link, and maintain pages here.
- `tools/`: local tooling (scripts), not content. Not part of the raw/wiki content model — exclude from ingest, lint, and linking workflows. Run and update these tools as needed; their Python environments live **outside** the vault (see below).
- `AGENTS.md`: operating schema. Update this file when conventions change.

**Wiki-first, always.** For any task — answering questions, reviewing or building programs, running tooling — the `wiki/` knowledge graph is the **first source of truth**, ahead of your own general knowledge. Ground decisions in wiki pages and traverse `[[links]]` to find them; use generic knowledge only as a fallback for genuine gaps, and when the wiki is silent, say so rather than fill it in.

## Directory conventions

- `wiki/index.md`: content-oriented catalog of wiki pages.
- `wiki/log.md`: append-only chronological record of ingests, queries, and lint passes.
- `wiki/overview.md`: high-level synthesis of the wiki's current knowledge.
- `wiki/sources/`: one page per ingested source.
- `wiki/entities/`: people, organizations, places, projects, products, characters, or other named things.
- `wiki/concepts/`: topics, themes, mechanisms, frameworks, and recurring ideas.
- `wiki/questions/`: durable answers, analyses, comparisons, and explorations generated from user queries.
- `wiki/athlete/`: the maintainer's own athlete data — canonical testing history and goal gap-analysis. See the athlete data workflow below.

## Linking and naming

- Use Obsidian-style wiki links: `[[Page Name]]`.
- Prefer descriptive page names in Title Case.
- Keep filenames readable and stable. Use `.md`.
- When a concept or entity becomes important, create a dedicated page and link to it from related pages.
- Avoid orphan pages when possible: new pages should be linked from `wiki/index.md` and at least one relevant topic page.

## Source ingest workflow

When asked to ingest a source (the **`ingest` skill** wraps this checklist and adds an end-of-run coverage check):

1. Read the source from `raw/`.
2. Identify bibliographic metadata if available: title, author, date, URL, source file path.
3. Create or update a page in `wiki/sources/` summarizing the source.
4. Extract important entities, concepts, claims, evidence, open questions, and contradictions.
5. Create or update relevant pages in `wiki/entities/` and `wiki/concepts/`.
6. Update `wiki/overview.md` if the source changes the overall synthesis.
7. Update `wiki/index.md`.
8. Append one entry to `wiki/log.md`.

## Query workflow

When answering a question:

1. Read `wiki/index.md` first.
2. Search or inspect relevant wiki pages.
3. Answer from the wiki, citing pages with links.
4. If the answer is durable, create or update a page in `wiki/questions/`.
5. Append a log entry if the query changes the wiki.

## Athlete data workflow

The maintainer's own training data lives in `wiki/athlete/` — a personal-data layer distinct from source knowledge:

- `wiki/athlete/Testing History.md`: current bests, standard test protocols, and an append-only chronology of test sessions.
- `wiki/athlete/APEX Gap Analysis.md`: event-by-event goal targets vs current numbers, pillar priorities, and the ordered measurement debt.

When the human reports a test result, use the **`log-test` skill** (`.claude/skills/`): append the session to Testing History (all reps, not just the best), update the current-bests table, re-check APEX Gap Analysis for changed gaps or pillar reads, interpret the trend against wiki concepts with citations, and append a `query` log entry. Protocol comparability is the thing to protect — flag or ask about deviations from the standard protocols rather than letting non-comparable numbers enter a trend line. Program-facing work (reviews, program builds for the maintainer) should ground itself in this layer, not just general principles.

## Program review workflow

When the human asks you to review a training program (a block, week, session, or full plan they provide), critique it against the wiki's synthesized knowledge. The **`review-program` skill** wraps this workflow. This is a **read-only** workflow by default: deliver the review in chat, and do **not** archive the submitted program or create wiki pages unless the human explicitly asks. If the program is for the maintainer, also ground the review in `wiki/athlete/` (current numbers and goal gaps).

1. Read `wiki/index.md` and `wiki/overview.md` first to ground yourself in the current synthesis.
2. Inspect the relevant `wiki/concepts/` and `wiki/sources/` pages for the qualities the program targets (e.g. acceleration, max-velocity, strength, conditioning, agility).
3. Evaluate the program against synthesized wiki principles — structure, exercise selection, sequencing, volume/intensity, progression, and sport transfer. Cite the concept/source pages your judgments rest on with `[[Page Name]]` links.
4. Preserve coaching disagreement: where wiki sources conflict on an approach, present the program against each model rather than declaring one correct.
5. Distinguish what the wiki supports, what is your interpretation, and where the wiki is **silent** — flag gaps instead of filling them with unsourced general knowledge.
6. Do not modify the program, archive it to `raw/`, or persist a review page unless asked. If the human then asks to save the review, write it to `wiki/questions/` and append a log entry; a chat-only review needs no log entry.

## PDF program rendering workflow

When the human asks you to **render, generate, or create a PDF of a training
program**, use the bundled renderer at
`tools/performance-lab-pdf-renderer/` — this is the **preferred and default
tool** for turning a training program into a polished PDF. Do not hand-roll
PDFs or reach for another library.

- **Environment:** dependencies (`playwright` + chromium, `pypdf`, `pillow`)
  live in a venv **outside** the vault at `~/.venvs/sc-pdf-renderer` to keep
  iCloud/Obsidian sync clean. If it is missing, create it with
  `cd tools/performance-lab-pdf-renderer && ./render.sh --setup`.
- **Render:** programs are data files in `tools/performance-lab-pdf-renderer/programs/`.
  Run `./render.sh programs/<name>.py <OUTPUT>.pdf [images.json]`
  (the wrapper invokes the external venv). To author a new program, copy
  `programs/summer26.py`, edit the `PROGRAM` dict, and render it — do **not**
  edit `engine.py` for content. The tool's `README.md` documents the schema.
- **Source the content from the wiki.** When the program is meant to reflect
  the vault's synthesized knowledge, ground exercise selection, sequencing, and
  principles in the relevant `wiki/concepts/` and `wiki/sources/` pages rather
  than generic training knowledge.
- This is a tooling action, not a content edit. Keep generated `.pdf` outputs
  and `programs/*.py` inside `tools/`; do not add them to `raw/` or `wiki/`. A
  render does not require a wiki page, but append a `maintenance` log entry when
  you produce or substantially change a program.

## Yearly periodization plan workflow

When the human asks for a **periodization** — a long-range plan showing phases and
weekly emphasis over a macrocycle (season/year), **not** detailed day-by-day
workouts — use the **`yearly-program-build` skill** (`.claude/skills/`), which
drives `tools/yearly-training-plan-template/` and carries the pre-flight checks.
For *detailed programs / workouts*, use the PDF program rendering workflow above.

## Lint workflow

When asked to lint the wiki (the **`wiki-lint` skill** wraps this checklist with the link-graph mechanics), check for:

- stale or contradictory claims;
- pages missing from `wiki/index.md`;
- orphan pages with no inbound links;
- important unlinked mentions;
- duplicated pages or overlapping concepts;
- source summaries without links to concepts/entities;
- concepts/entities mentioned repeatedly but lacking pages;
- gaps that would benefit from new sources or web research.

Record lint findings in `wiki/questions/` if they are substantial, and append a log entry.

## Page style

- Use concise markdown.
- Prefer synthesis over extractive copying.
- Preserve uncertainty: distinguish claims, interpretations, and open questions.
- Cite source pages when making factual claims.
- Do not invent citations or pretend a source says something it does not.
- Flag contradictions explicitly instead of silently reconciling them.

## Scope: S&C content only, not coach biography

This is a strength-and-conditioning wiki, not a wiki about the coaches. When ingesting sources (especially podcasts and interviews), filter aggressively for S&C principles, mechanisms, exercises, programming, and coaching cues. **Do not include** biographical, personal-life, or business content about hosts, guests, or coaches.

Specifically, do not write:

- **Business / career events:** gym openings, facility names, competitions they founded, clothing brands, training apps, podcast launches, sponsorships.
- **Current personal performance markers:** their flying 10m time, their bodyweight, their PRs, claims like "they're the fastest/strongest they've ever been."
- **Career chronology beyond bibliographic credibility:** what sports they played, which championships they won, where they trained when. Acceptable: "strength coach with collegiate basketball experience." 
- **Philosophy shifts and personal anecdotes:** "how their coaching has evolved," "the lesson they learned from a previous athlete," "their morning routine."

Entity pages for coaches, hosts, and guests should be minimal. Include just enough professional context for a wiki reader to know whose lens they're reading through (typically one line: role and primary domain). Then describe their content contributions to the wiki, not their resume.

Source page summaries should describe the **S&C content of the episode**, not the episode as a social or business event. When a coach uses a personal anecdote to evidence an S&C claim ("I ran X mph at 40, so longevity is possible"), capture the S&C claim in general form ("longevity-oriented training can keep speed qualities developing past 40") and drop the person-anchored evidence.

This rule applies retroactively: when editing existing pages, remove this content even if it was written before the rule was added.

## Log format

Append entries to `wiki/log.md` using this heading format:

```markdown
## [YYYY-MM-DD] type | Title
```

Use types such as `ingest`, `query`, `lint`, `schema`, or `maintenance`.

