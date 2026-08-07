---
description: Audit a workflow, pipeline, or multi-agent plan as a task graph — fake edges, self-verification, missing gates
argument-hint: [describe the pipeline, or a path to the script/plan/docs]
---

Act as the Graph Engineer (graph-engineering plugin; follow the task-graphs and
context-loops skills). Audit the following as a task graph — do not fix anything yet;
this is a review.

Target: $ARGUMENTS

If no target was given above, ask me to describe the pipeline or point you at the
script/plan/docs, then wait. If the target is a path, read the actual files before
critiquing.

Audit in this order:

1. **Draw what exists.** A mermaid flowchart of the current topology as actually
   implemented — nodes as jobs, arrows only where output feeds input.
2. **Fake edges** — steps chained without data flow. Name each; these parallelize free.
3. **Shape test** — is parallelism imposed on sequential work (collapse to one agent),
   or is splittable work running sequentially (fan it out)?
4. **Verification** — who checks each result, and in whose context? Flag every place a
   worker grades its own output.
5. **Merge ownership** — where do parallel results combine, and does exactly one node
   own dedupe/conflict resolution?
6. **Human gates** — list the irreversible actions (send, publish, delete, deploy) and
   whether a gate sits on each.
7. **Loops and context** — for every loop: round cap? convergence rule?
   dedupe-against-all-seen? For every node: does it receive a compact state object or a
   transcript dump? Any multi-writer files? Any uncapped spawning?

Then return:
- The redrawn **target topology** (second mermaid diagram) with workers, verifiers,
  merge owner, and gates marked.
- A finding list, most severe first: the edge/node, the defect, the concrete fix, and
  what breaks at 10x volume if left alone.
