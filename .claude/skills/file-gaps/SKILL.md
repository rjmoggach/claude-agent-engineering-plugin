---
name: file-gaps
description: >-
  Turn a pasted batch of plugin gap notes (from real usage of the agent-engineering
  plugin) into cleanly-formatted GitHub issues on this repo. Trigger: "file these gaps",
  "log these as issues", "here are the gaps from the run", "/file-gaps", or any pasted
  markdown list of P0/P1/P2 gaps with "What happened / Fix" shapes. LOG ONLY — it files
  issues, it does not implement them. Dedupes against open issues and flags overlaps
  before creating anything.
---

# File-gaps (agent-engineering)

Rob pastes a markdown block of gaps surfaced by real use of the plugin (a workflow
design session, a `/graph-audit`, a knowledge-graph build, a `/kg-tutor` run, etc.).
Convert each distinct gap into one well-scoped GitHub issue in a consistent house format,
dedupe against what's already open, and report the created numbers. **Do not implement
anything** — this is intake, not a fix cycle.

Repo: `rjmoggach/claude-agent-engineering-plugin`.

## Step 1 — Parse the paste into distinct gaps
Split the block into individual gaps (usually one per `##`/`###` heading or bullet
group). For each, capture: a **priority** (P0/P1/P2 if given, else infer), a
**what-happened** (the observed symptom from the run), and a **fix** (the proposed
change) plus an **acceptance criterion** if stated. Preserve the run's own wording —
don't editorialize. If two notes are really one gap with two fixes, merge them; if one
note bundles two independent gaps, split them.

## Step 2 — Dedupe against open issues
```
gh issue list -R rjmoggach/claude-agent-engineering-plugin --state open --limit 60 \
  --json number,title -q '.[] | "\(.number)  \(.title)"' | sort -n
```
If a gap overlaps an existing open issue, **don't file a duplicate** — either fold it in
(note "extends #N" in the body) or skip it and say so in the report. Surface these
overlaps to Rob rather than silently deciding.

## Step 3 — House format (title + body per issue)
**Title:** `P{n}: {one-line gap}` for prioritized items, or `{area}: {one-line gap}` for
untyped ones — areas here are the plugin's components (e.g. `task-graphs:`,
`context-loops:`, `knowledge-graphs:`, `agent:`, `commands:`, `docs:`). Keep it scannable.

**Body:**
```
**Surfaced by** {the run — e.g. a /graph-audit of the ingest pipeline}. {P-level / type}.

{What happened — the observed symptom, faithful to the run's wording.}

**Fix.** {the proposed change, and which skill/agent/command/reference it lives in}.

**AC.** {acceptance criterion, if the note gave one — else omit}.
```
Add `*(deferred — {reason})*` on the first line for items explicitly parked (net-new
tooling, "later", etc.), and `Extends #N.` when it builds on an existing issue.

## Step 4 — File them (foreground, tolerant, capture numbers)
Write each body to a temp file and create with `--body-file` (heredocs survive
newlines/backticks that inline `--body` mangles). Foreground, not background — a
backgrounded nested-heredoc `gh` loop has silently created nothing before.
```
d="$(mktemp -d)"; w(){ cat > "$d/$1"; }
w 1.md <<'EOF'
{body for issue 1}
EOF
# … one per gap …
R=rjmoggach/claude-agent-engineering-plugin
gh issue create -R "$R" --title "{title 1}" --body-file "$d/1.md" 2>&1 | tail -1
# … one per gap …
rm -rf "$d"
```

## Step 5 — Report
Print a compact map: `#NN {one-line}` per filed issue, grouped by area/priority, and
call out:
- any gap **folded into** or **skipped as dup** of an existing issue (with the #N),
- any **deferred** item and why,
- the two or three **highest-leverage** issues to implement first (Rob's usual next
  question).

## Critical rules
1. **Log only — never implement** from this skill. Filing ≠ fixing.
2. **Dedupe first.** Check open issues before creating; fold or skip overlaps, don't
   duplicate.
3. **Faithful to the run.** Use the paste's own symptoms/fixes; don't invent AC or scope.
4. **Foreground `gh` with `--body-file`.** No backgrounded heredoc loops (they no-op
   silently).
5. Mark deferred items explicitly so they aren't mistaken for ready work.
