# debutant — house style

**Version**: 1 · **Last reviewed**: 2026-05-13 · **Next review due**: 2026-08-13

This file is the source of truth for *prescriptive* packaging choices
that go beyond what Debian Policy mandates. Workers consult it on
every run via `${CLAUDE_PLUGIN_ROOT}/docs/house-style.md`; a
maintainer can override the active style by passing
`--house-style=<path>` to the orchestrator. Every rule below carries one of:

- **Policy** §x.y — Debian Policy Manual reference
- **devref** §x.y — Debian Developer's Reference reference
- **DEP-NN** — a DEP standard
- **DD-judgement** — opinion of the maintainer of this repo;
  callers may override with their own house style via
  `--house-style=<path>`.

> Workers MUST cite the rule when they apply it (e.g. in a diff
> hunk comment or bail-out summary). Maintainers will not accept
> opaque "because the LLM said so" changes.

---

## Source format

- `3.0 (quilt)` for non-native packages. **Policy §5.6.13.**
- `3.0 (native)` only for genuinely native packages (no upstream
  tarball, version has no Debian revision). **Policy §5.6.12.**
- Never `1.0`. **DD-judgement** — deprecated for new work.

## debhelper

- `debhelper-compat (= 13)` as Build-Depends (virtual package).
  **DD-judgement** — 13 is stable and ubiquitous as of 2026-05;
  14 only when the maintainer opts in via flag and the package has
  no setuid binaries that depend on `Rules-Requires-Root` interactions.
- Use `dh-sequence-<name>` virtual Build-Depends to load add-ons,
  not `--with` in `debian/rules`. **DD-judgement** — declarative,
  surfaces dependencies to apt, lintian-checkable.
- `debian/rules` minimal form:
  ```make
  #!/usr/bin/make -f
  %:
  	dh $@
  ```
  Override targets only when necessary; never preemptively.
  **DD-judgement.**

## Control fields

- `Standards-Version: 4.7.1`. **Policy** — pin a known-good value;
  bump only as a deliberate refresh, never auto-bumped per run.
- `Rules-Requires-Root: no` by default. **Policy §4.9.2.**
  Switch to `binary-targets` if (and only if) the package installs
  setuid/setgid files or specific ownership that fakeroot can't
  preserve. Workers MUST detect this and warn.
- `Priority: optional` unless the package belongs to `required`,
  `important`, or `standard` per Policy §2.5. **Policy §2.5.**
- `Section`: pick from
  https://packages.debian.org/unstable/ — workers MUST NOT invent
  a section.
- `Vcs-Git`/`Vcs-Browser`: present and correct, pointing at the
  Salsa repo by default (Debian-first). Ubuntu overlays point at
  Launchpad. **DD-judgement.**
- `Maintainer`/`Uploaders`: workers NEVER mutate these. **DD-judgement.**

## Binary packages

- `Multi-Arch:` annotations are advisory-only from workers. The
  worker may propose `same` for libraries with arch-specific paths,
  `foreign` for arch-independent helper binaries, but MUST verify
  no file collisions across architectures before recommending.
  **Policy §5.6.30.** Setting `M-A: same` blindly breaks
  coinstallation.
- `Pre-Depends:` only when essential (e.g. `${misc:Pre-Depends}`
  for multiarch). Workers MUST justify any other Pre-Depends.
  **Policy §7.2.**

## debian/copyright

- DEP-5 machine-readable format always. **DEP-5.**
- For vendored dependencies or large source trees, default to
  asking the maintainer rather than guessing. **DD-judgement** —
  bad DEP-5 is worse than no DEP-5.
- `licensecheck -r .` is a starting point, not an oracle. Workers
  MUST verify each `Files:` stanza against actual file headers.

## debian/changelog

- `dch --create` (new package) or `dch -i` (new entry) — never
  hand-edit the format. **DD-judgement.**
- Distribution `UNRELEASED` until the maintainer explicitly approves
  a release. Workers NEVER set `unstable`, `trixie-backports`,
  `noble`, etc. **DD-judgement** — release is a human action.
- Workers NEVER bump the version on a refresh run. Refresh ≠ release.

## debian/watch

- Version 4 syntax. **DD-judgement.**
- `pgpmode=auto` (or `mangle`) ONLY when
  `debian/upstream/signing-key.asc` exists. Otherwise `pgpmode=none`.
  Workers MUST check for the key before enabling signature
  verification.
- Prefer `git mode` for projects without tarball releases.
  **DD-judgement.**

## debian/source/format and options

- `3.0 (quilt)`: yes.
- `debian/source/options`: omit unless needed. Common justified
  exception: `extend-diff-ignore` for generated files the
  maintainer can't get upstream to add to `.gitignore`. **DD-judgement.**
- `debian/source/local-options`: never check in. **devref §6.7.4.**

## Patches

- All upstream modifications via `debian/patches/` with DEP-3
  headers (`Origin:`, `Forwarded:`, `Last-Update:`, `Author:`,
  `Description:`). **DEP-3.**
- `debian/patches/series` ordered, no blank lines, no comments
  (`#`-lines are tolerated but discouraged). **DD-judgement.**
- Workers NEVER edit upstream sources in-place to "fix" a build.
  Always a patch. **DD-judgement.**

## Salsa-CI

- `debian/salsa-ci.yml` present for new packages. **DD-judgement.**
- Pin the template version explicitly; do not `include:` a
  `master` ref. (See `docs/references/salsa-ci.md`.)

## Formatting

- `wrap-and-sort -ast` (or `-abst` if the maintainer prefers
  blank-line groups) applied to:
  `debian/control`, `debian/copyright`, `debian/*.install`,
  `debian/*.examples`, `debian/*.docs`.
  **DD-judgement** — diff-stable, review-friendly.

## VCS layout

- DEP-14: `debian/latest` for development, `pristine-tar` branch
  for tarballs when used, `upstream/latest` for upstream imports.
  **DEP-14.**
- Workers MUST detect existing layout and not assume `master` is
  the packaging branch.
- Tags signed (`gbp buildpackage --git-tag-only --git-sign-tags`)
  when the maintainer has a key configured. **DD-judgement.**

## Autopkgtest

- `debian/tests/control` minimum `Restrictions:` to make the test
  pass. **DEP-8.**
- `Restrictions: needs-root, isolation-container` only when
  genuinely required and with a written justification. **DD-judgement.**
- Library packages: ABI smoke (load + a trivial symbol). Daemons:
  start + ping + stop, isolation-container scoped. CLI tools: invoke
  with `--version` at minimum, then the most common subcommand.

## Hardening

- Rely on `dpkg-buildflags` defaults; do not override unless the
  package needs an opt-out (and justify it). **DD-judgement.**
- `blhc` clean on the build log. **DD-judgement.**

## Lintian

- Target: `lintian -EvIL +pedantic` clean OR overridden with
  justification.
- Overrides MUST have a `# reason` comment line directly above
  the override. **DD-judgement** — opaque overrides rot.
- Place overrides in `debian/source/lintian-overrides` for
  source-level tags, `debian/$pkg.lintian-overrides` for
  binary-level tags. **Policy/lintian convention.**

---

## Notes for future-me

- Re-check Standards-Version, debhelper-compat default, and
  Salsa-CI template version at every quarterly review.
- New DEPs accepted between reviews: add to this file before
  workers can rely on them.
- Items marked DD-judgement here may be overridden in a
  downstream house-style file passed via `--house-style=<path>`.
  Workers MUST then cite "house-style (custom)" rather than
  "DD-judgement".
