---
name: context-loops
description: >-
  Loop and context management for agent systems — the discipline that keeps multi-agent
  runs from drifting, looping forever, or drowning in their own transcripts. Use when an
  agent loop won't terminate, subagents lose or duplicate context, results degrade across
  iterations, a long task needs checkpoints and resume, or when designing what each
  subagent's context should hold. Covers state-object design, context isolation and
  budgets, compaction at merge points, convergence rules (loop-until-dry, budget-bounded,
  fixed-round), dedupe-against-seen, and checkpoint/resume journaling.
---

# Context Loops

Two resources ruin agent systems when unmanaged: **context** (what each agent can see)
and **iteration** (how many times work repeats). This skill treats both as engineering
budgets with explicit designs, not defaults.

Scope: iteration **within a run** — the inner loop. Recurring loops that run on a
schedule over days (triage, sweepers, babysitters) are the **loop-engineering** skill;
each of that outer loop's runs gets this skill's treatment inside it.

Core principle: **pass state, not transcript.** Work flows through a graph as a compact
state object; conversation history stays where it happened.

## The state object

The record that travels between nodes. Design it before the first agent spawns:

```yaml
found:      # facts/results produced so far — compact, deduplicated, with source pointers
decided:    # decisions already made, so no node re-litigates them
remaining:  # what is still open — the worklist driving the next round
```

Rules:

- Every node **reads** the state object and **returns a delta** in a declared shape —
  never free prose the merge owner must parse by guesswork. In Claude Code Workflow
  scripts, enforce the shape with a schema so returns are validated data.
- `decided` is append-only. A downstream node that disagrees raises a finding for the
  merge owner; it does not silently re-decide.
- Everything in `found` carries a source pointer (file:line, URL, agent id). Unattributed
  findings are unverifiable and get dropped at merge.
- Keep it small. If the state object grows past what a fresh agent can absorb in one
  read, that's a compaction signal (below), not a reason for bigger prompts.

## Context isolation

- **Minimum-necessary context per node.** A subagent gets: its one job, the slice of
  state it needs, and the return shape. Not the transcript, not sibling outputs, not the
  full plan. Isolation is why fan-out works — each worker's window is spent on its own
  job, and workers can't bias each other.
- **Verifiers get clean rooms.** A verifier receives the claim and how to check it —
  never the worker's reasoning, which anchors the verifier into agreeing. Prompt it to
  refute.
- **Read the primary source, not the summary of a summary.** When a node needs depth on
  one item, hand it the pointer and let it read fresh — don't relay a compressed copy
  through a third context.
- **Context budget per node**: scope each job so it fits comfortably in one context. If a
  job can't, that's a graph problem — split the job, don't stuff the window.

## Compaction points

Long runs die of accumulation. Schedule compression at the graph's natural joints:

- **At every merge node**: dedupe, drop dead ends, resolve conflicts, rewrite `found`
  compactly. The merge owner is also the compaction owner.
- **Between rounds of a loop**: the next round starts from the compacted state object,
  never from the previous round's full output.
- **Summarize decisions, keep pointers to evidence.** Compaction may drop reasoning
  transcripts, but never drops the decision list or source pointers — those are the two
  things that prevent re-derivation and hallucinated provenance.

## Loop engineering

Every loop declares, up front:

1. **A hard round cap.** No exceptions — this is the backstop, not the exit.
2. **A convergence rule** — the intended exit:
   - **Loop-until-dry** (unknown-size discovery: bugs, gaps, edge cases): stop after K
     consecutive rounds that produce nothing new (K=2 is standard). Fixed counts miss
     the tail; no rule never stops.
   - **Budget-bounded**: stop when the token/cost/time budget for the phase is spent;
     report coverage honestly ("swept 40 of 60 files").
   - **Fixed-round**: only when the workload is known in advance.
3. **Dedupe against everything seen, not just what was kept.** Track every candidate
   ever surfaced — including rejected ones. Dedupe against accepted-only and the loop
   rediscovers rejected items every round and never converges. (The single most common
   non-termination bug.)
4. **A progress metric logged per round** (new items found, items remaining). Two rounds
   with no movement and rounds left on the cap = the convergence rule is wrong; stop and
   report rather than burning the cap.
5. **Retry policy**: transient failures retry once with the same input; a node failing
   twice returns a null/failure marker that the merge handles — the loop never blocks on
   one dead node.

## Checkpoint and resume

- **Journal every node's return** (a results file or the Workflow journal) keyed by node
  identity. A rerun replays completed nodes from the journal and executes only what
  changed — this is what makes long graphs restartable instead of re-run-from-zero.
- **Checkpoint the state object at phase boundaries** to a file. A crashed or
  context-compacted session resumes from the last checkpoint, not from memory.
- **Location**: checkpoints and journals live in the working project at
  `context/graph/runs/` (create lazily; advise gitignoring `runs/`). Never in `~/.claude`
  or other config folders — run state must sit next to the project it belongs to, where
  both Claude Code and Cowork can reach it.
- Keep node functions **deterministic in their inputs** (no wall-clock, no randomness in
  the routing) so cached replays stay valid.
- State that must outlive the session graduates from checkpoints to persistent memory —
  that's the **knowledge-graphs** skill (extraction → fusion → retrieval, incremental),
  living in `context/graph/` alongside the run state.

## Failure smells → fixes

| Smell | Cause | Fix |
|---|---|---|
| Loop never ends | Dedupe vs accepted-only; no convergence rule | Dedupe vs all-seen; loop-until-dry + hard cap |
| Results drift/degrade across rounds | Each round feeds on prior output, errors compound | Rounds start from compacted state; verifier between rounds |
| Subagents contradict each other | Shared mutable context, no decision log | `decided` list in state object; one merge owner |
| "It forgot what we agreed" | Decisions lived in transcript, got compacted away | Decisions in the state object/checkpoint file, not prose |
| Worker output unusable at merge | Free-prose returns | Declared return shapes (schemas) |
| One stuck agent stalls the run | No retry/null policy | Retry once, then null-and-continue; merge tolerates nulls |
| Rerun repeats finished work | No journaling | Journal node returns; resume replays the prefix |
