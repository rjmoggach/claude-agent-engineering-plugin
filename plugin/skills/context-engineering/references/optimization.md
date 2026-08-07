# Context Optimization — Techniques, Targets, Gotchas

*(Read when applying a specific technique. Priority order: cache structure → masking
→ compaction → partitioning. Measure before and after — machinery that doesn't move a
metric is itself context cost.)*

## 1. Cache-stable prompt structure

Inference engines reuse cached Key/Value tensors when consecutive requests share an
identical prefix. Cheapest optimization, zero quality risk, applies whenever stable
prefixes exist.

Ordering, most-stable first:
1. System prompt (never changes within a session)
2. Tool definitions
3. Reused templates / few-shot examples
4. Conversation history (grows, but shares prefix with prior turns)
5. Current query and dynamic content — always last

Rules:
- **One changed byte invalidates everything after it.** No timestamps, session
  counters, or request IDs in the stable prefix — move dynamic metadata into a user
  message or tool result after it.
- Deployment changes (reordered tools, reworded system prompt) cold-start the cache:
  expect a temporary 2-5x cost spike; roll out gradually.
- Targets: 70%+ hit rate on stable workloads → ~50% cost and ~40% latency reduction
  on cached tokens.

## 2. Observation masking

Replace verbose tool outputs with compact references once their purpose is served:
`[Obs:{ref_id} elided. Key: {summary}. Full content retrievable.]` — the original
stays stored and fetchable by reference.

- **Never mask**: the most recent turn, observations in active reasoning, and error
  output while debugging is in progress (error in the last ~3 turns suspends masking
  for error-related observations).
- **Mask after ~3 turns**: verbose outputs whose key points are already extracted.
- **Mask immediately**: duplicates, boilerplate, already-summarized outputs.
- Tool outputs often dominate agent trajectories, so masking usually yields the
  largest capacity gain. Targets: 60-80% reduction in masked observations, under 2%
  quality impact.

## 3. Compaction

Summarize accumulated context and reinitialize with the summary. Trigger at ~70%
utilization — **not** at 85%+, where the compacting model is itself attention-starved
and drops goals and constraints. If forced late, run compaction as a separate clean
call over only the material to summarize.

Order of compression: tool outputs first, then old conversational turns, then
retrieved documents. **Never compress the system prompt** — it anchors behavior.

What to preserve, by type:
- **Tool outputs** → findings, metrics, error codes, conclusions. Drop raw bulk and
  resolved stack traces.
- **Turns** → decisions, commitments, user preferences, context shifts. Drop filler
  and the exploratory path to an already-captured conclusion.
- **Documents** → task-relevant claims and data points. Drop one-time supporting
  elaboration.

Targets: 50-70% token reduction at under 5% quality loss. Beyond 70% reduction, audit
for critical loss — over-aggressive compaction is the most common failure. After any
compaction, **re-validate the summary against the current goal**: summaries look
authoritative while silently carrying stale state.

## 4. Partitioning

Split work across subagents with isolated contexts when the task alone would exceed
~60% of one window. Each subagent runs clean and focused, returning a structured
result to one coordinator (design the split with the task-graphs skill; govern each
context with context-loops).

Economics: the coordinator prompt, result aggregation, and error handling all cost
tokens. Break-even is typically **3+ independent subtasks** — estimate total tokens
(coordinator + all subagents) before committing.

## Budget policy

Allocate explicit budgets per category before the session: system prompt, tools,
retrieved documents, history, tool outputs, plus a 5-10% reserved buffer. Then
optimize on triggers, not on a schedule:

- Utilization above ~70% → compact history (masking first).
- A category over its allocation → apply that category's technique (e.g. tool outputs
  over budget → mask resolved observations).
- Repetition or missed instructions → mask + compact.
- Quality below baseline → audit context composition before optimizing anything.

## Choosing by what dominates the window

| Dominant component | First action | Second action |
|---|---|---|
| Tool outputs (over ~50%) | Mask observations | Compact remaining turns |
| Retrieved documents | Summarize | Partition if docs are independent |
| Message history | Compact with selective preservation | Partition new subtasks |
| Mixed | Cache-stable structure first | Layer masking + compaction |
| Near-limit while debugging | Mask resolved outputs only — keep error detail | — |
