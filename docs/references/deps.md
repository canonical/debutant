# Debian Enhancement Proposals (DEPs)

A DEP is a formally-tracked proposal to change Debian-wide practice.
The ones below are the load-bearing DEPs for packaging work.

## DEP-3 — Patch tagging guidelines

Header fields for `debian/patches/*.patch` so reviewers know:
- Where the patch came from (`Origin:`, `Forwarded:`).
- Why it exists (`Description:`, `Bug:`, `Bug-Debian:`).
- When it should retire (`Forwarded: <url>` then deleted on next
  upstream release, or `Applied-Upstream: <commit>` after merge).
- Who wrote it (`Author:`).
- When it was last refreshed (`Last-Update:`).

Workers MUST emit DEP-3 headers on every patch they create.

https://dep-team.pages.debian.net/deps/dep3/

## DEP-5 — Machine-readable debian/copyright

Structured `Files: … / Copyright: … / License: …` stanzas.

- Required by Policy for new packages (de facto).
- `licensecheck` is a starting point, not a substitute for
  reading actual file headers.
- For vendored deps, list each upstream license stanza
  separately; do not merge.

Workers MUST emit DEP-5; if ambiguous, ASK rather than guess.

https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/

## DEP-8 — Autopkgtest

As-installed test interface, consumed by `autopkgtest`,
`britney`, and the QA infrastructure.

- `debian/tests/control` defines test cases.
- Tests run against the installed `.deb`, not the build tree.
- `Restrictions:` declares environmental needs
  (`needs-root`, `isolation-container`, `needs-internet`, ...).

The autopkgtest worker emits DEP-8 tests with minimum
restrictions.

https://salsa.debian.org/ci-team/autopkgtest/raw/master/doc/README.package-tests.rst

## DEP-12 — Upstream metadata

`debian/upstream/metadata` — YAML with stable fields:
`Contact`, `Name`, `Bug-Database`, `Bug-Submit`, `Repository`,
`Repository-Browse`, etc.

Workers DO NOT auto-populate this; the maintainer verifies the
URLs. Workers may propose a draft.

https://wiki.debian.org/UpstreamMetadata

## DEP-14 — Recommended git branch layout

- `debian/unstable` (or `debian/<distribution>` for release branches).
- `upstream/latest` for upstream imports.
- `pristine-tar` for tarball reconstruction.
- Tags: `debian/<version>` (signed when possible).

Workers detect existing layout from branch names; never force a
maintainer onto a new layout, but flag drift on refresh.

https://dep-team.pages.debian.net/deps/dep14/

## See also

- https://dep-team.pages.debian.net/ — all DEPs, including
  withdrawn/draft ones not listed above.
