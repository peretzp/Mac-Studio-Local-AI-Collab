# RAG Fleet Practices — adapting 2026 RAG doctrine to our substrate
*The Tower, dispatcher seat, anvil — 2026-08-29. Sources at bottom; recon was read-only.*

## 1. Verdict (one page)

We are one human + a handful of agents over a corpus of roughly: 229 session logs (FTS5-indexed),
7,860 prompts (FTS5, rich metadata incl. machine/cwd/branch), 408 concern rows, 146 memory files
carrying ~117 `[[wiki-links]]`, ~930 voice memos, one Obsidian vault, 10 repos. That is a *small,
high-metadata, fast-churning* corpus — the opposite of the enterprise case most RAG advice targets.

**Honest call: FTS5 keyword search already wins most of our queries.** Our retrieval is dominated by
known-item lookups — exact tokens like `beeper-port`, `NCT07393282`, `CL-0-KLI6SD-1`, a person's
name, a port number. 2026 hybrid-search literature itself concedes BM25/keyword beats dense vectors
on exact rare terms, IDs, and named entities; embeddings win on paraphrase and conceptual recall.
Our corpus is ~90% the first kind. Practitioner guidance (Towards Data Science, VentureBeat, 2026)
is blunt: small/flat corpora with single-fact queries do not repay embedding infrastructure, and
GraphRAG earns its cost only on multi-hop questions over a large entity-rich corpus.

Where embeddings DO pay here, narrowly:
- **Voice memos (MemoryAtlas)** — ASR paraphrase + Russian/English code-switching defeats keyword
  search ("parrots" = Peretz). Semantic search is the right tool. **FIT-NOW.**
- **Memory/feedback files** — "what did we decide about over-caution?" is conceptual, not lexical. **LATER.**
- **Cross-repo doc retrieval** — only after keyword unification fails us. **LATER.**

Where the graph pays: we already HAVE a proto-knowledge-graph (wiki-links, concern `to:`/`until:`/`log:`
fields, SERVICES.json ownership, people-keys). The win is *materializing* those edges queryably —
in SQLite, not Neo4j. **Neo4j server: SKIP** — JVM daemon violates the no-new-daemons sustainability
doctrine, and the GraphRAG Manifesto's 3x-accuracy claims come from enterprise corpora, not 150 files.

Patterns adopted vs skipped, summary:
- **FIT-NOW** — unified cross-store keyword search (RRF-fused); edges table in SQLite; voice-memo embeddings.
- **LATER** — hybrid BM25+vector merge; cross-encoder/LLM rerank; repo-doc embeddings; agentic multi-retriever.
- **SKIP** — Neo4j/JVM server; Microsoft GraphRAG community summaries (LLM-cost, corpus too small);
  sophisticated chunking (our docs are already small — chunk = file or `##` section); hosted vector DBs.

## 2. The Neo4j/GraphRAG inspiration, distilled for a filesystem

Neo4j's pitch (use-cases/ai-systems): knowledge graphs give AI accurate recall, explainable
reasoning paths, and durable agent memory — "agentic AI without a knowledge graph is like a
self-driving car with no GPS map." Their construction recipe: model nodes/relationships, then merge
structured data + entities extracted from text. Their key distinction: **domain graph** (your world
model) vs **lexical graph** (document structure). We already have both, unmaterialized:

**Entities (nodes):** people (`memory/person_*.md`, people-keys registry) · machines (SERVICES.json)
· repos (`~/repos/*`) · concerns (`concerns.jsonl` ids) · efforts/projects (`project_*.md`, vault
Efforts/) · sessions (session-index rows) · skills/scripts (dry-erase `used`).

**Edges (already on disk as data, free to extract):**
- `[[wiki-link]]` in memory files + vault → `mentions/relates-to`
- concerns `to:` → `assigned-to (agent|machine|person)`; `until:` → `blocked-by-condition`; `log:` → `evidenced-by (file)`
- git history per repo → `person-touched-file`, `files-co-change`
- prompts.db `cwd`/`machine_name` → `session-worked-in (repo, machine)`
- SERVICES.json → `machine-owns-service`; dry-erase WIPED `--placed` → `answer-lives-at`

**Queries this unlocks that FTS5 cannot express** (recursive CTE in SQLite handles all of them):
- *Neighborhood:* "everything within 2 hops of Boris" — concerns, sessions, files, people.
- *Multi-hop:* "which open concerns block work assigned to galen, and which session last touched their evidence files?"
- *Orphans:* memory files with zero inbound links (candidates for the dry-erase atrophy pass).
- *Provenance paths:* "how does repo biostack connect to Rachael?" — the explainability Neo4j sells.

## 3. Adoption ladder (each rung ≤ one weekend, local-only, no new daemons)

**Rung 0 — what exists (already a hybrid retrieval system, name it as such):** two FTS5 stores
(session-index.db, prompts.db) + structured filters (concerns.jsonl jq, dry-erase, people-keys) +
rg/grep over memory, vault, repos. This is BM25-class sparse retrieval with metadata filtering —
2026 best practice's first stage, already paid for.

