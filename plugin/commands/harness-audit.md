---
description: Audit an agent system's harness across the six layers — tools, orchestration, state, evaluation, constraints, context
argument-hint: [path to repo/agent setup, or describe the system and its symptoms]
---

Act as the Agent Engineer auditing a harness (agent-engineering plugin; follow the
harness-engineering skill — read its references/layers.md and
references/runtime-control.md first).

Target: $ARGUMENTS

If no target was given, audit the current repo's agent surface: CLAUDE.md/AGENTS.md,
`.claude/` (hooks, skills, agents, commands, settings), MCP/tool configuration,
workflow and loop definitions, CI-coupled automation. Read the actual files before
critiquing. If symptoms were described ("it keeps making the same mistake", "prompt
edits stopped helping"), note which layer each symptom implicates.

Walk all six layers and score each **present / partial / absent** with the evidence:

1. **Context management** — window composition, placement, compression triggers.
   Score it here; run /context-audit for the deep dive.
2. **Tool system** — count every tool the agent can select (including every MCP
   server's). Apply subtraction first: which tools could go? Then the tool-design
   checklist: overlaps, description quality, return design, error design. Flag thin
   wrappers and near-identical names.
3. **Execution orchestration** — topology, iteration, schedules. Score here; run
   /graph-audit and /loop-audit for depth.
4. **State and memory** — handoff shape between contexts, reset semantics,
   checkpoint/resume, prune steps, memory-file hygiene (is every line traceable to a
   failure?).
5. **Evaluation and observation** — generator/critic separation, artifact-based
   judgment, traces kept, evals built from real failures and replayed after changes.
6. **Constraints and recovery** — boundary validators and hooks vs. prose-only
   rules, denylists, attempt caps, loop detection, kill switches. Hunt the
   rationalization loophole: for each "must/always/never" in the instructions, what
   mechanically happens if the agent skips it?

Then return:

- A layer scorecard: layer · score · evidence · the one fix that would raise it.
- The missing layers, named. Expect at least two — say plainly which they are.
- Findings most-severe first: the file/component, the defect, the layer, the
  concrete fix (which file, what content), and what breaks in production if left.
- Every prompt-stated constraint that should be a hook or validator instead.
- The single highest-leverage change.

Save the report to `context/graph/audits/harness-audit.md` (create the folder if
needed). This is a review — change nothing unless I ask.
