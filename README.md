# gemini-cli-book-writer

**Write a whole book with Gemini CLI.** Gemini plus the [Proseify](https://proseify.xyz) MCP server: a genre
recipe read before any prose exists, 2,500+ chapters of real literature to calibrate register
against, an outline, chapter-by-chapter drafting, and a scoring gate before you call it done.

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/dionisselami/proseify-mcp) [![MCP Registry](https://img.shields.io/badge/MCP_registry-io.github.Dionisselami%2Fproseify--mcp-4a3f35)](https://registry.modelcontextprotocol.io/v0/servers?search=proseify)

---

## Why this exists

Gemini CLI is a strong long-context writer with no literary tradition loaded into it. Ask for a
36-chapter gothic novel and you get something structurally correct and tonally anonymous — the same
sentence lengths, the same soft ending at every chapter break.

Proseify gives it something to aim at: per-genre pacing beats and dialogue ratios derived from the
actual tradition, plus full-text search over public-domain classics so the model calibrates against
*Dracula* instead of against its own averages.

```text
premise  →  genre recipe  →  corpus passages  →  outline  →  chapter drafts  →  evaluate_book  →  revision
```

Never let the model start at "chapter drafts". The ordering is the whole product: a genre
recipe read *before* prose beats any amount of "write like Stephen King" prompting, because it
hands the model concrete numbers — pacing beats, dialogue ratio, sentence-length profile — that
its defaults would otherwise flatten to its own house style.

---

## Quick start

### 1. Get a Proseify key

https://proseify.xyz — sign in, pick a plan (from $9/mo, or the one-time Founding Lifetime tier),
key issued on payment.

### 2. Connect the MCP server

Gemini CLI reads `mcpServers` from `settings.json`. Two things matter here: the field is
**`httpUrl`** for streamable HTTP (a plain `url` means legacy SSE and will not connect), and header
values support `${VAR}` interpolation, so the key never has to be written into the file.

```json
{
  "mcpServers": {
    "proseify": {
      "httpUrl": "https://mcp.proseify.xyz/mcp",
      "headers": { "Authorization": "Bearer ${PROSEIFY_API_KEY}" },
      "timeout": 300000
    }
  }
}
```

`settings.json.example` in this repo is exactly that block. Append it to `~/.gemini/settings.json`
(user scope) or `.gemini/settings.json` inside a book project (workspace scope), then check:

```bash
gemini mcp list          # or /mcp inside a session
```

The generous `timeout` is deliberate: `plan_book` is quick, a long chapter draft is not.

### 3. Install the writing skill

```bash
npx skills add https://proseify.xyz --skill anti-prose-slop -y
```

The MCP server gives the agent the *tools*. The `anti-prose-slop` skill gives it the *method* —
genre recipe first, corpus grounding, chapter discipline, and the per-chapter edit pass. Install
both or you get tool access and the same generic prose you had before.

### 4. Give it a premise

> Write a 30-chapter literary adventure about a cartographer's apprentice who maps a coastline that
> keeps moving. Around 60,000 words.

---

## What's in here

| File | Purpose |
|------|---------|
| `settings.json.example` | The `proseify` server block for `~/.gemini/settings.json` |
| `GEMINI.md` | Context Gemini CLI loads automatically — the writing discipline |
| `LICENSE` | MIT |

### The five-step loop

1. **Pick the genre before any prose exists.** `list_genres`, then `get_genre_recipe` for the
   closest fit. Blends are allowed — name both and let the recipe argue for one.
2. **Ground the register.** `search_corpus` (or `get_style_references`) for 3–5 model passages
   in the target genre. Read them. This is the calibration step, not decoration.
3. **`plan_book` once.** Hold the outline. Do not re-plan mid-draft; if the outline is wrong,
   fix it deliberately and say so.
4. **Draft chapter by chapter**, pausing after each for the per-chapter edit pass below.
5. **`evaluate_book` before you call it done.** Fix the chapters it flags and re-run the gate.

### The per-chapter edit pass

The failure mode of AI prose is not grammar — it is sameness. Every chapter gets struck against
this list before it counts as drafted:

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard* — put the reader in the perception instead
- stacked adverbs on dialogue tags; `said` is not a problem, `said loudly, angrily` is
- sentence openings repeated across the chapter (and the weather-opening default)
- three-item lists used as rhythm filler
- dialogue that exists to explain the plot to the reader
- simile endings: the last line becoming a metaphor for the chapter

### If you only read one paragraph

The agent's own model does all the writing — Proseify calls no LLM and stores no manuscript.
There is no token bill from us: it is a corpus, a set of genre recipes, and a pipeline.


---

## Gemini CLI notes and gotchas

- **`httpUrl`, not `url`.** With `url` you get an SSE attempt against a streamable-HTTP endpoint and
  a connection that never completes — the most common setup error in this client by a wide margin.
- **Gemini CLI sanitises tool schemas for the Gemini API.** Complex or nested parameter schemas can
  be trimmed in transit, so stick to the documented arguments of each Proseify tool rather than
  improvising extras.
- **Raise the per-server `timeout`.** `evaluate_book` on a full manuscript can exceed the default and
  looks like a hang; the call is still running.
- **Put the discipline in `GEMINI.md` or the agent improvises.** With tools connected but no
  instructions, models call `plan_book` and then ignore their own outline. The order is stated
  explicitly in that file for exactly this reason.
- **`mcp.allowed` is worth using in shared setups** — allowlisting just `proseify` keeps a stray
  config from adding servers you have not vetted. `trust: true` bypasses per-call confirmations and
  should be a deliberate choice, not a copy-paste.
- **A 60,000-word book is not one turn.** Draft chapter by chapter, keep each chapter in
  `chapters/NN-title.md`, and re-read the outline file instead of re-planning mid-draft.

| Tool | What it does |
|------|--------------|
| `list_genres` | The 11 genres the corpus covers (theatre, horror, romance, adventure, literary, mystery, gothic, sci-fi, fantasy, comedy, children's) |
| `get_genre_recipe` | Pacing beats, dialogue ratio, sentence-length profile and stylistic anchors derived from that tradition |
| `search_corpus` | FTS5 full-text search across 2,500+ chapters of public-domain classics (quoted phrases, AND/OR/NOT, wildcards) |
| `get_style_references` | Model passages from books in the target genre — the register you are aiming at |
| `plan_book` | A full chapter-by-chapter outline from a one-line premise, each beat carrying a corpus style reference |
| `write_book` | The one-shot flow: plan → draft → evaluate, in a single session |
| `evaluate_book` | Scores a draft against genre benchmarks (structure, pacing, chapter coverage, word budget) so weak chapters get revised |

## The corpus

80+ books and 2,500+ chapters of public-domain literature — Project Gutenberg and similar
sources — every chapter verified against its own file header at download time, so nothing in
the library has a copyright question hanging over it. Works from Austen, Stevenson, Hugo,
Conrad, Verne, the Brontës, Shelley and the gothic masters, grouped by genre and indexed for
full-text search.

Commercial use of what you write on top of it is safe. Full-text searchable chapter by chapter,
not a scrape of titles.

## Pricing

| Plan | Price | Rate limit |
|------|-------|-----------|
| Starter | $9/mo | 120 req/min |
| Pro | $19/mo | 400 req/min |
| Studio | $49/mo | 9999 req/min |
| Founding Lifetime | one-time | 400 req/min, all genres |

Sign in at https://proseify.xyz, pick a plan, and the key is issued the moment the purchase
clears. Cancel from https://proseify.xyz/account; 14-day refund window
(https://proseify.xyz/refunds).

## Other clients

Same Proseify server, same workflow, different config file. The cookbook is the method itself.

- [claude-book-writer — write a book with Claude Code](https://github.com/Dionisselami/claude-book-writer)
- [chatgpt-book-writer — write a book on your ChatGPT plan](https://github.com/Dionisselami/chatgpt-book-writer)
- [cursor-book-writer — write a book in Cursor](https://github.com/Dionisselami/cursor-book-writer)
- [copilot-book-writer — write a book in VS Code with Copilot](https://github.com/Dionisselami/copilot-book-writer)
- [windsurf-book-writer — write a book in Windsurf](https://github.com/Dionisselami/windsurf-book-writer)
- [claude-book-cookbook — recipes for writing a whole book with Claude](https://github.com/Dionisselami/claude-book-cookbook)

## License

MIT for everything in this repository. The corpus texts themselves are public domain.

## Disclaimer

Unofficial. This repository is not affiliated with, endorsed by, or sponsored by Google, Gemini or Google DeepMind. Client names appear only to describe which configuration file and transport the instructions are for. Proseify is an independent product — https://proseify.xyz.

Keep your Proseify key in local agent config or an environment variable. Never commit it, print it, log it, or paste it into a repository file.

## Links

- **Sign up / key issuance:** https://proseify.xyz
- **Filled-in config for your key:** https://proseify.xyz/agent
- **FAQ (ownership, KDP, what an MCP server is):** https://proseify.xyz/faq
- **Server repo & MCP registry entry:** https://github.com/Dionisselami/proseify-mcp
- **Support:** support@proseify.xyz
