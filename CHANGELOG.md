# Changelog

All notable changes to the agent-engineering plugin (formerly graph-engineering).

## v0.5.0 - 2026-08-07

### /agent-audit — the full gamut in one command

- New command: `/agent-audit` — composes the four single-discipline audits
  (harness, task graph, context, loops) into one pass, run as the plugin's own
  diamond pattern: a cheap scout (plugin-target detection, loop-evidence check),
  the four audits fanned out as parallel subagents in isolated contexts (capped at
  4, structured returns, sequential fallback), and a single merge owner that
  dedupes cross-lens findings, resolves contradictions, ranks severity, and names
  the highest-leverage change. Sub-reports save to their standard
  `context/graph/audits/` paths plus a merged `agent-audit.md`, so re-runs diff
  against every layer.

## v0.4.0 - 2026-08-07

### Harness engineering — the runtime layer above the stack

- New `harness-engineering` skill: the runtime around the model
  (`Harness = Agent − Model`) as the umbrella discipline over context, loop, and
  graph engineering. Covers the six-layer audit surface (context, tools,
  orchestration, state, evaluation, constraints — most systems are missing at least
  two), the ETCLOVG reference taxonomy, tool-surface subtraction, state handoff and
  reset semantics, generator/critic separation, boundary enforcement (validators,
  hooks), backpressure and loop detection, the engineer-against-repeats rule, and
  the rationalization loophole. Two references: layers (taxonomies + the harness-only
  evidence table with dates and caveats + the post-training coupling mechanism) and
  runtime-control (reset handoff spec, compaction tuning on traces, the hook
  catalog, backpressure rules, feedback-loop design).
- New command: `/harness-audit` — six-layer scorecard of an agent system, missing
  layers named, prompt-stated constraints that belong in hooks, single
  highest-leverage change.
- context-engineering's tool-design reference gains the subtraction-first principle
  and return/truncation design; agent and cross-skill routing updated for the
  harness layer.
- Grounded in Anthropic's tool-writing and context-engineering guidance, the Agent
  Harness Engineering survey (ETCLOVG), and Addy Osmani's and HumanLayer's framing.
- Repo docs: the `graph-engineering/` reference clone is now local-only and
  gitignored like the other two; README/AGENTS layout tables updated to match.

## v0.3.0 - 2026-08-07

### Agent Engineering — umbrella rename + the context discipline

- **Renamed**: repo → `claude-agent-engineering-plugin`, plugin → `agent-engineering`,
  agent → `agent-engineer`. Graph, loop, and context engineering are one superpower;
  the name now says so. Existing installs of `graph-engineering` should remove the old
  marketplace/plugin and reinstall.
- New `context-engineering` skill: the context window as an attention budget — the
  five degradation patterns (lost-in-middle, poisoning, distraction, confusion,
  clash) with detection and recovery, the write/select/compress/isolate framework,
  optimization in priority order (cache-stable prompts, observation masking,
  compaction, partitioning), and tool design as context contracts. Three references:
  degradation, optimization, tool-design.
- New commands: `/context-audit` (context-hygiene review of an agent setup — window
  composition, degradation risk, tool surface, cache stability) and `/task-brief`
  (pseudo-formal launch briefs for long-horizon autonomous runs: success predicates,
  non-counting outcomes, auditor checklists, persistence paired with gates).
- Agent and cross-skill routing updated for the context discipline.
- Original adaptation inspired by muratcankoylan/Agent-Skills-for-Context-Engineering
  (MIT) — no dependency on upstream tooling.

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
