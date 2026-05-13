# debutant

Debian packaging automation skills for Claude Code.

`debutant` is a set of Claude Code skills that help a Debian /
Ubuntu package maintainer bootstrap, modernise, lint-clean, and
add autopkgtest coverage to a package — while keeping the
maintainer in the loop for every judgement call.

The skills are **prescriptive** (house-style + policy + tooling
indicators) and **respectful** (default dry-run on destructive
operations, no uploads, no version bumps without explicit
request, no Maintainer-field changes).

## Status

First version. Skills are scaffolded; fixtures are stubs. See
`/home/peb/.claude/plans/you-are-a-specialized-refactored-pascal.md`
for the plan this implements.

## Layout

```
debutant/
├── workshop.yaml             # Canonical workshop runner setup
├── skills/
│   ├── debutant/             # Orchestrator
│   ├── debutant-bootstrap/   # New package from scratch
│   ├── debutant-refresh/     # Modernise existing debian/
│   ├── debutant-lintian/     # Fix lintian tags + justified overrides
│   └── debutant-autopkgtest/ # Add or improve debian/tests/
├── docs/
│   ├── house-style.md        # Prescriptive packaging choices, cited
│   ├── developer.md          # How to extend debutant
│   └── references/           # Short notes: build tools, DEPs, vcs, …
└── tests/
    └── fixtures/             # E2E test inputs + reference corpus
```

## Invocation

From within a source tree:

```
# Full pipeline
/debutant

# Single phase
/debutant --only=lintian
/debutant-lintian

# Phase subset
/debutant --only=refresh,lintian --skip=autopkgtest

# Dry-run (refresh defaults to this anyway)
/debutant --dry-run

# Custom house style
/debutant --house-style=/path/to/team-style.md

# Disable reference corpus
/debutant --reference=none
```

## Design

See:

- `skills/debutant/SKILL.md` — orchestrator behaviour.
- `skills/debutant/shared-context.md` — the contract every worker
  obeys (JSON context schema, iteration budget, bail-out format,
  hard rules).
- `docs/house-style.md` — every prescriptive choice with a
  citation.

## What it will not do

- Upload anything (`dput`, `debrelease`, `dgit push`, `git push`).
- Edit `debian/changelog` distribution away from `UNRELEASED`.
- Edit `Maintainer:` or `Uploaders:`.
- Edit upstream sources in place (always a DEP-3 quilt patch).
- Suppress lintian tags without a `# reason:` comment.
- Set `Multi-Arch: same` without verifying file paths.

## Contributing

See `docs/developer.md` for how to add a new worker.

## License

TBD — pick before first release.
