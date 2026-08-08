# Agent Engineering for Claude Code & Desktop

**Design the structures agents work through.**

**Version**: 0.5.1 · **Updated**: 2026-08-07

This plugin helps you design five things: the topology work flows through (task
graphs), the iteration it runs on (loops), the attention budget it spends (context
engineering), the memory it keeps (knowledge graphs), and the harness that carries
all of it at runtime (harness engineering). Use it to build better subagent
fan-outs, workflows, recurring automations, agent memory, and production-reliable
agent systems.

## What you get

| Component | What it does |
|---|---|
| **`agent-engineer` agent** | A persona that designs and audits agent machinery: drawn task graphs, state-object specs, loop guardrails, operational-loop designs, context budgets, ontology drafts. |
| **`task-graphs` skill** | Orchestration topology: the shape test (does the work split?), fake-edge deletion, the diamond pattern (parallel workers → separate verifiers → one owned merge), the stop rule, human gates, four guardrail caps — mapped onto Claude Code subagents and Workflow scripts. |
| **`context-loops` skill** | Loop & context management within a run: state objects instead of transcripts, context isolation and budgets, compaction at merge points, convergence rules (loop-until-dry, budget-bounded), dedupe-against-seen, checkpoint/resume journaling. |
| **`loop-engineering` skill** | Recurring operational loops: the six primitives, the L0-L3 autonomy ladder (week one is always report-only), maker/checker split, the operational file set (LOOP.md, STATE.md, budget, run log, constraints, gate), multi-loop coordination, budgets and kill switches, the seven production patterns, and the failure-mode catalog. |
| **`context-engineering` skill** | The window itself: attention budget and the U-curve, the five degradation patterns (lost-in-middle, poisoning, distraction, confusion, clash) with detection and recovery, write/select/compress/isolate, optimization (cache-stable prompts, masking, compaction, partitioning), and tool design as context contracts. |
| **`knowledge-graphs` skill** | The 9-stage pipeline — scope, representation, ontology, entity/relation/event extraction, quality gate, fusion, GraphRAG serving — with distilled course references and a teaching mode. |
| **`harness-engineering` skill** | The runtime around the model (`Harness = Agent − Model`): the six-layer audit surface and ETCLOVG taxonomy, tool-surface subtraction, state handoff and reset semantics, generator/critic separation, boundary enforcement (validators, hooks), backpressure and loop detection, and the engineer-against-repeats rule — the umbrella over the other disciplines. |
| **16 commands** | `/agent-audit` (the full gamut — all four audits fanned out in parallel, merged by one owner), `/kg-tutor` (interactive course), `/kg-scope` → `/kg-rag` (eight single-purpose KG build steps that chain into a full pipeline), `/graph-audit` (review any pipeline — or another plugin — as a task graph), `/loop-design` + `/loop-audit` (recurring automation), `/context-audit` (context-hygiene review of an agent setup), `/harness-audit` (six-layer harness review of an agent system), and `/task-brief` (pseudo-formal launch briefs for long-horizon runs). |

## Install

```
/plugin marketplace add rjmoggach/claude-agent-engineering-plugin
/plugin install agent-engineering@claude-agent-engineering-plugin
```

Then try:

- `/agent-audit` on another plugin or agent system — the full gamut in one pass
- `/graph-audit` on an existing workflow or multi-agent plan
- `/harness-audit` on an agent system that underperforms despite prompt tuning
- "design this workflow" / "should these run in parallel" — the Agent Engineer takes it
- `/kg-tutor` to learn the whole knowledge-graph discipline interactively
- "build a knowledge graph from my docs" — the 9-stage pipeline runs

## The workspace: `context/graph/`

Durable graph work lives in your project's working folder — plain files, identical in
Claude Code and Cowork. The skills create it lazily and read prior artifacts from disk
so chained commands don't need pasting:

```
context/graph/
  scope.md → ontology.yaml/.ttl → extraction/relations/events/fusion/rag plans
  sources.md, extracted/, graph.json     # the graph itself, with provenance
  audits/                                # /graph-audit reports + drawn topologies
  runs/                                  # checkpoints, loop journals (gitignore this)
```

