# debutant — developer guide

How to extend `debutant` with a new worker, fixture, or reference
doc.

`debutant` is packaged as a Claude Code plugin. The plugin
manifest is `.claude-plugin/plugin.json`; everything else
(skills, scripts, docs, tests) sits at the plugin root. Skills
become slash commands under the `/debutant:` namespace —
`skills/lintian/SKILL.md` ⇒ `/debutant:lintian`, and so on. The
orchestrator lives at `skills/run/` and is invoked as
`/debutant:run`.

## Adding a new worker

A worker is a Claude Code skill — a directory under `skills/`
containing at least `SKILL.md`.

Steps:

1. **Pick a name.** Workers are named after their phase:
   `bootstrap`, `refresh`, `lintian`, `autopkgtest`. Pick a short
   verb or noun; the slash command will be `/debutant:<name>`.
2. **Create the skill directory.**
   ```
   mkdir -p skills/<name>
   ```
3. **Write `SKILL.md`** with frontmatter:
   ```yaml
   ---
   name: <name>
   description: One sentence the orchestrator (and skill discovery)
     will use to decide when to invoke this. Lead with the user
     intent, not the implementation. Mention "Debian" or "Debian
     source package" to anchor scope.
   ---
   ```
   The body MUST cover:
   - Preconditions (what must be true about the context, including
     the "if context.json is missing, build it via the probe
     scripts" recovery path).
   - Hard rules — both the suite-wide ones (inlined, see below)
     and any phase-specific bans.
   - Process (numbered steps from input to output).
   - Bail-out conditions.
4. **Inline the suite-wide hard rules.** Each worker's `## Hard
   rules` section opens with the suite-wide list verbatim, then
   adds phase-specific rules. The canonical list lives in
   `shared-context.md`. If you change one rule, update all five
   worker `SKILL.md` files and the spec file in the same commit.
5. **Reference shared assets by `${CLAUDE_PLUGIN_ROOT}`.** The
   plugin runtime substitutes that variable before the LLM sees
   the prompt. Always use it; never use `../<sibling-skill>/...`
   — that path will not resolve after the plugin is installed.

   Common references:
   - `${CLAUDE_PLUGIN_ROOT}/scripts/detect-source.sh`
   - `${CLAUDE_PLUGIN_ROOT}/scripts/tooling-probe.sh`
   - `${CLAUDE_PLUGIN_ROOT}/scripts/verify.sh`
   - `${CLAUDE_PLUGIN_ROOT}/shared-context.md`
   - `${CLAUDE_PLUGIN_ROOT}/docs/house-style.md`
   - `${CLAUDE_PLUGIN_ROOT}/docs/references/*.md`
6. **Cite the house style.** Every prescriptive choice in the
   worker output must trace back to
   `${CLAUDE_PLUGIN_ROOT}/docs/house-style.md` or
   `${CLAUDE_PLUGIN_ROOT}/docs/references/*.md`.
7. **Update the orchestrator.** Add the phase to the dispatch
   table in `skills/run/SKILL.md`.
8. **Add a fixture** under `tests/fixtures/` that exercises the
   worker end-to-end (or annotate an existing fixture to cover
   the new phase).
9. **Update the README phase list.**

## Adding a fixture

See `tests/fixtures/README.md`. Each fixture is a minimal
upstream tree plus an `expected/` golden output and a `test.sh`
driver. `tests/run-fixtures.sh` walks the fixtures directory and
runs each fixture's `test.sh`.

## Adding a reference doc

Reference docs live under `docs/references/`. Keep them:

- **Short.** One page each. Cheatsheets, not textbooks.
- **Opinionated.** State the preferred choice and why.
- **Cited.** Link to the canonical source (Debian Policy,
  devref, DEP, upstream manpage).
- **Dated.** Add `**Last reviewed**: YYYY-MM-DD` at the top.

## Quarterly review checklist

The house style and references go stale. Quarterly, check:

- [ ] `Standards-Version` default in `house-style.md` matches
      both `skills/bootstrap/templates/control.tmpl` and the
      current Debian Policy package version.
- [ ] `debhelper-compat` default in `house-style.md` matches
      `skills/bootstrap/templates/control.tmpl` and current
      debhelper.
- [ ] Salsa-CI pinned ref in `docs/references/salsa-ci.md` and
      `skills/bootstrap/templates/salsa-ci.yml.tmpl`.
- [ ] Each `docs/references/*.md` `Last reviewed` date < 6 months.
- [ ] Any new accepted DEPs.

**Drift hot-spots.** A handful of values are pinned in
`docs/house-style.md` AND duplicated in
`skills/bootstrap/templates/`. They must move together:

| Value | Files |
|---|---|
| `debhelper-compat` version | `docs/house-style.md` (debhelper §) + `skills/bootstrap/templates/control.tmpl` |
| `Standards-Version` | `docs/house-style.md` (Control fields §) + `skills/bootstrap/templates/control.tmpl` |
| Salsa-CI pinned ref | `docs/references/salsa-ci.md` + `skills/bootstrap/templates/salsa-ci.yml.tmpl` |
| Watch syntax version | `docs/house-style.md` (debian/watch §) + `skills/bootstrap/templates/watch.tmpl` |

`debian/control` has no comment syntax, so no inline reminder
lives in the template — the discipline is here.

## Style for SKILL.md prompts

- **Address the LLM.** "Run X, then Y, then ask the maintainer."
- **Bullet rules, not paragraphs.** Easier for the model to
  attend to.
- **Hard rules in a dedicated section** titled "Hard rules". Use
  the same phrasing across workers so the prompts reinforce.
- **Examples for fiddly formats.** E.g. show what a good
  `lintian-overrides` comment looks like; the model will copy
  the shape.
- **Avoid hedging in instructions.** "MUST" / "MUST NOT" / "MAY"
  / "SHOULD" with their RFC-2119 meanings.

## Testing changes locally

```
# Load the plugin from this checkout for the current session:
claude --plugin-dir /path/to/debutant

# Then invoke a worker as a slash command:
/debutant:lintian
```

After editing a skill, run `/reload-plugins` to pick up changes
without restarting Claude Code.

The workshop's `claude-exec` action wires the equivalent
non-interactive form — see `workshop.yaml`.

## Plugin manifest

`.claude-plugin/plugin.json` carries the plugin name, description,
version, and author. The `name` field controls the slash-command
namespace (`/debutant:<skill>`). Bump `version` on every release
that ships to users — Claude Code uses it to decide whether to
re-fetch the plugin.
