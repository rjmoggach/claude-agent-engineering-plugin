---
name: knowledge-graphs
description: >-
  Build knowledge graphs as agent memory through the 9-stage pipeline — scope test,
  representation choice, ontology modeling, entity/relation/event extraction, quality
  gate, fusion, and serving to LLMs via GraphRAG. Use when asked to build a knowledge
  graph, extract entities or relations from text, design an ontology, dedupe or merge
  entities, add graph memory or GraphRAG to an agent, or LEARN graph engineering — in
  teaching mode each stage is explained with worked examples and diagrams from the
  user's own domain. Distilled from Southeast University's graduate Knowledge Graph
  course (npubird/KnowledgeGraphCourse).
---

# Knowledge Graphs

Knowledge graphs are what agents *remember*: nodes are entities and facts, edges are
relationships with time and provenance. Treat a knowledge graph as a **product with a
schema**. Quality comes from the pipeline order: model the domain BEFORE extracting,
fuse BEFORE storing, evaluate at every stage.

(The other half of graph engineering — how agents *work* — is the **task-graphs** skill;
loop termination and context budgets are the **context-loops** skill.)

## The 9-Stage Pipeline

Run stages in order. For small projects stages 4-6 collapse into one extraction pass, but
never skip stages 3 (ontology) or 8 (fusion) — they are where real-world graphs fail.

1. **Scope & value test** — Confirm a graph beats a simpler structure. A graph pays off
   when queries are multi-hop ("who worked with X on projects using Y"), when entities
   recur across documents, or when relationships ARE the data. If lookups are single-hop,
   use a table and stop.

2. **Knowledge representation choice** — Pick how facts are encoded: property graph
   (Neo4j-style, pragmatic default), RDF triples (interop/standards), or plain typed
   edges in JSON/SQLite (small scale). Decide now how time and provenance attach to
   every fact.

3. **Ontology modeling** — Define entity types, relation types (with domain/range), and
   attributes BEFORE extraction. Start minimal: 5-15 entity types, 10-30 relation types.
   Two rules: every relation gets a precise verb name (`ACQUIRED`, not `RELATED_TO`),
   and if two types are always queried together, merge them.
   Details and worked examples: [references/modeling.md](references/modeling.md)

4. **Entity extraction (NER)** — Extract typed entities from sources. Method ladder:
   exact rules/dictionaries for closed vocabularies, LLM extraction with the ontology in
   the prompt for open text. Always extract with span + source pointer for provenance.

5. **Relation extraction** — Extract typed edges between recognized entities. Constrain
   the LLM to the ontology's relation list with domain/range checks; reject edges whose
   endpoints have incompatible types. This one validation step removes most hallucinated
   structure.

6. **Event extraction** — For dynamic domains (news, logs, transactions), extract events
   as first-class nodes (trigger + typed arguments + time), not just static edges.
   Extraction methods, prompt patterns, and failure modes for stages 4-6:
   [references/extraction.md](references/extraction.md)

7. **Quality gate** — Before fusion, sample and score: entity precision (are extracted
   entities real and correctly typed?), relation precision (does the source sentence
   actually assert the edge?). Fix the prompt/rules, not the output, then re-run. Target
   90%+ precision on a 50-item sample before proceeding — recall improves with more
   passes; bad precision poisons the graph permanently.

8. **Knowledge fusion** — Merge duplicates within and across sources: same real-world
   entity, different surface forms ("SEU" = "Southeast University" = "东南大学").
   Blocking + matching + merge policy. Skipping this is the #1 cause of useless graphs.
   Matching strategies: [references/fusion-and-llm.md](references/fusion-and-llm.md)

9. **Serve to LLMs (KG × LLM)** — Make the graph useful to agents: GraphRAG retrieval
   (subgraph → context), graph-as-memory (agent writes facts back through stages 4-8),
   and LLM-as-reasoner over paths. Patterns and pitfalls:
   [references/fusion-and-llm.md](references/fusion-and-llm.md)

## Working Rules

