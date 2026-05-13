---
name: debutant-bootstrap
description: Create a fresh debian/ directory for an unpackaged upstream source tree. Use when source.has_debian_dir is false. Output is builds-once, lintian-respectable, UNRELEASED packaging ready for maintainer review. Debian-first, Ubuntu overlay.
---

# debutant-bootstrap

Create a complete `debian/` directory from scratch for an upstream
source tree that has none.

## Preconditions

- A context JSON exists (built by the orchestrator or by you on
  direct invocation — see `../debutant/shared-context.md`).
- `source.has_debian_dir == false`. If `true`, refuse and suggest
  `debutant-refresh` instead.
- A maintainer identity is available via `user.debfullname` and
  `user.debemail`. If either is missing, ask the maintainer before
  proceeding.

## What you produce

A minimal but complete `debian/` directory with:

- `debian/control` — house-style fields, accurate `Source:`,
  `Section:`, `Priority:`, `Build-Depends:` derived from the
  build system, one or more binary stanzas. Long `Description:`
  is left as `<INSERT LONG DESCRIPTION HERE>` for the maintainer
  to fill in — NEVER invent prose marketing copy.
- `debian/changelog` — created with `dch --create
  --package=<name> --newversion=<upstream-version>-1
  "Initial packaging."` then immediately set distribution to
  `UNRELEASED`.
- `debian/copyright` — DEP-5 with one `Files: *` stanza for the
  upstream license (verified via `licensecheck -r .` AND a manual
  spot-check of file headers), plus one `Files: debian/*` stanza
  for the packaging.
- `debian/rules` — minimal `dh $@` form. Add `dh-sequence-*`
  Build-Depends instead of `--with`.
- `debian/source/format` — `3.0 (quilt)` (or `3.0 (native)` only
  if `source.upstream_vcs == none` AND version has no `-` revision).
- `debian/watch` — version 4. `pgpmode=auto` only if
  `debian/upstream/signing-key.asc` exists; otherwise `pgpmode=none`.
  For git-only upstreams, use `mode=git`.
- `debian/salsa-ci.yml` — include the pinned template version from
  `docs/references/salsa-ci.md`.
- `debian/gbp.conf` — DEP-14 layout (`debian-branch = debian/latest`,
  `upstream-branch = upstream/latest`, `pristine-tar = True` if the
  package will use it).

Templates live in `./templates/`. Render each through the
house-style rules; do not copy verbatim.

## What you do NOT produce

- A finished long-form `Description:`. Leave the placeholder.
- A populated `debian/$pkg.install` unless the build system makes
  the install layout unambiguous (e.g. a single binary going to
  `/usr/bin/`). Otherwise, defer to the maintainer.
- A populated `debian/tests/` — that's `debutant-autopkgtest`.
- `debian/upstream/metadata` — propose it as a follow-up; do not
  bootstrap it (DEP-12 fields need maintainer verification).
- `debian/upstream/signing-key.asc` — you cannot verify a key;
  the maintainer must place this.
- Lintian overrides at bootstrap time. If a tag fires, fix it,
  don't suppress.

## Process

1. **Load context.** Read `./.debutant/context.json` or build it
   yourself (see shared-context.md).
2. **Sanity-check inputs.** Confirm with maintainer: package name,
   upstream version, section (suggest one, ask to confirm),
   priority (default `optional`).
3. **License discovery.** Run `licensecheck -r .` and read the
   top-level `LICENSE`/`COPYING`/`README` files. For ambiguous
   results, ASK rather than guess.
4. **Build-deps discovery.** Read the build-system config
   (`Cargo.toml`, `go.mod`, `pyproject.toml`, `configure.ac`,
   `CMakeLists.txt`, etc.) and translate to Debian package names
   when known. For unknown mappings, list them in a follow-up
   block for the maintainer.
5. **Generate `debian/`** from templates + computed values.
6. **`wrap-and-sort -ast`** on the result.
7. **First build attempt** using `sbuild` (if available) or
   `dpkg-buildpackage -us -uc -b`. Enter the verification loop
   (see `../debutant/shared-context.md` § "Iteration-budget
   envelope").
8. **Report.** Print:
   - Files created.
   - First sbuild result.
   - First lintian result (`lintian -EvIL +pedantic`).
   - Any unresolved questions the maintainer must answer
     (Description, ambiguous license, unknown build-dep mappings).

## Hard rules

Inherited from `../debutant/shared-context.md` § "What workers
MUST NOT do". Special emphasis for bootstrap:

- **Do NOT ship `dh_make` output.** You may invoke `dh_make
  --native=no --copyright=<spdx>` IN A TEMPORARY DIRECTORY to
  compare its choices against yours, but the generated files go
  to `/tmp`, not to `debian/`.
- **Do NOT invent a `Maintainer:` for the package.** Use
  `user.debfullname <user.debemail>` from context. If those are
  missing, ask.
- **Do NOT enable `pgpmode=auto`** without an existing signing
  key file.

## Bail-out conditions

- Cannot determine the upstream license unambiguously.
- Cannot map a build dependency to a Debian package.
- Build system is unsupported (no `dh-sequence-*` for it).
- First sbuild fails AND you cannot identify a recovery path
  within budget.

Bail-out summary format: see shared-context.md § "Bail-out
summary format".
