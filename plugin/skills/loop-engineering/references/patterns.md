# The Seven Production Loop Patterns

*(Adapted from field practice — see the skill's credits. Each pattern names its goal,
cadence, risk, phases, state file, human gates, and cost profile — everything
/loop-design needs to scaffold one.)*

Pick by risk appetite and where your time actually goes. Low-risk report-only patterns
(daily triage, issue triage, changelog drafter, post-merge cleanup) are the right first
loop; the medium-risk actors (CI sweeper, PR babysitter, dependency sweeper) come after
a pattern has proven its triage accuracy.

## Daily Triage — the anchor pattern

- **Goal**: prioritized morning scan of CI, issues, commits, and chat.
- **Cadence** 1d-2h · **risk low** · **week one L1** · **cost low** (~50k/report run;
  suggested daily cap 100k).
- **Phases**: report → act-on-small-wins → escalate. **State**: `STATE.md`.
- **Gates**: design decisions, multi-file refactors.
- The L1 loop that schedules everything else; start here on any repo.

## Issue Triage

- **Goal**: discover, dedupe, prioritize, and label incoming issues/discussions so the
  queue stays clean and actionable. Low-risk companion to Daily Triage.
- **Cadence** 2h-1d · **risk low** · **week one L1** · **cost low** (~30k/report;
  cap 80k).
- **Phases**: discover → dedupe → score → propose-labels → human-review.
  **State**: `issue-triage-state.md`.
- **Gates**: security reports, P0/P1 calls, ambiguous duplicates, stale closures.

## Changelog Drafter

- **Goal**: scan merged PRs/commits and draft categorized release notes for human
  review (a draft file — never publishes).
- **Cadence** 1d or on release prep · **risk low** · **week one L1** · **cost low**
  (~35k/report; cap 100k).
- **Phases**: scan-merges → categorize → draft → review → publish (human).
  **State**: `changelog-drafter-state.md`.
- **Gates**: breaking changes, security notes, marketing-sensitive wording.

## Post-Merge Cleanup

- **Goal**: follow-up tech debt and cleanup after merges to main; off-peak, lowest
  urgency of the action loops.
- **Cadence** 1d-6h · **risk low** · **week one L1** · **cost low** (~40k/report;
  cap 200k).
- **Phases**: scan-merges → prioritize → fix-small → ticket-large.
  **State**: `post-merge-state.md`.
- **Gates**: architectural debt, feature flags, large diffs.

## Dependency Sweeper

- **Goal**: discover, safely apply, and verify dependency + vulnerability updates.
- **Cadence** 6h-1d · **risk medium** · **week one L2 patch-only** · **cost medium**
  (~60k/report, ~300k/action; cap 500k; **early exit required**).
- **Phases**: scan → triage-risk → patch-safe → verify-in-worktree → escalate-risky.
  **State**: `dependency-sweeper-state.md`.
- **Gates**: major bumps, high-severity CVEs, denylisted packages, attempt cap.
- First 30 days: patch + low-risk CVE only; verifier = full install + test suite in a
  worktree.

## CI Sweeper

- **Goal**: react to failing CI with minimal fixes and escalation. Highest urgency
  when active — red main blocks everything.
- **Cadence** 5-15m · **risk medium** · **cost very high** (a full run every 15m is
  ~5M tokens/day — **early exit on green CI is mandatory**; cap 1M).
- **Phases**: detect → classify → fix → verify → escalate.
  **State**: `ci-sweeper-state.md` with attempt counts.
- **Gates**: infra failures, security tests, attempt cap.
- Classification matters most: a flake gets quarantined via ticket, never "fixed" in
  application code.

## PR Babysitter

- **Goal**: shepherd PRs through review, CI, rebase, and merge.
- **Cadence** 5-15m active hours · **risk medium** · **week one L1 watch** ·
  **cost high** (~80k/report, ~250k/action; cap 2M; **early exit required**).
- **Phases**: discover → triage → fix → verify → notify.
  **State**: `pr-babysitter-state.md`.
- **Gates**: security, payments, auth paths, attempt cap. No auto-merge by default;
  skip a PR when another loop's state says it's acting on it.

## Scaffolding

/loop-design scaffolds the operational file set (LOOP.md, STATE.md, budget, run log,
constraints, gate) for whichever pattern fits. Whatever the pattern, week one runs
report-only and the human reads what the loop found before any fix is enabled.
