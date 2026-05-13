# debutant — developer guide

How to extend `debutant` with a new worker, fixture, or reference
doc.

## Adding a new worker

A worker is a Claude Code skill — a directory under `skills/`
containing at least `SKILL.md`.

Steps:

1. **Pick a name.** Workers are named `debutant-<phase>`. Phases
   so far: `bootstrap`, `refresh`, `lintian`, `autopkgtest`.
2. **Create the skill directory.**
   ```
   mkdir -p skills/debutant-<phase>
   ```
3. **Write `SKILL.md`** with frontmatter:
   ```yaml
   ---
   name: debutant-<phase>
   description: One sentence the orchestrator (and skill discovery)
     will use to decide when to invoke this. Be specific about the
     trigger.
   ---
   ```
   The body MUST cover:
   - Preconditions (what must be true about the context).
   - Hard rules (inherit from `shared-context.md`, plus
     phase-specific bans).
   - Process (numbered steps from input to output).
   - Bail-out conditions.
4. **Honour the shared context.** Read
   `skills/debutant/shared-context.md` and ensure the worker:
   - Reads context from `$DEBUTANT_CONTEXT` or
     `./.debutant/context.json`.
   - Builds context itself if neither exists.
   - Honours `budget.max_attempts_per_error_class` and
     `budget.diff_threshold_lines`.
   - Uses the bail-out summary format on failure.
5. **Cite the house style.** Every prescriptive choice in the
   worker output must trace back to `docs/house-style.md` or
   `docs/references/*.md`.
6. **Update the orchestrator.** Add the phase to the list in
   `skills/debutant/SKILL.md` and the dispatch logic.
7. **Add a fixture** under `tests/fixtures/` that exercises the
   worker end-to-end (or annotate an existing fixture to cover
   the new phase).
8. **Update the README phase list.**

## Adding a fixture

See `tests/fixtures/README.md`. Each fixture is a minimal
upstream tree plus an `expected/` golden output and a `test.sh`
driver.

## Adding a reference doc

Reference docs live under `docs/references/`. Keep them:

- **Short.** One page each. Cheatsheets, not textbooks.
- **Opinionated.** State the preferred choice and why.
- **Cited.** Link to the canonical source (Debian Policy,
  devref, DEP, upstream manpage).
- **Dated.** Add `**Last reviewed**: YYYY-MM-DD` at the top.

## Quarterly review checklist

The house style and references go stale. Quarterly, check:

- [ ] `Standards-Version` default in `house-style.md` vs current
      Debian Policy package version.
- [ ] `debhelper-compat` default vs current debhelper.
- [ ] Salsa-CI pinned ref in `salsa-ci.md` and the bootstrap
      template.
- [ ] Each `references/*.md` `Last reviewed` date < 6 months.
- [ ] Any new accepted DEPs.

## Style for SKILL.md prompts

- **Address the LLM.** "Run X, then Y, then ask the maintainer."
- **Bullet rules, not paragraphs.** Easier for the model to
  attend to.
- **Hard rules in a dedicated section** titled "Hard rules". Use
  the same phrasing as `shared-context.md` so the prompts
  reinforce.
- **Examples for fiddly formats.** E.g. show what a good
  `lintian-overrides` comment looks like; the model will copy
  the shape.
- **Avoid hedging in instructions.** "MUST" / "MUST NOT" / "MAY"
  / "SHOULD" with their RFC-2119 meanings.

## Testing changes locally

```
# From the source tree of a real package:
DEBUTANT_CONTEXT=$PWD/.debutant/context.json \
  claude --bare --print "$(< prompt-fixture.md)"
```

The workshop's `claude-exec` action wires the same thing — see
`workshop.yaml`.
