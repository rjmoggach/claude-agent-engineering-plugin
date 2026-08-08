---
name: harness-engineering
description: >-
  The runtime around the model — everything except the model itself that determines
  whether an agent delivers reliably: tool surface, execution environment, state and
  handoff, evaluation and observation, constraints and recovery. Use when an agent
  underperforms despite a capable model and a tuned prompt, when the same mistake
  recurs despite prompt edits, when deciding what to fix first across
  prompt/context/tools/orchestration, when hardening an agent system for production,
  or when auditing an agent end to end. Covers the six-layer audit surface and the
  ETCLOVG reference taxonomy, tool-surface subtraction, state handoff and reset
  semantics, generator/critic separation, boundary enforcement (validators, hooks),
  backpressure and loop detection, and the engineer-against-repeats rule. The umbrella
  over the context-engineering, task-graphs, context-loops, and loop-engineering
  skills; routes deep work to them.
---

# Harness Engineering

`Harness = Agent − Model`: the harness is everything around the model — tools,
execution environment, orchestration logic, state, hooks, observability — and it is
usually the binding constraint on real-world reliability. The layering: prompt
engineering optimizes how intent is expressed, context engineering optimizes what
information reaches the model, harness engineering optimizes runtime control. Each
layer contains the previous one.

The evidence for treating the harness as the lever is a wave of results in which the
model was held fixed and only the scaffolding changed — double-digit benchmark gains,
success rates that jumped when most of a tool surface was deleted, agents moved up
leaderboards on harness changes alone (numbers, sources, and caveats:
[references/layers.md](references/layers.md)). The mechanism: models are post-trained
coupled to the harness they were trained against. Moved into a differently fitted
harness — tools matched to the specific system, a tighter prompt, sharper
backpressure — the same model can deliver capability the original harness was leaving
on the floor. Much of the gap between what a model can do and what you observe it
doing is a harness gap.

## The six layers — the audit surface

Every production harness is built from six layers. The most useful diagnostic in the
field: most agents are missing at least two. Audit the layers before tuning anything
inside one.

