# Changelog

All notable changes to the graph-engineering plugin.

## v0.1.0 - 2026-08-07

### context/graph workspace + design-time optimizer principle

- New `context/graph/` workspace convention in the user's working project: scope →
  ontology → stage plans → graph.json with provenance, `audits/` for reports,
  `runs/` for transient checkpoints/journals (gitignorable). Skills create it lazily;
  all `/kg-*` commands now read their predecessor's artifact from disk instead of
  requiring pastes, and save their output back. `/kg-tutor` auto-resumes from
  `runs/tutor.md`.
- Design-time optimizer principle wired through the agent, task-graphs skill, and
  `/graph-audit`: a new plugin-target mode audits another plugin's agents/commands as a
  task graph and emits self-contained rewrites in the target's own files — never a
  runtime dependency on this plugin.
- Graph state never goes in `~/.claude` or config folders — it travels with the project,
  working identically in Claude Code and Cowork.

## v0.0.1 - 2026-08-07

### Initial release

- `graph-engineer` agent: designs and audits agent orchestration, loops, and memory
  architecture — drawn task graphs, state-object specs, loop guardrails, ontology drafts.
- `task-graphs` skill: shape test, fake-edge deletion, diamond pattern, stop rule, human
  gates, guardrail caps, applied to Claude Code subagents and Workflow scripts.
- `context-loops` skill: state-object design, context isolation and budgets, compaction
  points, convergence rules, dedupe-against-seen, checkpoint/resume journaling.
- `knowledge-graphs` skill: the 9-stage pipeline (scope → representation → ontology →
  extraction → quality gate → fusion → GraphRAG serving) with distilled course
  references (modeling, extraction, fusion-and-llm, curriculum) and teaching mode.
- 10 commands: `/kg-tutor`, `/kg-scope`, `/kg-schema`, `/kg-extract`, `/kg-relations`,
  `/kg-events`, `/kg-fuse`, `/kg-eval`, `/kg-rag`, `/graph-audit`.
- Marketplace manifest for `/plugin marketplace add rjmoggach/claude-graph-engineering-plugin`.
