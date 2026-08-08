---
description: Run the full audit gamut on an agent system or plugin — harness, task graph, context, loops — merged into one report
argument-hint: [path to the repo/plugin, or describe the system and its symptoms]
---

Act as the Agent Engineer running the complete audit gamut (agent-engineering
plugin). This command composes the four single-discipline audits; each one's
protocol is canonical and lives in its own command file — follow them as written:
`/harness-audit` (harness-engineering skill), `/graph-audit` (task-graphs +
context-loops), `/context-audit` (context-engineering), `/loop-audit`
(loop-engineering).

Target: $ARGUMENTS

If no target was given, audit the current repo. Read the actual files before
critiquing anything.

## Step 1 — Scout, cheaply

One quick pass to shape the run:

- Is the target a plugin (`.claude-plugin/`, or `agents/`/`skills/`/`commands/`
  directories)? Then /graph-audit runs in its plugin-target mode, and every fix
  must be a self-contained rewrite in the target's own files — the target must
  work for users who never installed this plugin.
- Is there loop evidence (`LOOP.md`, `STATE.md`, `loop-*.md`, `gate.yaml`,
  `.github/workflows/`, scheduled routines)? If none, skip the loop audit and say
  so in the report rather than inventing findings.
- Are there prior reports under `context/graph/audits/`? If so, this is a re-run:
  hand each auditor its predecessor report, and its first job is verifying that
  every previously claimed fix actually held — a fix that didn't hold outranks a
  new finding of the same severity. Then it audits what changed since.

## Step 2 — Run the audits as a diamond

The four audits are independent reviews; none reads another's output. If subagents
are available, fan them out in parallel — one audit per subagent, each in an
isolated context carrying only: the target path, its one audit protocol, and the
return shape below. Subagents do not see the sibling command files on their own —
give each one its protocol verbatim in the spawn prompt, or the path to its
command file to read first. Hard cap: 4 subagents, no re-spawning. If subagents
are not available, run the protocols sequentially in this order: harness, graph,
context, loop.

Tell every auditor that finding-count is not a quality signal: an honest
three-finding report beats a padded twelve-finding one, and template-filling —
findings invented to satisfy a checklist section that doesn't apply to this
target — is a defect in the audit itself.

Every audit returns findings as structured items, never free prose:
`{file/component, defect, discipline, severity, concrete fix, what breaks if left}`
plus its discipline's summary artifact (the six-layer scorecard, the drawn
topologies, the context budget table, the loop readiness table).

## Step 3 — Merge (you are the single owner)

- **Dedupe across lenses.** Tool-surface defects surface in both the harness and
  context audits; self-verification in both harness and graph; state hygiene in
  both harness and loop. Keep one finding per defect, note every lens that caught
  it (two lenses agreeing raises severity).
- **Resolve contradictions** between audits explicitly instead of shipping both
  recommendations.
- **Rank** the merged list most-severe first.

## Step 4 — Return

- The six-layer scorecard with missing layers named.
- The drawn current and target topologies (from the graph audit).
- The merged findings table, most severe first, each with its concrete fix.
- A suggested fix order — cheapest high-severity fixes first, and which fixes
  unblock others.
- The single highest-leverage change across all four disciplines.

Save each discipline's full report to its standard path under
`context/graph/audits/` (`harness-audit.md`, `{target-name}-audit.md`,
`context-audit.md`, `loops-audit.md`) and the merged summary to
`context/graph/audits/agent-audit.md` (create the folder if needed), so a re-run
after fixes can diff against every layer. This is a review — change nothing unless
I ask.
