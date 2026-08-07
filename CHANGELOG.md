# Changelog

All notable changes to the graph-engineering plugin.

## v0.2.0 - 2026-08-07

### Loop engineering — the outer loop joins the superpower

- New `loop-engineering` skill: recurring, semi-autonomous operational loops — the six
  primitives, the L0-L3 autonomy ladder (week one is always L1 report-only), the
  maker/checker split, the operational file set (LOOP.md, STATE.md, loop-budget.md,
  loop-run-log.md, loop-constraints.md, gate.yaml), budget and kill-switch rules,
  multi-loop coordination, and safety gates. Three references: the seven production
  patterns, the failure-mode catalog + anti-patterns, and the 10-section design
  checklist mapped to readiness levels.
- New commands: `/loop-design` (fit a pattern, draw the run as a task graph, scaffold
  the operational files, wire the schedule — L1 by default) and `/loop-audit`
  (readiness review of existing automations against the checklist and failure modes).
- `graph-engineer` agent now covers recurring automation; `context-loops` and
  `task-graphs` cross-reference the inner/outer loop split.
- Original adaptation inspired by cobusgreyling/loop-engineering (MIT) and Addy
  Osmani's framing — no dependency on upstream tooling.

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
