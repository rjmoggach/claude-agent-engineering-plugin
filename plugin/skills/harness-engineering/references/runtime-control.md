# Runtime Control — Handoff, Enforcement, Recovery

*(Read when hardening a specific harness. These are the control patterns that keep a
well-designed loop or graph reliable once it meets production.)*

## The reset handoff

For long tasks, prefer spawning a fresh agent with a structured handoff over
compacting a degrading window in place. The handoff is a file (or state object) the
successor reads first, and it must carry:

- **Goal and completion bar** — verbatim, never re-summarized from a summary.
- **Hard constraints** — the rules that survived from the launch brief; note which
  are enforced by hooks vs. stated in prose.
- **Decisions made** (append-only) with one-line rationale each — the successor never
  re-litigates them.
- **Verified findings** with source pointers (file:line, URL, run ID) — unverified
  material is marked as such or dropped.
- **Remaining work** as a concrete list, ordered.
- **Failure log** — what was tried and failed, so the successor doesn't repeat it.

The shape is the context-loops skill's state object with the goal and constraint
blocks added; the difference is intent — a checkpoint assumes the same context may
resume, a handoff assumes a fresh one will.

## Compaction, tuned on traces

Where in-place compaction is used instead of reset:

1. Collect real traces of the agent's work, including failures.
2. Tune the compaction prompt for **recall first**: verify against traces that every
   load-bearing item (decisions, unresolved errors, constraints, pointers) survives.
3. Then iterate for precision — drop the redundant bulk.
4. The safest first-line reduction is **clearing already-consumed tool results**:
   the call and its conclusion stay, the raw payload goes.
5. After any compaction, re-validate the summary against the current goal; summaries
   look authoritative while silently carrying stale state.

Thresholds and per-type compression rules: context-engineering's optimization
reference.

## Validators and hooks — the boundary catalog

Enforce hard at the tool and environment edge; leave latitude inside the reasoning
step. The standard mount points:

| Mount point | Enforces |
|---|---|
| Pre-tool-call | Argument validation, path denylists, scope checks, budget guards |
| Post-edit | Linters, formatters, type checks, forbidden-pattern scans |
| Pre-commit / pre-push | Test suite, secret scanning, diff-size caps |
| Pre-send / pre-publish | Human gates on irreversible actions (task-graphs skill) |
| Scheduler entry | Constraint-file read, kill-switch check (loop-engineering skill) |

Rules:

- A constraint that must survive optimization pressure goes here, never only in the
  prompt — prompt-stated constraints are advisory under pressure.
- Validator failures are corrective prompts: state what was violated and what a
  passing call looks like.
- Hooks are versioned with the project and tested like code — a silently broken hook
  is worse than no hook, because the harness is trusted to be enforcing it.

## Backpressure and loop detection

- **Attempt caps are mechanical**: count per item in state, cap (3 is standard),
  escalate with full context on the cap. "Keep trying until it works" is the classic
  infinite loop.
- **Detect repetition, not just failure**: two near-identical tool calls with
  near-identical results is a spiral signal even when neither call errored. Halt and
  reassess rather than letting retries burn the window.
- **Budget the retry path separately** from the happy path — retry storms are where
  token burn hides.
- **Escalation must reach a human channel** that is actually watched; a "stuck" note
  in a state file no one reads is an escalation failure.

## Engineer against repeats

The ratchet procedure — run it every time the agent makes a mistake worth
preventing:

1. **Capture** the failure from the trace: what the agent did, what it should have
   done, which layer let it happen.
2. **Classify**: wrong tool selected (tool system) → fix names/descriptions or
   subtract; skipped a required step (constraints) → add a boundary check; acted on
   stale state (state layer) → fix prune/handoff; bad output passed (evaluation) →
   strengthen the critic; drowned mid-window (context) → route to
   context-engineering.
3. **Encode** the fix in the harness — a validator, hook, workflow change, or a
   single traceable line in the memory file. Editing the prompt and hoping is
   explicitly the anti-pattern.
4. **Replay** the original failure to prove it can no longer occur, and add it to
   the eval set so regressions surface.

Corollary for memory files: every line in CLAUDE.md / AGENTS.md should trace back to
a specific thing that went wrong. A rule with no failure behind it is a candidate
for deletion — accumulated instruction stacks measurably hurt.

## Feedback-loop design

- **Success is silent, failures are verbose.** Quiet success preserves context;
  loud, specific failure gives the agent (and the human) something to act on.
- Failure output carries: what failed, the exact input, the expected shape, and the
  suggested correction — at the moment of failure, in the failing context.
- Aggregate for humans: digests for routine outcomes, immediate pings only when a
  decision is required (loop-engineering's notification rule, applied harness-wide).

## The rationalization loophole

The most common single defect in skill/instruction design: a required step with no
guardrail against the agent reasoning its way around it ("the tests obviously pass,
skipping the run"). A 2026 UC Irvine audit found it in 94% of skills studied.

- Assume it is present in anything unaudited.
- Close it at the boundary: a hook or validator that checks the step's *artifact*
  (the test log exists and is fresh, the file was actually read), never additional
  prose emphasis — "you MUST" loses to rationalization under pressure; an artifact
  check does not.
- When auditing, search for every "always/never/must" in the instructions and ask:
  what mechanically happens if the agent skips this? If the answer is nothing, the
  loophole is open.
