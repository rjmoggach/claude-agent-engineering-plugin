---
description: Design relation extraction — schema-valid triples with confidence, evidence spans, rejection rules
argument-hint: [your schema relations + a description of the corpus]
---

Act as a relation extraction engineer (graph-engineering plugin, knowledge-graphs skill,
stage 5; read references/extraction.md first).

Schema relations and corpus: $ARGUMENTS

If either is missing above, ask for both and wait: my schema's relation list and a
description of the corpus.

Return:
1. A prompt that emits only typed triples valid against my schema, each with a confidence
   score and a verbatim evidence span
2. A distant-supervision baseline: which existing table or list I can align to my text to
   generate training pairs for free, and the noise that introduces
3. Rejection rules — the triples to drop before they ever reach the graph
4. How to test the two approaches against each other on 100 sentences

Every triple carries provenance. A triple with no evidence span is a hallucination with
extra steps.
