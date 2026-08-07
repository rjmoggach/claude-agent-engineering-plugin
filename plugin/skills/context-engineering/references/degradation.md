# Context Degradation — Detection, Recovery, Thresholds

*(Read when an agent misbehaves in a long session. Diagnose the pattern first; each
has a different fix. Adapted field guidance — see the skill's credits.)*

## Pattern 1: Lost-in-middle

**Symptoms**: correct information exists in context but the model ignores it;
responses contradict provided data; instructions from earlier in a long prompt get
"forgotten".

**Mechanics**: attention is U-shaped — the first tokens act as an attention sink and
the tail stays fresh, leaving the middle under-attended. Middle-positioned content
loses 10-40% recall; the effect is significant from ~4k tokens.

**Fix**: place task goal and constraints at the start, key findings and the ask at the
end, bulk detail in the middle. Add section headers as attention anchors. For a
must-include long document: prepend a summary, append the conclusions.

## Pattern 2: Poisoning

**Symptoms**: degraded quality on previously-successful tasks; wrong tools or
parameters; a hallucination that persists despite explicit correction.

**Mechanics**: one bad claim (hallucination, tool error, wrong retrieved fact) enters
context and compounds — every downstream step treats it as ground truth and
re-asserts it.

**Fix — remove, never correct-on-top**: identify the turn where the bad claim entered;
truncate to before it (or restart with only verified content); reload verified
sources; record the rejected claim so it isn't re-ingested. Corrections layered onto
poisoned context rarely stick — the original error retains attention weight.

**Prevention**: validate tool outputs, retrieved documents, and model-generated
summaries before they enter context — those are the three poisoning vectors.

## Pattern 3: Distraction

**Symptoms**: attention diluted; the model addresses tangents or over-weights
irrelevant material.

**Mechanics**: models must attend to everything provided. One irrelevant document
causes a measurable, disproportionate hit — a step function, not a slope.

**Fix**: relevance-filter before loading; keep reference material behind tool calls
so it enters only when the current step needs it. Treat distractor prevention as
binary: clean context, or accept degradation.

## Pattern 4: Confusion

**Symptoms**: responses answer the wrong aspect; tool calls fit a different task;
outputs blend requirements from multiple objectives.

**Mechanics**: distinct from distraction — the model applies the *wrong task's*
constraints, not just diluted attention.

**Fix**: task isolation. Separate contexts (subagents/sessions) per objective; when
switching within one session is unavoidable, use explicit reset markers stating which
constraints now apply.

## Pattern 5: Clash

**Symptoms**: the model silently picks one of two contradictory sources — looks like
a correct answer, is effectively random.

**Mechanics**: individually-correct but mutually contradictory content (version
conflicts, divergent retrievals) with no precedence rule.

**Fix**: filter outdated versions before load; state source precedence; annotate
unavoidable conflicts explicitly (what conflicts, which source, which wins, why).
Detect contradictions in the retrieval layer, not after.

## Thresholds and expectations

- Expect degradation onset around **60-70% of the advertised window** for complex
  multi-fact work; only about half of models claiming 32k+ maintained quality at that
  length in RULER-style testing.
- The curve is a **cliff, not a slope** — set mitigation triggers at ~70% of the known
  onset, not at first symptoms.
- Near-perfect needle-in-haystack scores do NOT predict real workload performance —
  needle tests measure single-fact retrieval, not synthesis under instructions.
- Thresholds go stale with every model update: re-benchmark on your own workload.

## Counterintuitive findings

- **Shuffled context can beat coherent context** on some retrieval tasks — coherent
  ordering creates false associations. Don't assume heavier organization helps; test.
- **The first distractor costs the most.** Additional distractors add less than the
  first one did.
- **Low query-content similarity accelerates degradation** — inference across
  dissimilar content decays faster with length than surface-similar lookups.

## Diagnosis gotchas

1. **Normal variance mimics degradation**: one bad run is noise; the same dip
   consistently past a token threshold is signal. Baseline over multiple runs.
2. **Prompt defects mimic degradation**: if it fails at 2k tokens too, fix the prompt.
3. **Contradictory retrievals poison silently** — implement clash detection upstream.
4. **"Graceful degradation" designs miss the cliff** — monitoring that assumes linear
   decline fires too late.
