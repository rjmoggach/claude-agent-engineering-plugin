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

**If the target is a plugin** (a repo containing `.claude-plugin/`, or
`agents/`/`skills/`/`commands/` directories): its components are the graph. Agents and
commands are nodes; the hand-offs their docs describe ("then use X", "hands execution
to Y", chained command sequences) are the edges. Apply every check below to that
topology — fake hand-offs, agents that verify their own output, review loops with no
round cap, skills that pass whole transcripts where a state object would do. Findings
must be **self-contained rewrites in the target plugin's own files and vocabulary** —
never "install the graph-engineering plugin" as a fix. This plugin is a design-time
optimizer; the target must work for users who have never installed it.

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

Save the full report to `context/graph/audits/{target-name}-audit.md` in the working
folder (create the folder if needed) so later audits can diff against it.