**Rung 1 — `fleet-search`, one query across all stores (FIT-NOW).** A single CLI
(`~/.local/bin/fleet-search`) that fans one query to: sessions FTS5, prompts FTS5, concerns.jsonl,
memory/ + vault rg, dry-erase WIPED — and merges by **Reciprocal Rank Fusion** (rank-based, since
the scores are incomparable — the exact reason 2026 guides use RRF). Zero new deps, pure keyword.
Biggest single win available: today four search interfaces exist and no agent queries all four.

**Rung 2 — `graph.db`, the edges table (FIT-NOW).** One SQLite file, `nodes(id,type,path)` +
`edges(src,dst,rel,evidence)`. Loader script (rebuildable, like session-index) extracts the edge
sources listed in §2. Query via recursive CTEs; expose as `fleet-graph neighbors <entity>` and
`fleet-graph path <a> <b>`. This is 80% of GraphRAG's value at 0% of Neo4j's operational cost.
Rebuild-from-source doctrine (like prompt-store) means it can never rot silently.

**Rung 3 — local embeddings for the semantic corpora only (voice FIT-NOW, rest LATER).**
`sqlite-vec` extension (brute-force scan — fine at our scale, ~10^3–10^4 vectors; benchmarks
only punish it at 10^6+) + `nomic-embed-text` via hearth Ollama (free, local, already doctrine).
Batch cron-less job run at park time, no daemon. Order of onboarding: MemoryAtlas transcripts →
memory/*.md → repo READMEs+docs. LanceDB is the fallback if scan latency ever hurts; Neo4j never.

**Rung 4 — true hybrid + rerank (LATER, only when Rung 3 shows misses).** RRF-merge FTS5 and
sqlite-vec lists inside `fleet-search`; optional rerank of top-20 by a small local model via
LiteLLM :4000 acting as cross-encoder-by-prompt. Agentic retrieval (LLM picks retriever per query,
Neo4j's "agentic traversal") is what we ALREADY do manually — codify router rules in the CLI
(`id-shaped → FTS5 only; question-shaped → hybrid; entity-shaped → graph neighbors first`).

**SKIP rungs, permanently unless scale changes:** Neo4j server (JVM daemon, one human's laptop
fleet); MS GraphRAG community-summary pipeline (indexing LLM spend > corpus value); semantic
chunkers (files are the chunks); cloud vector stores (data is personal/medical/legal).

## 4. Repos specifically (10 in ~/repos) + doc placement as retrieval engineering

**Repo-level RAG, minimum viable (FIT-NOW):** a generated `~/repos/REPO-MAP.md` — per repo: one-para
purpose, key entrypoints, owners, links to AGENTS/CLAUDE/COUNCIL files — rebuilt by script from
READMEs. That single file *in the retrieval path of every agent* outperforms an embedding index at
our repo count; ten READMEs fit in one context window (the "LLM wiki" pattern beats RAG at this
scale). **Cross-repo embedding index (LATER):** Rung 3 job over `README* docs/** *.md` per repo,
tagged `repo=` for filtered retrieval — adopt when REPO-MAP misses start being felt, not before.

**Doc-placement practices that make ANY retrieval work (FIT-NOW, costs nothing):**
- **The dry-erase doctrine IS a RAG practice.** "RAISED ≥ 3 → move the answer to where the asking
  happens" is literally retrieval optimization: relocating content into the path the query already
  takes. `wipe --placed <path>` is writing the retrieval index by hand. Name this equivalence in
  the board's docs so agents treat placement as index maintenance, not tidying.
- **Front-load exact tokens.** BM25/FTS5 lives on rare exact terms — put ids, ports, hostnames,
  case numbers in headers and first lines (we already do this well in MEMORY.md; extend to READMEs).
- **Self-contained `##` sections.** Chunking best practice says a chunk must survive without its
  neighbors; since our chunk = section, never write a section whose meaning depends on the one above.
- **One answer, one home, links elsewhere.** Duplicated answers drift (the 8/10 false-broadcast
  lesson); retrieval should find one canonical row. Registries (SERVICES.json) outrank prose — the
  "JSON wins" rule is our version of RAG's structured-source-priority.

## Sources
Fetched: https://neo4j.com/use-cases/ai-systems/ · https://neo4j.com/blog/genai/what-is-graphrag/ ·
https://neo4j.com/blog/genai/graphrag-manifesto/
Via WebSearch (2026): denser.ai hybrid-search-for-rag · digitalapplied.com hybrid-search-bm25-vector-reranking-reference-2026 ·
towardsdatascience.com do-you-really-need-graphrag · venturebeat.com stop-graphing-everything ·
shaharia.com embedded-vector-db comparison (sqlite-vec vs LanceDB) · microsoft.github.io/graphrag ·
mindstudio.ai karpathy-llm-wiki-pattern · arxiv 2604.01733 (BM25→corrective-RAG benchmark).
