# Graph Engineering — a Claude Code & Desktop plugin

**Design the structures agents work through — not the prompts.**

**Version**: 0.0.1 · **Updated**: 2026-08-07

Prompt engineers steered the model's words. Loop engineers steered its iterations.
Graph engineers steer its **topology**. This plugin gives Claude a graph-engineering
superpower for building better subagent fan-outs and workflows — with real loop and
context management — plus the full knowledge-graph pipeline for durable agent memory.

## What you get

| Component | What it does |
|---|---|
| **`graph-engineer` agent** | A persona that designs and audits orchestration: drawn task graphs, state-object specs, loop guardrails, ontology drafts. |
| **`task-graphs` skill** | Orchestration topology: the shape test (does the work split?), fake-edge deletion, the diamond pattern (parallel workers → separate verifiers → one owned merge), the stop rule, human gates, four guardrail caps — mapped onto Claude Code subagents and Workflow scripts. |
| **`context-loops` skill** | Loop & context management: state objects instead of transcripts, context isolation and budgets, compaction at merge points, convergence rules (loop-until-dry, budget-bounded), dedupe-against-seen, checkpoint/resume journaling. |
| **`knowledge-graphs` skill** | The 9-stage pipeline — scope, representation, ontology, entity/relation/event extraction, quality gate, fusion, GraphRAG serving — with distilled course references and a teaching mode. |
| **10 commands** | `/kg-tutor` (interactive course), `/kg-scope` → `/kg-rag` (eight single-purpose KG build steps that chain into a full pipeline), and `/graph-audit` (review any pipeline as a task graph). |

## Install

```
/plugin marketplace add rjmoggach/claude-graph-engineering-plugin
/plugin install graph-engineering@claude-graph-engineering-plugin
```

Then try:

- `/graph-audit` on an existing workflow or multi-agent plan
- "design this workflow" / "should these run in parallel" — the Graph Engineer takes it
- `/kg-tutor` to learn the whole knowledge-graph discipline interactively
- "build a knowledge graph from my docs" — the 9-stage pipeline runs

## The two halves

1. **Task graphs** — how agents *work*. Nodes are jobs, edges are execution
   dependencies. Split only what never reads each other's results; verify in separate
   contexts; one owner merges; gates sit on irreversible edges; every loop has a cap and
   a convergence rule.
2. **Knowledge graphs** — what agents *remember*. Nodes are entities and facts, edges
   are relationships with time and provenance. Model before extracting, fuse before
   storing, evaluate at every stage.

## Repo layout

| Path | Role |
|---|---|
| `plugin/` | The plugin as installed (agent, skills, commands, manifest) |
| `.claude-plugin/marketplace.json` | Marketplace entry for `/plugin marketplace add` |
| `graph-engineering/` | Vendored upstream reference material (not shipped in the plugin) |
| `AGENTS.md` | Repo standards & release runbook |

## Credits

The knowledge-graph half is distilled and translated from 东南大学《知识图谱》研究生课程
(Southeast University's graduate Knowledge Graph course), Prof. Peng Wang —
[npubird/KnowledgeGraphCourse](https://github.com/npubird/KnowledgeGraphCourse); no
original lecture materials are redistributed. Task-graph material draws on Google
DeepMind × MIT's ["Towards a Science of Scaling Agent Systems"](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)
and Anthropic's published multi-agent engineering work. Builds on the
[graph-engineering skill](https://github.com/codejunkie99/graph-engineering) by
[@Av1dlive](https://x.com/Av1dlive) (MIT).

MIT licensed.
