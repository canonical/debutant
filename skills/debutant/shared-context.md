# Shared context contract

This file documents the runtime contract that all `debutant-*`
workers obey. When workers are invoked through the orchestrator,
the orchestrator builds the context once and passes it in. When a
worker is invoked directly, the worker builds the context itself
using the same scripts.

## How context is passed

The orchestrator (or a directly-invoked worker) writes a single
JSON document to `${DEBUTANT_CONTEXT}` if that env var is set,
else to `./.debutant/context.json` relative to the source tree.

Workers MUST:

1. Check `${DEBUTANT_CONTEXT}` first.
2. Fall back to `./.debutant/context.json`.
3. If neither exists, run `skills/debutant/scripts/detect-source.sh`
   and `skills/debutant/scripts/tooling-probe.sh` themselves, then
   merge their outputs.

## JSON schema (v1)

```json
{
  "schema_version": 1,
  "generated_at": "ISO 8601 UTC timestamp",
  "source": {
    "path": "absolute path to source tree",
    "language": "go|rust|python|c|cpp|java|nodejs|perl|ruby|haskell|unknown",
    "build_system": "autotools|meson|cmake|cargo|go-mod|setuptools|pyproject|nodejs|make|unknown",
    "has_debian_dir": true,
    "has_quilt_patches": false,
    "debian_branch_layout": "monorepo|dep14|separate-branch|none|unknown",
    "upstream_vcs": "git|hg|svn|tarball|none|unknown"
  },
  "tooling": {
    "sbuild":         {"available": true,  "version": "x.y"},
    "pbuilder":       {"available": false, "version": null},
    "autopkgtest":    {"available": true,  "version": "x.y"},
    "lintian":        {"available": true,  "version": "x.y"},
    "debputy":        {"available": false, "version": null},
    "wrap-and-sort":  {"available": true,  "version": "x.y"},
    "gbp":            {"available": true,  "version": "x.y"},
    "dh_make":        {"available": true,  "version": "x.y"},
    "cme":            {"available": false, "version": null}
  },
  "target": {
    "distro":  "debian|ubuntu",
    "release": "unstable|trixie|noble|...|unknown",
    "host_arch": "amd64|arm64|..."
  },
  "user": {
    "debfullname": "value of DEBFULLNAME or git config user.name",
    "debemail":    "value of DEBEMAIL or git config user.email"
  },
  "budget": {
    "max_attempts_per_error_class": 3,
    "diff_threshold_lines": 200,
    "repeat_budget": 2
  },
  "reference_corpus": "absolute path to reference debian/ trees, or null",
  "house_style": "absolute path to house-style.md (orchestrator passes the active one)"
}
```

## Iteration-budget envelope

All workers that mutate the source tree MUST honour:

- **`max_attempts_per_error_class`** — at most N attempts to resolve
  any single class of error (lintian tag, sbuild stage failure,
  autopkgtest test name). After N attempts targeting the same error
  class, bail to the maintainer with a structured summary.
- **`repeat_budget`** — if the same exact error reappears after a
  fix attempt N=`repeat_budget` times in a row, bail immediately
  regardless of `max_attempts_per_error_class`.
- **`diff_threshold_lines`** — before producing a diff larger than
  this, the worker MUST surface a summary to the maintainer and ask
  for confirmation to proceed. Refresh worker is especially subject
  to this.

## Reference-corpus contract

- If `reference_corpus` is set, workers MAY consult `${corpus}/<language>/`
  for exemplar `debian/` trees. Consultation is read-only.
- The default corpus is `tests/fixtures/` of this repo.
- Override with `--reference=<path>` on the orchestrator or worker;
  disable with `--reference=none` (sets `reference_corpus: null`).
- Workers MUST NOT copy corpus files verbatim — corpus exemplars
  are reference points for idiom, not templates. Generated files
  go through house-style rendering.

## Bail-out summary format

When a worker bails to the maintainer, it MUST produce a message
with these sections:

```
## What I was trying to do
<one paragraph>

## What I tried
- attempt 1: <change> → <result>
- attempt 2: <change> → <result>
- attempt 3: <change> → <result>

## Current state
- build: <pass|fail with stage>
- lintian: <N E, M W, K I>
- diff size: <N lines>

## Where I'm stuck
<concrete error, no hedging>

## Proposed options
1. <option> — <consequence>
2. <option> — <consequence>
3. Stop and let me investigate manually

Which would you like?
```

## What workers MUST NOT do

- Invoke `dput`, `debrelease`, `dgit push`, or any upload command.
- Run `git push` to any remote.
- Edit `debian/changelog` distribution from `UNRELEASED` to anything else.
- Edit `Maintainer:` or `Uploaders:` fields.
- Edit upstream source files (use `debian/patches/` with DEP-3 headers).
- Run `rm -rf` or `git clean -fdx` on the workspace.
- Write a lintian override without a `# reason` comment.
- Set `Multi-Arch: same` without verifying file paths.
