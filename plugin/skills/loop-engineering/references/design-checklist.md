# Loop Design Checklist

Score honestly before enabling a loop — a loop missing verification is not ready for
unattended runs. Sections map to the autonomy ladder at the bottom; /loop-audit walks
this list.

## 1. Purpose & scope
- Single clear goal, one sentence.
- Explicit non-goals — what this loop will NOT do.
- Watched scope: which repos, branches, PRs, tickets.
- Phased rollout planned: report-only first.
- Ambiguous input handled: too-vague items get clarified or escalated, never guessed.

## 2. Scheduling
- Cadence matches urgency (and its cost multiplier).
- First-run behavior decided (fire immediately vs wait one interval).
- Survives restarts if it needs to.
- Off-hours behavior: slower or paused.
- Self-cleanup: schedule removed when the watchlist is empty.

## 3. Skills
- Triage skill exists with a tight, structured output format.
- Action skills match project conventions.
- Build/test commands documented where the loop reads them.

## 4. Maker / checker split
- Implementer and verifier are separate (agent, model, or at minimum instructions).
- Implementer cannot mark its own work done.
- Verifier runs the tests in isolation (worktree) before approving.

## 5. State / memory
- State file schema documented; loop reads it at start, writes outcomes at end.
- Prune of resolved/merged/closed items every run.
- Human overrides recorded in state.

## 6. Human handoff
- Escalation triggers explicit: attempt cap, risk paths, ambiguity.
- Denylist paths listed (auth, payments, secrets, infra, migrations).
- Notification rule: ping only when action is required.
- An inbox where ambiguous items land — and that a human actually watches.

## 7. Connectors
- Minimum permissions (read/comment before write; never merge/delete on day one).
- Bot identity clear on anything it posts.

## 8. Cost & limits
- Token budget estimated; daily caps in the budget file with a kill switch.
- Append-only run log.
- Max iterations per item, max auto-actions per day.
- Pause and kill criteria defined.

## 9. Observability
- Every run logged: items found, actions taken, escalations, token estimate.
- Success metrics chosen (per pattern).
- Team can inspect state without reading chat logs.

## 10. Safety
- No auto-merge without an explicit allowlist.
- Secrets/env files in the denylist.
- Flake handling: classify and quarantine, never retry-to-green.

## Readiness levels

| Level | Description | Requires sections |
|---|---|---|
| **L0 draft** | Documented intent only | 1 |
| **L1 report** | Triage → state, no auto-action | 1-3, 5 |
| **L2 assisted** | Small auto-fixes with verifier | 1-7 |
| **L3 unattended** | Runs without you watching | all ten |

## Red flags — stop and fix before continuing

- Same item has had 3+ automated fix attempts without progress.
- Verifier shares the implementer's session.
- No state file — the loop has amnesia every run.
- Notifications fire every run regardless of findings.
- Auto-merge enabled without a path allowlist.
