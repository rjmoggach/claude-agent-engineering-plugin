---
name: task-graphs
description: >-
  Design agent orchestration as an explicit task graph before spawning any subagent.
  Use when planning multi-agent work, writing a Workflow script, fanning out subagents,
  reviewing an existing pipeline, or deciding whether work should be parallel or
  sequential. Covers the shape test (does the work split?), fake-edge deletion, the
  diamond pattern (parallel workers, separate verifier contexts, one owned merge),
  the stop rule, human gates on irreversible edges, and the four guardrail caps.
  Applies directly to Claude Code subagents and Workflow scripts.
---

# Task Graphs

A task graph is the plan drawn as a DAG: nodes are jobs — each one something you would
hand to a single assistant — and an arrow exists only when a job needs another job's
*result* before it can start. A small state object (what was found, what was decided,
what remains) travels with the work. This is the pattern that has run data infrastructure
for decades (Airflow, Prefect, Temporal), applied to agents.

**Rule zero: draw the graph before spawning anything.** Even three bullet points naming
the nodes and the real edges. Orchestration invented mid-flight produces fake edges,
self-graded work, and unmergeable results.

## Step 1 — The shape test (decide whether to parallelize at all)

From the DeepMind × MIT scaling study (180 controlled configurations): coordinated teams
beat a single agent by ~80% on work that splits into independent pieces — and **every**
multi-agent configuration lost on sequential work where each step needs the full picture
(degrading 39-70%). Uncoordinated agents amplified each other's errors 17.2x; a single
coordinator owning the merge cut it to 4.4x.

The decision procedure:

1. Ask: *where does this work split into pieces that never read each other's results?*
2. Split only that. Everything sequential stays with one agent.
3. Never let findings merge without one owner of the merge.

If nothing splits, stop here: one agent, no graph.

## Step 2 — Delete fake edges

For every "and then" in the plan, check whether the next job actually reads the previous
job's output. "Summarize this file and then check my calendar" — the calendar step never
uses the summary; the edge is fake. Delete fake edges and those jobs run in parallel.
Most hand-built pipelines contain two or three.

## Step 3 — The diamond pattern

The shape serious systems converge to:

```
        worker 1
plan →  worker 2  → verify → merge → result
        worker 3
```

- **Split** into independent angles sized so each worker is one coherent job.
- **Workers run in parallel** with only the inputs they need — never the full transcript.
- **Verify in a separate context.** Non-negotiable: a model grading its own work in its
  own context misses most of its own mistakes. Give each verifier a *different* question
  (is it correct? is it current? is the source real?) — diverse skeptics catch what
  identical ones cannot.
- **A verifier choosing between candidates needs a tighter clean room.** The
  context-loops rule (the verifier sees the claim and how to check it, never the
  worker's reasoning) is the floor. A verifier comparing two results must also not
  know which side is which: anonymize the outputs, randomize their order, and withhold
  what change is being tested and which result you are hoping for. A verifier told
  what you expect will find it.
- **One owner merges.** The merge node dedupes, resolves conflicts, and produces the
  single result. Workers never write to the shared result directly.

## Step 4 — The human gate

The human is a node. Route every irreversible edge — send, publish, refund, delete,
deploy — through explicit approval. Placement rule: **put the gate where a mistake is
expensive to undo, not on every step.** A gate on everything makes the human the
bottleneck; a gate on nothing means nobody is watching. Judge the system on numbers that
cannot argue back (tests that ran, money that landed), never on its own self-reports.

## Step 5 — Guardrails (all four, every graph)

1. Every loop gets a maximum number of rounds.
2. One writer per file — no two jobs mutate the same artifact.
3. The routing lives in written steps; the model fills the jobs, not the plan.
4. A hard cap on how many agents can spawn.

## Applying this in Claude Code

- **Independent nodes → one message, multiple Agent calls** (or a Workflow
  pipeline/parallel). Sequential nodes stay in the main loop or a single agent.
- **Worker prompts carry the state object, not the conversation.** Each subagent gets:
  its one job, the inputs it needs (file paths, prior findings as compact data), and the
  exact shape to return. Subagent contexts are naturally isolated — that is a feature;
  don't defeat it by pasting the whole transcript in.
- **Verifiers are separately spawned agents** prompted to *refute*, never the worker
  asked "are you sure?". In Workflow scripts, this is the find → verify stage split.
- **The main loop (or one merge agent) owns the merge** — dedupe, conflict resolution,
  final synthesis happen in exactly one place.
- **Prefer pipeline over barrier**: let each item flow through its stages independently;
  synchronize only when a stage genuinely needs all prior results together (dedupe across
  the full set, early-exit on zero findings).
- **Files needing parallel mutation → per-agent isolation** (worktrees), or reshape the
  split so each file has one writer.
- Loop termination and state-object design in depth: use the **context-loops** skill.
  A graph that should run recurrently on a schedule (triage, sweepers): use the
  **loop-engineering** skill. Persistent memory across sessions: use the
  **knowledge-graphs** skill. Runtime enforcement around the graph — validators,
  hooks, backpressure, recovery: use the **harness-engineering** skill.

## Reviewing an existing pipeline

Persist drawn topologies and audit reports to `context/graph/audits/` in the working
project (create lazily) so later sessions can diff against them.

**When the target is another plugin or reusable system**, the deliverable is
self-contained rewrites in the target's own files and vocabulary — bake the patterns in.
Never make the target depend on this plugin at runtime ("install graph-engineering" is
not a fix); this plugin is a design-time optimizer, and Claude Code has no inter-plugin
dependency mechanism anyway.

Audit in this order and report per finding — the edge/node, the defect, the fix:

1. Fake edges (steps chained without data flow) → parallelize.
2. Missing shape test (parallelism imposed on sequential work) → collapse to one agent.
3. Self-verification (worker grades its own output) → separate verifier contexts.
4. Ownerless merge (results concatenated, never reconciled) → one merge owner.
5. Missing gates on irreversible actions → insert human gate.
6. Unbounded loops, multi-writer files, uncapped spawning → apply the four guardrails.

## Credits

The stop-rule numbers are from Google DeepMind × MIT, "Towards a Science of Scaling
Agent Systems". Pattern distillation draws on Anthropic's published multi-agent
engineering work and the graph-engineering skill by @Av1dlive (MIT).
