# AGENTS.md — Repo Standards & Release Runbook

Guidance for any agent (or human) working on **claude-graph-engineering-plugin**. This
repo ships a **Claude Code / Claude Desktop plugin** (the Graph Engineering superpower:
task graphs, context loops, knowledge graphs) plus a marketplace entry. Everything is
hand-maintained Markdown/JSON — there is **no build or assemble step**; what's in
`plugin/` is exactly what installs.

---

## 1. Layout — everything is canonical, edit in place

| Path | Role |
|---|---|
| `plugin/.claude-plugin/plugin.json` | Plugin manifest |
| `plugin/agents/*.md` | Agent personas (`graph-engineer`) |
| `plugin/skills/*/SKILL.md` (+ `references/`) | Skills: `task-graphs`, `context-loops`, `knowledge-graphs` |
| `plugin/commands/*.md` | Slash commands (`/kg-*`, `/graph-audit`) |
| `.claude-plugin/marketplace.json` | Marketplace entry (`/plugin marketplace add`) |
| `graph-engineering/` | **Vendored upstream reference** (@Av1dlive's skill + course distillation). Read-only source material — not shipped, don't edit |
| `README.md`, `plugin/README.md`, `CHANGELOG.md` | Docs |
| `.claude/skills/` | Repo-local workflow skills (`release`, `file-gaps`) — not part of the plugin |

There are no generated directories. Edit the file you mean; nothing overwrites it.

---

## 2. Validate gate (mandatory before every release)

```bash
claude plugin validate plugin    # plugin manifest + components
claude plugin validate .         # marketplace manifest
```

Both must print **`✔ Validation passed`** — warnings count as failures for release
(e.g. unknown manifest fields like `display_name` vs `displayName`). Also enforce the
plugin-loader rules that have bricked real installs elsewhere:

- skill/agent `description` frontmatter **≤ 1024 characters**
- **no XML/angle-bracket tags** (`<...>`) in any skill or agent body — use `{placeholder}`
  (check: `grep -rn '<[a-zA-Z]' plugin/skills plugin/agents`)
- agent frontmatter is strict YAML with keys `name, description, model, color, tools`
- `plugin.json` `name` is kebab-case
- **no empty component dir** under `plugin/` (an empty `agents/`/`skills/`/`commands/`
  bricks install)
- every relative link in a SKILL.md (e.g. `references/modeling.md`) resolves to a file

Integrity check (JSON parses, nothing truncated):

```bash
python3 -c "import json;json.load(open('.claude-plugin/marketplace.json'));json.load(open('plugin/.claude-plugin/plugin.json'));print('json ok')"
for f in .claude-plugin/marketplace.json README.md CHANGELOG.md; do [ -s "$f" ] || echo "EMPTY $f"; done
```

---

## 3. Version lives in FOUR places — bump all of them

1. `plugin/.claude-plugin/plugin.json` → `"version"`
2. `.claude-plugin/marketplace.json` → **both** `metadata.version` **and**
   `plugins[0].version`
3. `README.md` → the `**Version**: X.Y.Z · **Updated**: YYYY-MM-DD` line
4. `CHANGELOG.md` → prepend a new `## vX.Y.Z - YYYY-MM-DD` section

Edit these with the Read/Edit/Write tools (see Gotchas §6).

---

## 4. Release runbook (exact sequence)

The `.claude/skills/release` skill encodes this end to end — prefer `/release`.

```bash
# 1. Validate gate (§2) — both validates pass, no warnings

# 2. Bump the 4 version locations + prepend CHANGELOG (§3)
grep -rn '<OLD_VERSION>' README.md plugin/.claude-plugin/plugin.json .claude-plugin/marketplace.json && echo OLD_REMAINS

# 3. Commit, tag, push
git add -A
git status --porcelain | grep -iE 'superpowers|scratch' && echo "LEAK — unstage before commit"
git commit -m "feat: vX.Y.Z — <summary>"
git tag vX.Y.Z
git push origin main --tags

# 4. Package + GitHub release
(cd plugin && zip -r ../graph-engineering.plugin . -x '*.DS_Store')
notes="$(awk '/^## vX.Y.Z/{f=1;next} /^## v/{if(f)exit} f' CHANGELOG.md)"
gh release create vX.Y.Z -R rjmoggach/claude-graph-engineering-plugin \
  --title "vX.Y.Z — <title>" --notes "$notes" graph-engineering.plugin
rm -f graph-engineering.plugin

# 5. Close referenced issues EXPLICITLY (never rely on "Closes #N"), then verify
gh release view vX.Y.Z -R rjmoggach/claude-graph-engineering-plugin --json tagName --jq .tagName
git status --porcelain    # expect empty
```

**After any manifest change, verify the remote is install-ready:**

```bash
gh api repos/rjmoggach/claude-graph-engineering-plugin/contents/.claude-plugin/marketplace.json --jq .size   # must be > 0
```

An empty `marketplace.json` on `main` silently breaks `/plugin marketplace add` for
everyone.

---

## 5. Commit & issue conventions

- **No `Co-Authored-By` trailer.** None. (Explicit maintainer preference.)
- Conventional-commit prefixes: `feat:` / `fix:` / `docs:` / `chore:`.
- Commit **directly to `main`** for releases. **Never force-push `main`.**
- When an issue exists for a change, reference it in the commit body — but always
  `gh issue close` explicitly at release time; `Closes #N` only auto-closes the first.
- Versioning: **patch** for fixes/consistency, **minor** for new skills/agents/commands,
  **major** for breaking changes. The maintainer decides the number — propose, don't
  assume. Docs-only changes are a `docs:` commit with no version bump and no release.

---

## 6. Gotchas / hard rules

- **Never `open(f,'w').write(open(f).read()...)`** in one-liners — the write-mode open
  truncates the file to 0 bytes before the read runs. This has shipped empty
  `README.md`/`marketplace.json` files in a sibling repo. Use Read/Edit/Write tools.
- **`marketplace.json` carries the version in two fields** — miss one and the
  marketplace entry disagrees with the plugin manifest.
- **`*.plugin` and `*.skill` are gitignored** — the release artifact must never be
  committed; it's uploaded to the GitHub release then deleted.
- The marketplace schema uses **camelCase** (`displayName`) — `claude plugin validate .`
  warns on `display_name`; treat warnings as failures.
- **Don't edit `graph-engineering/`** — vendored upstream reference; our shipped
  adaptations live in `plugin/skills/`.
