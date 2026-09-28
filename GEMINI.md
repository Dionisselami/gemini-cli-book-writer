# Book writing with Proseify

The `proseify` MCP server is connected (see `settings.json` / `~/.gemini/settings.json`). It is a
corpus and a genre-discipline layer: **you** write the prose, it supplies the literature, the genre
recipes and the scoring gate. It calls no model of its own, so there is no second token bill.

## Order of operations

1. `list_genres` -> `get_genre_recipe` for the closest fit. State the recipe and the reason in one line.
2. `search_corpus` for 3-5 model passages in the target genre. Quote one line that sets the register.
3. `plan_book` **once** — premise, chapter count, word budget. Print the outline. Hold it.
4. Draft **one chapter per writing turn**. After each chapter run the edit pass below and state what
   you cut. Do not proceed to the next chapter on the assumption it will be fine.
5. `evaluate_book` before declaring the book finished. Fix flagged chapters and re-run the gate.

## Per-chapter edit pass

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard*
- stacked adverbs on dialogue tags; plain `said` is fine
- repeated sentence openings; weather openings
- three-item lists as rhythm filler
- dialogue that exists to explain the plot to the reader
- the last line turning into a metaphor for the chapter

## Files and rules

- One file per chapter: `chapters/NN-title.md`, written before the next chapter starts.
- `outline.md` holds the `plan_book` output verbatim; re-read it instead of re-planning.
- Never print or commit the Proseify key — it is read from the `PROSEIFY_API_KEY` environment
  variable.
