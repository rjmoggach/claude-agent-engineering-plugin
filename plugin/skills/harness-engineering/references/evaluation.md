# Evaluation — Contracts, Blind Comparison, Held-Out Sets

*(Read when you need to know whether a change to an agent system actually improved
it. The question here is not "did this run succeed" but "is this version better
than the one it replaces, and will that hold." Answering it badly is how teams
ship confident regressions.)*

## Start from an outcome contract

Before changing any component (a skill, an agent definition, a prompt, a tool
surface), write down what it is for, in terms that survive a rewrite. Extract the
contract from the component and its real use, not from your plan for it:

- **The need**: who invokes this, in what situation, what they are trying to get.
- **Success**: checkable properties of the deliverable, not of the agent's
  confidence or effort.
- **Ranked qualities**, with the tradeoff stated where two conflict.
- **Hard constraints** and **disqualifying failures**.
- **Boundaries**: when it should fire, and when something else owns the work.

**Separate ends from means.** Do not carry a procedure, section order, checklist
shape, taxonomy, or vocabulary into the contract because the current version
happens to use it. A method belongs in the contract only when the method itself
is the requirement. Everything else is one implementation of the end, and
freezing it there blocks any better implementation from passing.

**Strip identifying wording.** Distinctive phrases, named frameworks, specific
numbers, and signature examples let a judge recognize which version produced an
output. Remove them or the comparison stops being blind.

**Have a second reader attack it** before you freeze. Give them the contract and
the original and ask for four things: what the original requires that the
contract lost, what the contract states more strongly or weakly than the original
does, where a current implementation choice got fossilized into a requirement,
and which properties a judge could not actually check. Resolve those, then freeze.
The contract does not change again during the run.

## Build the benchmark before you build the candidate

Order matters. A benchmark written after the candidate tests the candidate's
strengths. Give the benchmark designer the frozen contract and a neutral
description of the capability, and not the current implementation.

Cover ordinary use, hard cases where the obvious cause is wrong, the case where
the right answer is that little should change, edge cases including out-of-scope
requests the component should decline, realistic variation, and whether the
component fires when it should and stays quiet when it should not.

### The evaluation packet

Each task carries a packet with everything an informed judge needs and nothing
that reveals the contestants:

| Include | Exclude |
|---|---|
| The request and inputs the contestant got | Either version's instructions |
| The contract clauses bearing on this task | Which condition produced which output |
| Ground truth and invariants a correct answer must satisfy | Builder reasoning or prior verdicts |
| Relative importance where two qualities conflict | Any hint of the outcome you want |
| Disqualifying failures for this task | Wording that fingerprints one source |

Where no single ideal answer exists, describe what success means instead of
inventing a rigid gold answer. A rigid answer key on an open-ended task measures
similarity to one solution, not quality.

## Split iteration from held-out

Two sets, drawn to cover the contract independently:

- **Iteration set**: builders see failures here and learn from them.
- **Held-out set**: sealed. Builders never see the tasks, the packets, the
  expected results, or the verdicts until the final evaluation.

**The builder must be blind to the sealed set, and the person orchestrating the
run is usually a builder.** Have the designer write held-out tasks to a file and
return only a coverage summary. If the tasks pass through the builder's context
on the way to storage, the set is contaminated on arrival and the final
evaluation proves nothing.

**Replace tests once iteration has touched them.** A held-out task that gets
inspected to explain a loss has become an iteration task. Retire it and write a
fresh one. Reusing it is how a benchmark quietly turns into a training set.

## Judge hygiene

Judgment goes to a fresh run that produced none of the work and is separate from
the builders, the designers, the contract extractors, the orchestrator, and other
judges.

Give the judge the task, its neutral evaluation packet, and the anonymized
outputs in randomized order. The judge must not see either version's
instructions, which model or version produced an output, what change is being
tested, builder reasoning, previous verdicts, or which result anyone is hoping
for. That last one matters more than it looks: a judge told what the orchestrator
expects will find it.

Ask the judge to pick the better result, state its confidence, explain concretely
why it is better for this user under the contract, and name any requirement
either result violated or handled unusually well. Judge the real deliverable and
the actual behaviour, never a summary written by whoever ran the comparison.

This is the same clean-room rule the task-graphs and context-loops skills apply
to verifiers, tightened for comparison: a verifier must not see the reasoning
behind the claim, and a judge must additionally not see which side is which.

## Conditions worth running

At minimum, on identical tasks:

1. **The component as it stands**, unchanged.
2. **No component at all.** The condition people skip, and the one that most
   often settles the question. A component that does not beat its own absence is
   costing context for nothing.
3. **The candidate.**

Every sample comes from a fresh run holding only what its condition needs. No
shared conversations, artifacts, or memory between runs, and no contestant seeing
another's output.

**Be exact about models.** Record the model actually used for every run. Never
silently substitute one, never route two conditions through the same model and
report them as two, and never claim a model was tested that was not available.
If a planned condition cannot run, drop it and say so. An honest three-condition
result beats a four-condition table with one row invented.

## What counts as a real improvement

Green means decisive and repeatable. Specifically, the candidate should beat the
unchanged version on the same tasks, add something beyond running with no
component at all, satisfy the contract, hold up on tasks it has never seen, and
introduce no regression that matters.

These are not improvements, however good the score looks:

- A narrow win inside the margin between samples.
- A win that came from length. Longer output reads as more thorough to judges and
  usually is not.
- A generic stylistic preference the contract never asked for.
- A win on the iteration set that does not survive the held-out set.
- A win produced by tuning to one judge's habits, or by picking the best of
  several generations.

Do not relax the standard to reach green, and do not fold benchmark answers into
the component.

## Retirement is a valid result

If the unchanged version keeps losing to no component at all, the honest outcome
is to retire or disable it, recorded as a success with the evidence attached. Do
not manufacture a revision to avoid a negative result. This is the subtraction
rule from the tool layer applied to instruction surface: a component that does
not earn its context is a cost, and removing it is a real improvement.

## The authoring test

**A component should mostly contain what the model could not reasonably work out
on its own.** Apply it line by line when building the candidate:

| Keep | Cut |
|---|---|
| Facts about this system, org, or codebase | General good practice the model already applies |
| Conventions and decisions with no external evidence | Restatements of the request |
| Failures that actually happened here, and their fix | Motivational emphasis |
| Numbers, thresholds, and names that must be exact | Explanations of well-known concepts |
| Non-obvious ordering where the wrong order breaks | Steps the model would take unprompted |

The related rule for memory and instruction files still holds: every line should
trace to a specific thing that went wrong. A rule with no failure behind it is a
deletion candidate.

## Failure modes of the evaluation itself

| Symptom | Cause | Fix |
|---|---|---|
| Candidate wins everything, gains vanish in production | Benchmark written after the candidate | Design from the contract, before building |
| Held-out results match iteration results exactly | Sealed set leaked through the builder | Route sealed tasks to storage, not through the builder |
| Judges agree with each other and with the builder | Judge saw the instructions or the expected winner | Anonymize, randomize order, withhold the hoped-for result |
| Scores rise, users report nothing changed | Optimized to the judge, not the contract | Rotate judges; re-check against contract properties |
| Every version passes | Criteria restate the current implementation | Re-extract the contract, ends only |
| Improvement fades over releases | Nothing replays the original failures | Retain reproductions; run them on every change |
