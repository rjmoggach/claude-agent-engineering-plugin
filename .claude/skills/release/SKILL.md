---
name: release
description: >-
  Ship a graph-engineering plugin release end to end. Trigger: "release vX.Y.Z",
  "ship the release", "cut a release", "/release". Runs the full pipeline — validate gate,
  4-place version bump, CHANGELOG, commit (no Co-Authored-By), tag, push, package,
  GitHub release, and EXPLICIT issue closes (never rely on "Closes #N") — then verifies a
  clean tree. Use after the code/doc changes for a release are already made and validated.
---

# Release (graph-engineering)

Take an already-implemented, already-validated change from "ready" to "shipped" in one
pass. Do the steps in order; stop and report if any gate fails.

Repo: `rjmoggach/claude-graph-engineering-plugin`. Never force-push `main`.
**No `Co-Authored-By` trailer** on the commit. This repo has no build step — what's in
`plugin/` is what ships.

## Inputs (confirm before starting)
- **Version** `vX.Y.Z` — **patch** for a behavioral fix/clarification, **minor** for a new
  skill/agent/command, **major** only on a breaking change. If unstated, infer from the
  change and state your choice.
- **Issue numbers** this release closes (if any). GitHub auto-closes **only the first**
  of a `Closes #A, #B` list — you will close them all by hand in Step 7 regardless.

## Step 1 — Validate gate (must pass before anything else)
```
claude plugin validate plugin 2>&1 | tail -2
claude plugin validate . 2>&1 | tail -2
grep -rn '<[a-zA-Z]' plugin/skills plugin/agents && echo ANGLE_TAGS || echo clean
```
Require **`Validation passed`** from BOTH validates with **no warnings** (a warning like
`display_name → displayName` is a failure here), and `clean` from the angle-tag grep
(angle-bracket placeholders must be `{id}`, never a bare tag). Fix and re-run before
continuing.

## Step 2 — Bump the version in all 4 locations
Today's date is `date +%F`. Bump every occurrence (marketplace.json has **two**):
- `README.md` — `**Version**: X.Y.Z · **Updated**: {today}`
- `plugin/.claude-plugin/plugin.json` — `"version": "X.Y.Z"`
- `.claude-plugin/marketplace.json` — `"version": "X.Y.Z"` (**2x**: metadata + plugins[0])

Verify none of the old version string remains:
```
grep -rn '{OLD}' README.md plugin/.claude-plugin/plugin.json .claude-plugin/marketplace.json \
  && echo OLD_REMAINS || echo bumped-clean
```

## Step 3 — CHANGELOG entry (top, under the intro line)
Insert a new section directly above the previous version:
```
## vX.Y.Z - {today}

### {Title} (#A, #B, …)

{1-3 sentences of what changed and why, per area.}

Closes #A, #B, #C.
```
Keep it faithful to the actual diff. The `Closes` line documents intent — it does **not**
auto-close (Step 7 does).

## Step 4 — Integrity check (version survived, JSON parses, nothing truncated)
```
python3 -c "import json;json.load(open('.claude-plugin/marketplace.json'));json.load(open('plugin/.claude-plugin/plugin.json'));print('json ok')"
grep '"version"' plugin/.claude-plugin/plugin.json .claude-plugin/marketplace.json
for f in .claude-plugin/marketplace.json README.md CHANGELOG.md; do [ -s "$f" ] || echo "EMPTY $f"; done
```
Confirm `json ok`, the bumped version in all three JSON slots, and no `EMPTY` lines.

## Step 5 — Commit + tag (no Co-Authored-By)
```
git add -A
git status --porcelain | grep -iE 'superpowers|scratch|\.plugin$' && echo "LEAK — unstage first"
git commit -q -m "feat: vX.Y.Z — {short summary}"   # fix: for a patch
git tag vX.Y.Z
git push origin main --tags 2>&1 | tail -2
```

## Step 6 — Package + GitHub release
```
(cd plugin && zip -qr ../graph-engineering.plugin . -x '*.DS_Store')
notes="$(awk '/^## vX.Y.Z/{f=1;next} /^## v/{if(f)exit} f' CHANGELOG.md)"
gh release create vX.Y.Z -R rjmoggach/claude-graph-engineering-plugin \
  --title "vX.Y.Z — {title}" --notes "$notes" graph-engineering.plugin | tail -1
rm -f graph-engineering.plugin
```

## Step 7 — Close every issue EXPLICITLY
Do **not** trust `Closes #N` — close each one with a release link:
```
for n in A B C; do
  gh issue close "$n" -R rjmoggach/claude-graph-engineering-plugin \
    -c "Shipped in vX.Y.Z — https://github.com/rjmoggach/claude-graph-engineering-plugin/releases/tag/vX.Y.Z" \
    && echo "closed #$n"
done
```
Leave any deliberately-deferred issue **open** (say which, and why). Skip this step when
the release references no issues.

## Step 8 — Verify + report
```
gh release view vX.Y.Z -R rjmoggach/claude-graph-engineering-plugin --json tagName,assets --jq '{tag:.tagName,assets:[.assets[].name]}'
gh api repos/rjmoggach/claude-graph-engineering-plugin/contents/.claude-plugin/marketplace.json --jq .size   # > 0 or installs break
git status --porcelain | wc -l    # expect 0
```
Report: release URL, the `.plugin` asset present, marketplace.json non-empty on remote,
issues closed (and any left open on purpose), and **tree clean (0)**. If anything didn't
verify, say so — don't claim done.

## Critical rules
1. Validate gate is a hard gate — never bump/commit on a failed or warning validate.
2. Bump **all 4** version slots (marketplace has two); grep to prove the old string is gone.
3. **No `Co-Authored-By`.** Never force-push `main`.
4. **Always `gh issue close` explicitly** — `Closes #N` in a commit only closes the first.
5. Clean up `graph-engineering.plugin` after the release uploads it (it's gitignored, but
   never leave it in the tree).
6. Report honestly: verified numbers, not assertions.
