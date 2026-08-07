# Loop Failure Modes & Anti-Patterns

How operational loops actually fail — read when a loop misbehaves, and before
promoting one past L1. Severity: **S1** wastes time/tokens, **S2** ships harm (wrong
merges, alert fatigue), **S3** is critical (security, data loss, production).

## Runtime failure catalog

**Infinite fix loop (S2).** Same PR/CI job gets fix attempts forever; never converges.
Causes: verifier too weak or sharing the implementer's session; symptom-fixing a
misdiagnosed root cause; flake treated as regression. Fix: hard attempt cap (3) with
the count recorded in state → escalate; verifier in a separate context; classify
flakes at triage and quarantine instead of patching.

**State rot (S1→S2).** State file references merged PRs, closed tickets, stale
branches — the loop acts on ghosts. Causes: no prune step, state not read at run
start, multiple loops writing one unstructured file. Fix: prune resolved items every
run; last-run timestamp; validate IDs against the live source; one state file per
pattern.

**Verifier theater (S2).** Verifier "approves" but CI fails or review finds obvious
bugs. Causes: vague verifier prompt; verifier doesn't actually run tests; same
model/context as the implementer. Fix: verifier must execute test/lint commands and
quote their output; instructions say "find reasons to reject"; stronger model or
higher effort on the verifier for unattended loops.

**Notification fatigue (S1→S2).** Pings every run; team mutes the bot; real
escalations die unseen. Fix: notify only when a human decision is required; digest
mode for report loops; tighten the "high priority" bar in triage.

**Token burn (S1).** Bill spikes from full sub-agent chains on empty or noisy triage.
Fix: cheap triage-only pass first; spawn sub-agents only for actionable items; early
exit in a few thousand tokens on an empty watchlist; daily budget that pauses the
loop; delete the schedule when there's nothing to watch.

**Over-reach (S2→S3).** Loop refactors unrelated modules or touches denylisted paths.
Fix: denylist enforced in the skills AND mechanically (gate file); smallest-possible-
diff rule; verifier checks the touched-file list; triage reports signal, never invents
work.

**Comprehension debt spiral (S2, long-term).** Velocity up, but nobody can explain
recent changes; review becomes rubber-stamp. Fix: mandatory human review for
non-trivial PRs; a weekly digest the owner actually reads; auto-merge capped to truly
trivial paths. Related trap — *cognitive surrender*: "the loop handles it" replaces
having opinions. Success metric is time saved **with the quality bar held**, never
volume shipped.

**Parallel collision (S2).** Two agents edit the same files; conflicts or corrupted
state. Fix: worktree isolation for every code-editing sub-agent; `acting_on:` lock in
each action loop's state, checked before spawning.

**Escalation failure (S2).** Loop stuck retrying; no human ever notified. Fix:
escalation pings a channel humans watch, not just a state file; a
"waiting on human" section in state with an alert when an item sits over 24h.

## Design anti-patterns (catch these before enabling)

1. **Same agent implements and verifies** — confirmation bias rubber-stamps weak
   tests. Separate verifier, REJECT by default.
2. **No attempt cap** — "keep trying until green" is the infinite loop.
3. **Vague triage output** — narrative paragraphs; neither loop nor human can act.
   Structured one-line items with an explicit suggested action.
4. **L3 before L1 quality** — auto-fix on day one acts on bad signal. Report-only
   first; measure triage accuracy.
5. **Shared state without schema** — three loops appending to one file = state rot.
6. **Write-everything connector scopes on day one** — blast radius of one bad triage
   decision. Read-only until trust is earned.
7. **No kill switch** — no pause criteria means weekend incidents and budget overrun.
8. **Fixing flakes with code** — masks infra problems, introduces random diffs.
   Classify → quarantine → escalate.
9. **Auto-merge without an allowlist** — any path, any change, no human. Default off.