- **Schema first, always.** Extraction without an ontology produces a "graph" that is
  really a word cloud with arrows. If the user resists schema design, build the minimal
  5-type ontology from 3 sample documents and show it for approval.
- **Provenance on every fact.** Each node/edge stores `source`, `extracted_at`, and
  confidence. Non-negotiable — fusion (stage 8) and trust both depend on it.
- **Incremental over big-bang.** Process a 10-document pilot through all 9 stages before
  scaling. The pilot exposes ontology gaps at 1% of the cost.
- **LLM extraction is stage machinery, not the pipeline.** The LLM slots into stages
  4-6; the surrounding schema, validation, and fusion are what make the output a
  knowledge graph.

## Workspace: context/graph/

Durable graph work lives in the user's working project under `context/graph/` — plain
files, so the same layout works identically in Claude Code and in Cowork project
folders. Create it lazily on first write. Before asking the user to paste prior work,
check whether the artifact is already on disk.

```
context/graph/
  scope.md            # stages 1-2 (/kg-scope): domain, questions, draft types
  ontology.yaml       # stage 3 source of truth (/kg-schema) — every extraction prompt embeds it
  ontology.ttl        # Turtle serialization of the same ontology
  sources.md          # registry of ingested sources with provenance
  extracted/          # per-source, pre-fusion extraction output
  graph.json          # the fused graph (typed-edges JSON; swap for a DB when scale demands)
  extraction-plan.md  # stage 4-6 designs (/kg-extract, /kg-relations, /kg-events)
  relations-plan.md
  events-plan.md
  fusion-plan.md      # stage 8 design (/kg-fuse)
  eval-report.md      # stage 7 findings (/kg-eval)
  rag-plan.md         # stage 9 design (/kg-rag)
  audits/             # /graph-audit reports and drawn topologies
  runs/               # checkpoints, loop journals, tutor progress — transient
```

Everything except `runs/` is a committed product; recommend the user gitignore
`context/graph/runs/`. Never store graph state in `~/.claude` or any config folder —
it must travel with the project.

## Teaching Mode

When the user wants to LEARN graph engineering (rather than build something), teach it —
do not just execute. Rules:

1. Anchor every stage in the user's own domain: ask for one real project or dataset,
   then use it as the running example through all stages.
2. **Generate visual artifacts as you teach.** Concepts in this discipline are shapes;
   show them. For each major concept, produce a small diagram the user can keep —
   mermaid diagrams (flowchart for the pipeline and task graphs, `graph LR` for example
   ontologies and subgraphs) or a single self-contained HTML page when interactivity
   helps. At minimum: the 9-stage pipeline, a 3-type ontology drawn from the user's
   domain, one extracted subgraph (5-10 nodes) from a real sample, and the diamond
   pattern with the user's own jobs as nodes.
3. Teach in the pipeline's order, one stage per exchange, each ending with a small
   exercise ("write 3 competency questions for your project") before moving on.
4. Close by assembling what was built during the lesson into a starter `ontology.yaml`
   and a drawn task graph for the user's first real build.

## Reference Files

- [references/curriculum.md](references/curriculum.md) — Full translated curriculum of
  the source course with per-lecture summaries and links to the original Chinese slide
  decks. Read when the user wants theory depth or the academic grounding.
- [references/modeling.md](references/modeling.md) — Knowledge representation & ontology
  engineering (course lectures 2-3). Read during stages 2-3.
- [references/extraction.md](references/extraction.md) — Entity, relation, and event
  extraction from rules to LLM prompting (lectures 4-7). Read during stages 4-7.
- [references/fusion-and-llm.md](references/fusion-and-llm.md) — Knowledge fusion and
  KG × LLM integration (lectures 8-9). Read during stages 8-9.

## Credits

Distilled and translated from 东南大学《知识图谱》研究生课程 (Southeast University graduate
course on Knowledge Graphs), Prof. Peng Wang —
https://github.com/npubird/KnowledgeGraphCourse. All original lecture PDFs are in
Chinese; this is an independent English distillation adapted for AI-agent workflows,
building on the graph-engineering skill by @Av1dlive (MIT).
