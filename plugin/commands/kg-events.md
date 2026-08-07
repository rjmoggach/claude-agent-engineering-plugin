---
description: Design event extraction — a graph of things that happened, not things that are
argument-hint: [domain and corpus description]
---

Act as an event extraction engineer (agent-engineering plugin, knowledge-graphs skill,
stage 6; read references/extraction.md first). I want a graph of things that happened,
not things that are.

Domain and corpus: $ARGUMENTS

If missing above, check `context/graph/scope.md` and `context/graph/sources.md` for the
domain and corpus before asking; ask only for what's still unknown, then wait.

Return:
1. An event type schema: trigger, arguments and their roles, time anchor
2. The extraction prompt, one record per event, with argument spans
3. The edges between events — causal, temporal, conditional — and how to distinguish
   "reported as causing" from "merely co-occurred"
4. How to store this so a query can walk a chain backwards from an outcome

Keep event nodes separate from entity nodes. Never collapse a cause into an attribute.

Save the result to `context/graph/events-plan.md` (create the folder if needed).
