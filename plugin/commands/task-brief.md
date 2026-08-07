---
description: Write a pseudo-formal launch brief for a long-running autonomous agent or parallel orchestration
argument-hint: [the hard problem or long-running task you want an agent (or fleet) to attempt]
---

Act as the Agent Engineer writing a long-horizon task brief (agent-engineering
plugin). Long autonomous runs succeed or fail on the launch prompt: under persistence
pressure, agents produce answer-shaped near misses, and lenient judges wave them
through. The brief's job is to make that impossible — state success so precisely that
an adversarial reader cannot satisfy its letter without satisfying its intent.

The problem: $ARGUMENTS

If nothing was given above, ask for the problem and what a complete answer would let
me do, then wait.

## Build the brief in this order

1. **Success predicate first.** One sentence, explicit quantifiers and scope, a
   checkable property of the *deliverable* — never of the agent's confidence or
   elapsed effort. If it can't be written, say so and stop: the problem needs
   decomposition or a scoping session, not a long-horizon run.
2. **Definitions with degenerate cases.** Operationalize every load-bearing term —
   units, populations, boundaries, and the edge cases a lazy solution would exploit.
3. **Non-counting outcomes** (highest-leverage block). Enumerate what a capable agent
   under pressure would return instead of a solution: the narrowed-scope version, the
   reduction to an unvalidated assumption, bounded/anecdotal verification, the survey,
   the plan, the confident sketch. Each exclusion removes one escape hatch.
4. **Auditor checklist.** The domain-specific ways a candidate can look right and be
   wrong. Ask me what I would refuse to accept from a junior collaborator — that
   refusal list becomes the checklist. Verifiers hunt from this list in fresh
   contexts; generic "check the work" catches nothing.
5. **Orchestration policy** (for parallel runs) — heuristics, never fixed
   assignments: genuinely diverse opening portfolio; early workers blind to the
   favored approach; a registry of approach families grouped by idea (not wording);
   routes marked blocked when they stall at a gap as hard as the goal, reopened only
   for a materially new mechanism; cross-pollination late. Treat tight inter-agent
   agreement as a diversity failure signal, not confirmation.
6. **Reporting contract + return condition.** Concrete artifacts with every claim
   traceable to a tool result or artifact from the session — status reports rejected.
   Return only when the artifact survives adversarial audit against the checklist.
7. **Effort floor, solvability framing, contamination guards.** Minimum effort before
   giving up may be considered; "assume a solution exists" only where plausible; what
   external search may and may not be used for. **Every persistence instruction must
   be paired with a verification gate** — persistence against a loose predicate
   produces confident non-solutions.

## Then, before handing it to me

- **Red-team the brief**: ask "how could an agent satisfy the letter of this brief
  without solving the problem?" and patch every credible answer into the non-counting
  list or the auditor checklist.
- **Score it**: adversarially checkable predicate? every near miss excluded?
  enumerated auditor list? persistence paired with gates? return condition a
  predicate over the artifact? early diversity preserved? artifact-based reporting?
  contamination guards? Any "no" is a defect — fix it, don't hand it over.
- Flag anything in the brief that is a *constraint that must survive optimization
  pressure* — those belong in the harness (loop-engineering skill: constraints file,
  gates), not the prompt; prompt-stated constraints are advisory.
- Keep it lean: outcome, hard constraints, evidence sources, completion bar. Leave
  the path to the model — accumulated instruction stacks measurably hurt.

Deliver the complete brief as a single copy-ready block, then a short note on
suggested orchestration shape (single agent vs parallel; design the topology with the
task-graphs skill if parallel). Save a copy to `context/graph/briefs/{slug}.md`
(create the folder if needed).