Everything except `runs/` is a committed product. Graph state never goes in `~/.claude`
or config folders — it travels with the project.

## A design-time optimizer, not a dependency

This plugin improves *other* plugins and pipelines; it is never a runtime requirement
for them. `/graph-audit` pointed at a plugin repo treats its agents/commands as nodes
and their documented hand-offs as edges, then emits self-contained rewrites in the
target's own files and vocabulary. The target must work for users who never installed
agent-engineering — Claude Code has no inter-plugin dependency mechanism, and this
plugin never asks for one. (The `context/` convention is likewise free to adopt without
installing anything.)

## The disciplines

1. **Task graphs** — how agents *work*. Nodes are jobs, edges are execution
   dependencies. Split only what never reads each other's results; verify in separate
   contexts; one owner merges; gates sit on irreversible edges; every loop has a cap and
   a convergence rule.
2. **Loops, at two timescales** — the inner loop (iteration within a run: convergence,
   state objects, compaction) and the outer loop (recurring automation over days:
   autonomy ladder, budgets, kill switches, maker/checker). A recurring loop is a task
   graph executed on a schedule with durable state between runs.
3. **Context engineering** — what agents *attend to*. Treat the window as an
   attention budget: place at the edges, filter before loading, mask and compact
   before the cliff, isolate across subagents, and design tools as context contracts.
4. **Knowledge graphs** — what agents *remember*. Nodes are entities and facts, edges
   are relationships with time and provenance. Model before extracting, fuse before
   storing, evaluate at every stage.
5. **Harness engineering** — the runtime that carries the other four.
   `Harness = Agent − Model`: prompt engineering optimizes how intent is expressed,
   context engineering optimizes what information reaches the model, harness
   engineering optimizes runtime control — tool surface, state handoff, evaluation,
   constraints, recovery. Audit the six layers before tuning inside one; most
   systems are missing at least two.

## Repo layout

| Path | Role |
|---|---|
| `plugin/` | The plugin as installed (agent, skills, commands, manifest) |
| `.claude-plugin/marketplace.json` | Marketplace entry for `/plugin marketplace add` |
| `graph-engineering/` | Local-only reference clone (gitignored — inspiration, not integration) |
| `loop-engineering/` | Local-only reference clone (gitignored — inspiration, not integration) |
| `context-engineering/` | Local-only reference clone (gitignored — inspiration, not integration) |
| `AGENTS.md` | Repo standards & release runbook |

## Credits

The knowledge-graph half is distilled and translated from 东南大学《知识图谱》研究生课程
(Southeast University's graduate Knowledge Graph course), Prof. Peng Wang —
[npubird/KnowledgeGraphCourse](https://github.com/npubird/KnowledgeGraphCourse); no
original lecture materials are redistributed. Task-graph material draws on Google
DeepMind × MIT's ["Towards a Science of Scaling Agent Systems"](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)
and Anthropic's published multi-agent engineering work. Builds on the
[graph-engineering skill](https://github.com/codejunkie99/graph-engineering) by
[@Av1dlive](https://x.com/Av1dlive) (MIT). The loop-engineering skill is an original
adaptation inspired by
[cobusgreyling/loop-engineering](https://github.com/cobusgreyling/loop-engineering)
(Cobus Greyling, MIT) and Addy Osmani's harness/factory/intent-debt framing. The
context-engineering skill is an original adaptation inspired by
[Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering)
(Muratcan Koylan, MIT) and the research it distills. The harness-engineering skill is
an original distillation grounded in Anthropic's engineering guidance
(["Writing effective tools for agents"](https://www.anthropic.com/engineering/writing-tools-for-agents),
["Effective context engineering for AI agents"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)),
the [Agent Harness Engineering survey](https://openreview.net/pdf?id=eONq7FdiHa) and
its ETCLOVG taxonomy, [Addy Osmani's agent-harness-engineering essay](https://addyosmani.com/blog/agent-harness-engineering/),
and the Viv Trivedy / HumanLayer framing of the harness as the deciding component.

MIT licensed.
