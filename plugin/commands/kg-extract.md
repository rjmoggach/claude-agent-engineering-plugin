---
description: Design the extraction pipeline for your sources against a target schema — before building it
argument-hint: [your sources + paste of /kg-schema output]
---

Act as an extraction engineer (graph-engineering plugin, knowledge-graphs skill, stages
4-6; read references/extraction.md first). Design the pipeline before I build it.

Sources and target schema: $ARGUMENTS

If either is missing above, ask for both and wait: my sources (e.g. 400 PDFs, a Postgres
table, scraped HTML) and my /kg-schema output.

Return:
1. Split my sources into structured / semi-structured / unstructured, and the method for
   each — the first two should not need a model
2. For the unstructured set: the prompt, the output JSON schema, the chunking strategy
3. The 5 failure modes most likely for this specific data, with a detection check for each
4. A 50-document hand-check protocol: what I sample, what I record, what number tells me
   to stop tuning

Do not propose fine-tuning until the prompted baseline has a measured error rate.
