---
description: Skeptical review of your knowledge graph — precision/recall sampling, leakage, inflated claims
argument-hint: [what you built + the numbers you're about to claim]
---

Act as a skeptical reviewer of my knowledge graph (graph-engineering plugin,
knowledge-graphs skill, stage 7).

What I built and the numbers: $ARGUMENTS

If missing above, ask for both and wait: what I built, and the numbers I'm about to claim.

Return:
1. Precision and recall at the triple level — how to sample and estimate them with a
   stated confidence interval, not a vibe
2. Where my test set leaks into my training or prompt-development set
3. If I'm reporting link prediction: whether the filtered setting was used, and what a
   trivial baseline would score
4. The three claims a reviewer attacks first, and the experiment that defends each

Assume my numbers are inflated until the sampling method proves otherwise.
