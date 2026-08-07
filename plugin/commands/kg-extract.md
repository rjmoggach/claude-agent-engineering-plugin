---
description: Design the extraction pipeline for your sources against a target schema — before building it
argument-hint: [your sources + paste of /kg-schema output]
---

Act as an extraction engineer (agent-engineering plugin, knowledge-graphs skill, stages
4-6; read references/extraction.md first). Design the pipeline before I build it.

Sources and target schema: $ARGUMENTS

If the schema is missing above, read `context/graph/ontology.yaml` from the working
folder. If the sources are missing, ask for them and wait (e.g. 400 PDFs, a Postgres
table, scraped HTML). If neither the paste nor ontology.yaml exists, point me at
/kg-schema first.

Return:
1. Split my sources into structured / semi-structured / unstructured, and the method for
   each — the first two should not need a model
2. For the unstructured set: the prompt, the output JSON schema, the chunking strategy
3. The 5 failure modes most likely for this specific data, with a detection check for each
4. A 50-document hand-check protocol: what I sample, what I record, what number tells me
   to stop tuning

Do not propose fine-tuning until the prompted baseline has a measured error rate.

Save the plan to `context/graph/extraction-plan.md` and register the sources (with
provenance notes) in `context/graph/sources.md` (create the folder if needed).
