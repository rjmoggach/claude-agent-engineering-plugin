---
name: loop-engineering
description: >-
  Design recurring, semi-autonomous operational loops — the outer loop that schedules
  and orchestrates agent runs over days, as opposed to iteration within one run. Use
  when setting up scheduled automation (/loop, cron routines, GitHub Actions), a
  maintenance loop (triage, CI sweeper, PR babysitter, dependency sweeper, changelog
  drafter), deciding how much autonomy a loop gets, or reviewing why an automation
  misbehaves. Covers the six primitives, the L0-L3 autonomy ladder with phased rollout,
  the maker/checker split, the operational file set (LOOP.md, STATE.md, budget, run log,
  constraints, gate), budget and kill-switch rules, multi-loop coordination, and safety
  gates. Inspired by the loop-engineering discipline (Cobus Greyling, Addy Osmani).
---

# Loop Engineering

A loop replaces you as the prompter: a system that discovers work, assigns it, verifies
results, and persists state on a schedule — you design the loop instead of typing the
next prompt. The relationship to a single session:

```
Harness = single session setup (tools, context, permissions)
Loop    = harness + schedule + state + verification chain
```

This is the **outer** loop — recurring runs over days. Iteration *within* one run
(convergence rules, state objects, compaction) is the **context-loops** skill; the
topology of each run is the **task-graphs** skill. A recurring loop is a task graph
executed on a schedule with durable state between executions.

## The six primitives

Every production loop composes these; a missing one is usually the design gap:

1. **Scheduling** — discovery and triage on a cadence (`/loop`, cron routines,
   scheduled cloud agents, GitHub Actions).
2. **Worktrees** — isolation for every unattended code-change attempt; one worktree per
   fix, discarded on reject.
3. **Skills** — persistent project knowledge; how the loop pays down *intent debt*
   (conventions and "we don't do it this way", written once, read every run).
4. **Connectors (MCP)** — reach into real tools, at least privilege (read + comment
   until the loop earns trust; never merge/delete scopes on day one).
5. **Sub-agents, maker/checker split** — the implementer never grades its own work.
   The verifier runs in a separate context with a REJECT-by-default stance, executes
   the actual tests, and reports their output — "looks good" is verifier theater.
6. **Memory/state** — a durable spine outside any conversation (files below). A loop
   without a state file has amnesia every run.

## The autonomy ladder — never skip L1

| Level | Meaning | Requires |
|---|---|---|
| **L0 draft** | Documented intent only | Purpose, non-goals, watched scope |
| **L1 report** | Triage → state file; no auto-action | + schedule, triage skill, state |
| **L2 assisted** | Small auto-fixes behind a verifier | + maker/checker, worktrees, human gates, attempt caps |
| **L3 unattended** | Runs without you watching | + denylist, budget + kill switch, run log, metrics |

Phased rollout is mandatory: **week one is always L1 report-only** on a production
repo. Measure triage accuracy (false-positive rate) before enabling L2. L3 only after
the full safety pre-flight (below). Promoting a loop is a human decision recorded in
LOOP.md — never the loop's own.

## The operational file set

Repo-root files — the loop's durable spine:

| File | Role |
|---|---|
| `LOOP.md` | The loops this repo runs: pattern, cadence, level, gates, schedule |
| `STATE.md` | Durable loop state: high-priority (acting / waiting-on-human), watch list, ignored noise, last-run stamp |
| `loop-run-log.md` | Append-only run history: items found, actions, escalations, token estimate |
| `loop-budget.md` | Daily caps per loop, on-exceed procedure, kill switch — **only humans edit caps** |
| `loop-constraints.md` | Binding rules read at the start of every run (push/merge, paths, code, communication, budget) |
| `gate.yaml` | Machine-readable path denylist + auto-merge allowlist + max-files |

Rules that keep state honest:

- The loop **reads state at start, writes outcomes at end**, and **prunes**
  closed/merged/stale items every run — state rot makes loops act on ghosts.
- One state file per pattern (`ci-sweeper-state.md`, …); `STATE.md` belongs to triage.
  Shared unstructured state across loops is how loops fight each other.
