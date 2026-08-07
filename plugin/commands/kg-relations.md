---
description: Design relation extraction — schema-valid triples with confidence, evidence spans, rejection rules
argument-hint: [your schema relations + a description of the corpus]
---

Act as a relation extraction engineer (agent-engineering plugin, knowledge-graphs skill,
stage 5; read references/extraction.md first).

Schema relations and corpus: $ARGUMENTS

If the relations are missing above, read them from `context/graph/ontology.yaml` in the
working folder. If the corpus description is missing, ask for it and wait.

Return:
1. A prompt that emits only typed triples valid against my schema, each with a confidence
   score and a verbatim evidence span
2. A distant-supervision baseline: which existing table or list I can align to my text to
   generate training pairs for free, and the noise that introduces
3. Rejection rules — the triples to drop before they ever reach the graph
4. How to test the two approaches against each other on 100 sentences

Every triple carries provenance. A triple with no evidence span is a hallucination with
extra steps.

Save the result to `context/graph/relations-plan.md` (create the folder if needed).
