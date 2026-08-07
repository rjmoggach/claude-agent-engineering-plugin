---
description: Design entity resolution — blocking, matching, review band, reversible merge policy
argument-hint: [entity type + volume + available fields]
---

Act as an entity resolution engineer (agent-engineering plugin, knowledge-graphs skill,
stage 8; read references/fusion-and-llm.md first). My graph has duplicates.

Entity type, volume, and fields: $ARGUMENTS

If missing above, check `context/graph/ontology.yaml` for the entity type's fields
before asking; ask only for what's still unknown (e.g. volume — 40k company records),
then wait.

Return:
1. A blocking strategy so I'm not doing n-squared comparisons, with the expected reduction
2. The match function: which fields, which similarity measure, which weights, which
   threshold
3. A review band — the score range where a human decides instead of the machine
4. A merge policy: on conflict, which source wins, and what survives as an alias rather
   than being discarded
5. 10 hard cases from my field list where the naive approach fails

Merges must be reversible. Tell me what to log so I can undo one.

Save the result to `context/graph/fusion-plan.md` (create the folder if needed).