- Structured, one-line items with an explicit suggested action — narrative paragraphs
  are unparseable by the loop and unread by humans.
- State files are committed — never write secrets into them; redact CI logs at triage.

## Budget and cost

- **Cheap triage first, spawn only on signal.** The triage pass must exit in a few
  thousand tokens when the watchlist is empty; sub-agent chains fire only for
  actionable items. Cadence is a linear cost multiplier (5m vs 1d = 288x runs/day).
- Daily token caps per loop in `loop-budget.md`; at 80% switch to report-only; on
  exceed, pause schedulers and notify. Agents may *request* a raise through a gate —
  they can never edit their own caps.
- Kill switch: a documented flag/label plus the pause criteria (production incident in
  progress, migration underway, false positives above ~30%, same item escalated twice
  in 48h). Kill a loop when cost exceeds value for two consecutive weeks or the team
  mutes its notifications.
- Notify humans only when a decision is required; digest everything else. A bot that
  pings every run gets muted, and then real escalations die (see references/failure-modes.md).

## Multi-loop coordination

1. **One owner per branch** — at most one loop mutates a branch at a time; each action
   loop writes `acting_on:` in its state file and checks the others' before spawning.
2. Priority when loops collide: CI sweeper (red main blocks all) → PR babysitter →
   dependency sweeper (pause while CI red) → post-merge cleanup (off-peak) → daily
   triage (report-only; schedules the others).
3. Shared denylist across every loop; aggregate token budget across all of them.

## Safety (minimum bar for anything that touches code)

- **Path denylist** (encode in `gate.yaml` AND the skills): `.env*`, secrets/,
  credentials/, key/secret globs, terraform/, k8s production, migrations/, auth/,
  payments/, billing/.
- **Auto-merge default off.** If allowed: docs typos, lint fixes in test files, import
  ordering — never behavior changes, dependency bumps, lockfiles, or denylisted paths.
- **Human gates always**: security/auth, payments/PII, infra, dependency majors and
  high-severity CVEs, diffs touching more than ~10 files, and the third failed attempt
  on the same item.
- **Attempt caps are mechanical**, not aspirational: record the count in state, cap at
  3, escalate with full context. "Keep trying until CI is green" is the classic
  infinite loop.
- **Flakes are classified, not fixed**: quarantine via ticket + human approval. Never
  disable tests or blindly raise timeouts to go green.
- Incident response: pause all loops, revert, record, tighten the verifier or shrink
  scope before restarting.

## Claude Code mapping

- Schedulers: `/loop` for in-session cadence, scheduled cloud agents/routines for
  cron-style runs, GitHub Actions for CI-coupled loops.
- Maker/checker: separate subagents; give the verifier a fresh context and, for
  unattended loops, a stronger model or higher reasoning effort.
- Worktrees: per-attempt isolation for any loop that edits files.
- The triage → act → verify shape of each run is a task graph — design it with the
  **task-graphs** skill; give each run's internals the **context-loops** treatment.

## Reference Files

- [references/patterns.md](references/patterns.md) — the seven production patterns
  (daily triage, issue triage, PR babysitter, CI sweeper, dependency sweeper,
  changelog drafter, post-merge cleanup): goal, cadence, risk, phases, state shape,
  gates, cost profile. Read when picking or scaffolding a pattern.
- [references/failure-modes.md](references/failure-modes.md) — the incident catalog
  (infinite fix loop, state rot, verifier theater, token burn, over-reach,
  comprehension debt, parallel collision, escalation failure) and the design
  anti-patterns behind them. Read when a loop misbehaves or before enabling L2/L3.
- [references/design-checklist.md](references/design-checklist.md) — the 10-section
  ship-readiness checklist mapped to L0-L3, plus the red flags that stop a rollout.
  Read before enabling any loop and during /loop-audit.

## Credits

This skill is an original adaptation for Claude Code, inspired by Cobus Greyling's
loop-engineering reference (https://github.com/cobusgreyling/loop-engineering, MIT)
and Addy Osmani's harness / factory / intent-debt framing.