| Layer | What it covers | Where in this plugin |
|---|---|---|
| Context management | What the model sees, placement, compression | **context-engineering** skill |
| Execution orchestration | Topology, iteration, schedules | **task-graphs**, **context-loops**, **loop-engineering** skills |
| Tool system | What the agent can do, and how reliably it selects | This skill + context-engineering's [tool-design reference](../context-engineering/references/tool-design.md) |
| State and memory | Handoff, persistence, reset semantics | This skill (within a system's runtime) + **knowledge-graphs** (durable memory) |
| Evaluation and observation | Traces, verification, generator/critic separation | This skill |
| Constraints and recovery | Validators, hooks, backpressure, loop detection | This skill |

A deeper seven-layer reference taxonomy (ETCLOVG, from the field's survey) splits
observability and governance into their own layers; use six for auditing, ETCLOVG
when you need the finer cut ([references/layers.md](references/layers.md)).

## Tool system — subtract before adding

The highest-leverage layer, and the one where the field's headline result lives: one
production team raised agent success from 80% to 100% by deleting roughly 80% of the
agent's tools. Every tool costs selection accuracy and attention before it delivers
any value.

- **Subtract first.** Before adding a tool, ask which existing ones can go. Prefer a
  few high-leverage tools that genuinely expand capability over thin wrappers around
  existing APIs.
- **Selection is probabilistic.** No compiler confirms the right tool fired; the
  authoring choices — name, namespace, description — decide whether it is selected at
  all. Namespace by domain and keep names distinct enough that the agent never
  disambiguates between near-identical options.
- **Descriptions are prompt surface.** They load into context and collectively steer
  tool-calling. Precise description refinements alone have materially reduced error
  rates on real benchmarks.
- **Returns are context.** Semantic, human-readable fields over raw internal IDs;
  concise/detailed response options; pagination, filtering, and truncation with sane
  defaults. When truncating, say so and steer the agent toward many small targeted
  calls rather than silently cutting.
- **Errors are corrective prompts.** An error response should state what went wrong
  and how to correct it — a retry hint, a fixed format example, the missing fields.
  An opaque code or traceback is zero recovery signal.

A tool is a contract between a deterministic system and a non-deterministic caller —
that is the shift from ordinary API design. Full description-engineering and
consolidation detail:
[context-engineering/references/tool-design.md](../context-engineering/references/tool-design.md).

## State and handoff

- **Prefer context reset over in-place compaction for long tasks**: spawn a fresh
  agent with a structured state handoff (goal, constraints, decisions, verified
  findings with pointers, remaining work) instead of summarizing inside a degrading
  window. The handoff shape is the context-loops skill's state object.
- **Where compaction is used, tune it on real traces**: maximize recall first so
  nothing load-bearing is dropped, then iterate for precision. Clearing
  already-consumed tool results is the safest, lightest-touch reduction.
- **Hold identifiers, load data just in time**: lightweight pointers (file paths,
  stored queries, links) with runtime loading beat pre-processing everything up
  front. Structure knowledge as an always-loaded index with details on demand
  (progressive disclosure).
- **Durable memory files earn their lines**: every rule in an instruction/memory file
  should be traceable to a specific failure it prevents. Keep them lean — accumulated
  instruction stacks measurably hurt.

## Evaluation and observation

- **Separate generation from evaluation.** The same model in a clean context catches
  errors the generator missed in its own. Verifier separation within a run is the
  task-graphs skill; the maker/checker split for recurring automation is
  loop-engineering; this layer says the principle governs every harness.
- **Judge on artifacts**: tests that ran, output that was checked — never the agent's
  self-report.
- **Keep traces.** Failure attribution needs the actual trajectory: which tool was
  called with what, what came back, where the run went wrong. An unobserved harness
  cannot be improved, only rewritten.
- **Build evals from real failures**, replay them after every harness change, and
  hold out a set so fixes aren't overfit to the incidents that prompted them.

## Constraints and recovery

- **Enforce hard at the boundary, trust the reasoning layer.** Validators and hooks
  at the tool and environment edge (pre-tool-call checks, post-edit linting,
  pre-commit gates); latitude inside the reasoning step. A constraint that must
  survive optimization pressure belongs in the harness — prompt-stated constraints
  are advisory.
- **Backpressure and loop detection.** Cap retries per item, detect repeated
  near-identical calls, and stop the spiral before it burns the window and the
  budget. Loop-engineering's attempt caps are this layer applied to recurring runs.
- **Engineer against repeats.** When the agent makes a mistake, add a validator,
  hook, or workflow change so the mistake cannot recur. Editing the prompt and hoping
  is the anti-pattern; each encoded failure permanently ratchets reliability.
- **Design feedback asymmetrically**: success stays quiet, failures surface verbosely
  and immediately with actionable detail.

## Anti-patterns

- **The rationalization loophole**: a required step with no guardrail against the
  agent talking itself out of it. A 2026 UC Irvine audit found it in 94% of skills
  studied — assume it is present in anything unaudited, and close it with a boundary
  check, never with more prose.
- **Prompt iteration as a substitute for harness change**: a hundred prompt
  iterations still landing under the target is a harness problem. Stop editing the
  prompt; audit the six layers.
- **Context rot**: more window does not mean better performance — attention degrades
  as tokens accumulate. Diagnosis and mitigation live in the **context-engineering**
  skill.

## Claude Code mapping

- **Hooks** are the boundary-enforcement layer: pre-tool-call validation, post-edit
  checks, pre-commit gates. Encode every recurring mistake here.
- **Permission modes, sandboxes, and worktrees** are execution-layer isolation; one
  worktree per unattended change attempt.
- **CLAUDE.md / AGENTS.md are harness memory**: every line traceable to a failure;
  keep them short enough to be read, not skimmed.
- **Skills are progressive disclosure** — an index of names and descriptions always
  loaded, full instructions on demand.
- **Subagents provide clean-context evaluation** — spawn the critic separately;
  never append "check your work" to the generator.
- **MCP servers are tool surface**: audit and prune them like any other tool
  collection; every connected server's descriptions load into the window.

## Reference Files

- [references/layers.md](references/layers.md) — the six-layer audit surface in
  depth, the ETCLOVG taxonomy with per-layer definitions, the harness-only evidence
  table with dates and caveats, and the post-training coupling mechanism. Read when
  auditing a harness or making the case for harness work.
- [references/runtime-control.md](references/runtime-control.md) — the control
  patterns in depth: the reset handoff spec, compaction tuning on traces, the
  validator/hook catalog, backpressure rules, the engineer-against-repeats
  procedure, and feedback-loop design. Read when hardening a specific harness.

## Credits

Original adaptation for this plugin. Grounded in Anthropic's engineering guidance
("Writing effective tools for agents" and "Effective context engineering for AI
agents"), the Agent Harness Engineering survey and its ETCLOVG taxonomy with its
open-source harness catalog, Addy Osmani's agent-harness-engineering essay, and the
Viv Trivedy / HumanLayer framing of the harness as the deciding component.
