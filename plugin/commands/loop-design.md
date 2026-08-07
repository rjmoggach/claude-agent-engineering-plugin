---
description: Design and scaffold a recurring operational loop — pattern, cadence, autonomy level, state files, gates
argument-hint: [what you want automated, e.g. "triage new issues daily" — or a pattern name]
---

Act as the Agent Engineer designing an operational loop (agent-engineering plugin;
follow the loop-engineering skill, and read its references/patterns.md and
references/design-checklist.md first).

What to automate: $ARGUMENTS

If nothing was given above, ask what recurring work I want off my plate and wait.

## Step 1 — Fit the pattern

Match the request to one of the seven patterns (daily triage, issue triage, changelog
drafter, post-merge cleanup, dependency sweeper, CI sweeper, PR babysitter) or declare
a custom pattern. State: goal in one sentence, explicit non-goals, watched scope,
cadence (with its cost multiplier), risk level, and the starting autonomy level —
**week one is L1 report-only unless I explicitly override**.

## Step 2 — Design the run as a task graph

Draw the single run's topology (mermaid): triage → act → verify → escalate/notify,
with the maker/checker split explicit — the verifier is a separate context with a
reject-by-default stance that runs the actual tests. Mark human gates and early-exit
points (empty watchlist must exit cheap).

## Step 3 — Scaffold the operational files

Create at the repo root (do not overwrite existing ones — merge additively):

- `LOOP.md` — this loop's entry: pattern, cadence, level, gates, schedule, promotion
  criteria to the next level
- `STATE.md` (or the pattern's own state file) — sections: High Priority (acting /
  waiting on human), Watch List, Recent Noise (ignored), last-run stamp
- `loop-budget.md` — daily caps for this loop, on-exceed procedure, kill switch;
  note that only humans edit caps
- `loop-run-log.md` — append-only, with the entry format
- `loop-constraints.md` — binding rules: push/merge, denylisted paths, code rules
  (attempt cap 3, no disabling tests, one fix per run), communication, budget
- `gate.yaml` — machine-readable denylist + auto-merge allowlist (default: none) +
  max-files

## Step 4 — Wire the schedule

Give me the exact command(s) to start it in Claude Code — /loop for in-session
cadence, a scheduled routine for cron-style runs, or a GitHub Actions workflow if
CI-coupled — plus the kill switch and how to check on it.

## Step 5 — Report

Summarize: the pattern and level, what week one will and will not do, the promotion
criteria to L2, and the first thing I should read after the first run. Do not enable
anything beyond L1 without my explicit say-so.
