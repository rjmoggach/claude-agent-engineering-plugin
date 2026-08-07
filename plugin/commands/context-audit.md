---
description: Audit an agent setup's context hygiene — window composition, degradation risk, tool surface, cache structure
argument-hint: [path to repo/agent config, or describe the setup and symptoms]
---

Act as the Agent Engineer auditing context hygiene (agent-engineering plugin; follow
the context-engineering skill — read its references/degradation.md and
references/tool-design.md first).

Target: $ARGUMENTS

If no target was given, audit the current repo's agent surface: CLAUDE.md/AGENTS.md,
`.claude/` (skills, agents, commands, settings), MCP/tool configuration, and any
subagent or workflow definitions. Read the actual files before critiquing. If the
user described symptoms, map them to a degradation pattern first.

Audit in this order:

1. **Window composition.** Estimate what loads at session start and per turn: system
   prompt and always-on instructions, skill/agent descriptions, tool definitions,
   memory/context files. Flag the heaviest items and anything loaded eagerly that
   could load just-in-time.
2. **Placement.** Are critical constraints at attention-favored positions (start/end)
   or buried mid-file? Are there monster files where the load-bearing rule sits in
   the middle?
3. **Degradation risk.** For each of the five patterns — lost-in-middle, poisoning,
   distraction, confusion, clash — say whether this setup invites it and where
   (e.g. contradictory instructions across CLAUDE.md files = clash; unvalidated tool
   output flowing into state files = poisoning vector).
4. **Tool surface.** Apply the tool-design checklist: overlapping tools, descriptions
   missing what/when/inputs/returns, non-actionable errors, inconsistent naming,
   missing namespacing, reduction candidates.
5. **Offloading and compression.** What accumulates in-window that should be written
   to files with pointers? Are there compaction triggers, or does the setup ride
   toward the cliff? Do long-running flows re-validate summaries after compaction?
6. **Cache stability.** Dynamic content (dates, counters) interpolated into stable
   prefixes? Stable content ordered first?

Then return:

- A context budget table: component, estimated tokens, verdict (keep / trim / defer /
  offload).
- Findings most-severe first: the file/tool, the defect, the degradation pattern it
  invites, the concrete fix.
- The single highest-leverage change.

Save the report to `context/graph/audits/context-audit.md` (create the folder if
needed). This is a review — change nothing unless I ask.
