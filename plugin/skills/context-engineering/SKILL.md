---
name: context-engineering
description: >-
  The discipline of the context window itself — curating everything that enters the
  model's limited attention budget: system prompts, tool definitions, retrieved
  documents, history, tool outputs. Use when an agent forgets or ignores information
  that is present, quality degrades as sessions grow, token costs balloon, prompts need
  cache-friendly structure, tools need designing as context contracts, or when
  diagnosing lost-in-the-middle, context poisoning, distraction, confusion, or clash.
  Covers the attention budget, the five degradation patterns with detection and
  recovery, the write/select/compress/isolate framework, optimization techniques
  (masking, compaction, caching, partitioning), and tool design. Inspired by the
  context-engineering skills collection (Muratcan Koylan).
---

# Context Engineering

Prompt engineering crafts the instructions; context engineering curates **everything**
that enters the window — system prompt, tool definitions, retrieved documents, message
history, tool outputs. The constraint is not raw token capacity but **attention**: as
context grows, models degrade in predictable patterns. The goal is the smallest set of
high-signal tokens that maximizes the likelihood of the outcome.

Boundaries inside this plugin: this skill owns the window itself — what's in it, where,
and how it's compressed. **context-loops** owns run-state mechanics (state objects,
convergence, checkpoints); **task-graphs** owns splitting work across agents;
**knowledge-graphs** owns persistent memory. They compose: partitioning (here) creates
the subagents whose contexts task-graphs and context-loops then govern.

## The attention budget

- Attention is U-shaped: beginnings and ends of context get reliable attention; the
  middle loses 10-40% recall. Place critical information at the edges, never the
  middle; when a long document must be included whole, prepend its key points and
  append its conclusions.
- Models cannot skip what you load: every irrelevant token competes for attention.
  Even a single irrelevant document measurably degrades performance — the effect is a
  step function, not linear. Keep context clean or accept the hit.
- Degradation is non-linear with a **cliff edge**: performance holds, then drops
  sharply at a model-specific threshold — often at 60-70% of the advertised window for
  complex tasks. Trigger mitigation at ~70% utilization, before the cliff, not at
  symptoms.
- More window is not more capability: bigger windows delay the curve, they don't
  remove it. Splitting across subagents beats stuffing one context.

## The five degradation patterns

Diagnose before mitigating — each pattern has a different fix
(details, detection signals, and gotchas: [references/degradation.md](references/degradation.md)):

1. **Lost-in-middle** — correct information is in context but ignored. Fix: placement
   at the edges, structural headers as attention anchors.
2. **Poisoning** — a hallucination or bad tool output entered context and compounds
   through self-reference. Fix: remove, don't correct — truncate to before the
   poisoning point and rebuild from verified sources; corrections layered on top
   rarely work.
3. **Distraction** — irrelevant loaded content dilutes attention. Fix: filter before
   loading; move might-need material behind tool calls.
4. **Confusion** — constraints from one task bleed into another. Fix: task isolation —
   separate contexts or explicit reset markers.
5. **Clash** — individually-correct but contradictory sources. Fix: precedence rules
   and version filtering before load; annotate unavoidable conflicts explicitly.

## The four mitigations: write / select / compress / isolate

- **Write** — move context out of the window into files (the `context/graph/`
  workspace, scratchpads, run logs); keep a pointer. Use above ~70% utilization.
- **Select** — retrieve just-in-time instead of pre-loading; relevance-filter
  everything that enters.
- **Compress** — mask and summarize what must stay (below). Use when everything
  present is relevant but heavy.
- **Isolate** — partition across subagents so no single context nears its cliff. The
  most aggressive and often most effective — design the split with **task-graphs**.

## Optimization, in priority order

Full technique detail and targets: [references/optimization.md](references/optimization.md)

1. **Cache-stable prompt structure** (cheapest, zero quality risk): stable content
   first — system prompt, tool definitions, templates — dynamic content last. No
   timestamps or session IDs in the stable prefix; one changed byte invalidates the
   cache from that point on.
2. **Observation masking**: replace verbose tool outputs with compact references once
   their purpose is served — keep the pointer so the original stays retrievable.
   Never mask the latest turn or error output during active debugging.
3. **Compaction** at ~70% utilization: summarize and reinitialize. Mask first, then
   compact; compress tool outputs first, then old turns, then retrieved documents;
   **never compress the system prompt**. Late compaction (above 85%) degrades the
   summary itself — the compactor is also attention-starved.
4. **Partitioning** across subagents when the task alone would exceed ~60% of the
   window. Coordination costs real tokens — worth it from roughly 3+ independent
   subtasks.

## Tools are context

Every tool definition loads into the window and steers behavior — tool design IS
context engineering ([references/tool-design.md](references/tool-design.md)):

- **Consolidate**: if a human can't say definitively which tool applies, the agent
  can't either. Fewer, comprehensive tools beat many overlapping ones.
- **Descriptions are prompts**: what it does, when to use it, what it accepts (with
  format examples and defaults), what it returns.
- **Errors must be actionable**: what went wrong + how to correct it. "Failed" is
  zero recovery signal.
- **Consider architectural reduction**: primitives (filesystem, shell) plus good
  documentation often beat a scaffold of specialized tools — and improve as models do.

## Working rules

- Measure before optimizing; measure the optimization. Machinery that doesn't move a
  metric is itself context cost.
- Verify the prompt works at small context before diagnosing degradation — a prompt
  that fails at 2k tokens has a prompt problem, not a context problem.
- One quality dip is noise; the same dip recurring past a token threshold is signal.
- Compaction summaries go stale: re-validate against the current goal after compacting.
- Design lean and outcome-first: carry the outcome, hard constraints, and completion
  bar; leave the path to the model. Accumulated instruction stacks measurably hurt.

## Reference Files

- [references/degradation.md](references/degradation.md) — the five patterns in
  depth: detection signals, recovery procedures, thresholds, counterintuitive
  findings, diagnosis gotchas. Read when an agent is misbehaving in a long session.
- [references/optimization.md](references/optimization.md) — masking rules, compaction
  strategy by message type, cache mechanics, partitioning economics, budget policy,
  performance targets. Read when applying a specific technique.
- [references/tool-design.md](references/tool-design.md) — consolidation, description
  engineering, response formats, error design, namespacing, architectural reduction.
  Read when creating or auditing an agent's tool surface.

## Credits

Original adaptation for this plugin, inspired by Muratcan Koylan's Agent Skills for
Context Engineering collection
(https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering, MIT) and the
research it distills (lost-in-the-middle, RULER, and published production guidance
from AI labs).
