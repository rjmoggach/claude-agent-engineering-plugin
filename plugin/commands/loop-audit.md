---
description: Audit existing automations/loops for readiness — verifier separation, state hygiene, budgets, kill switches
argument-hint: [path to repo or loop files, or describe the automations you run]
---

Act as the Graph Engineer auditing operational loops (graph-engineering plugin; follow
the loop-engineering skill — walk references/design-checklist.md and check against
references/failure-modes.md).

Target: $ARGUMENTS

If no target was given, look for loop evidence in the current repo — `LOOP.md`,
`STATE.md`, `loop-*.md`, `gate.yaml`, `.github/workflows/`, scheduled routines — and
ask me only if nothing is found. Read the actual files before critiquing.

For EACH loop found:

1. **Identify it**: pattern, cadence, current autonomy level (L0-L3) as actually
   configured — not as documented, if the two disagree.
2. **Walk the ten checklist sections** and score which are satisfied. The verdict is
   the highest level the evidence supports; flag any loop running above its evidence.
3. **Hunt the failure modes**: infinite fix loops (attempt counts in state?), state
   rot (stale items? prune step?), verifier theater (does the verifier run tests in a
   separate context?), token burn (early exit on empty watchlist? budget file?),
   notification fatigue, missing kill switch, auto-merge without allowlist, shared
   state without schema, parallel collision (acting_on locks?).
4. **Check multi-loop coordination** if more than one loop exists: one owner per
   branch, separate state files, priority order, shared denylist, aggregate budget.

Then return:

- A table: loop · pattern · cadence · configured level · evidenced level · verdict.
- Findings most-severe first: the loop, the defect, the failure mode it invites, the
  concrete fix (which file, what content).
- The single highest-leverage fix to make first.

Save the full report to `context/graph/audits/loops-audit.md` (create the folder if
needed). This is a review — do not change any loop files unless I ask.
