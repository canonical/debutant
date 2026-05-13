---
name: bootstrap
description: Create a fresh debian/ directory for an unpackaged upstream source tree. Use when source.has_debian_dir is false. Output is builds-once, lintian-respectable, UNRELEASED packaging ready for maintainer review. Debian-first, Ubuntu overlay.
---

# debutant:bootstrap

Create a complete `debian/` directory from scratch for an upstream
source tree that has none.

## Preconditions

- A context JSON exists at `./.debutant/context.json`. If missing,
  build it: run `${CLAUDE_PLUGIN_ROOT}/scripts/detect-source.sh`
  and `${CLAUDE_PLUGIN_ROOT}/scripts/tooling-probe.sh`, merge their
  outputs (see `${CLAUDE_PLUGIN_ROOT}/shared-context.md` for the
  full schema).
- `source.has_debian_dir == false`. If `true`, refuse and suggest
  `/debutant:refresh` instead.
- A maintainer identity is available via `user.debfullname` and
  `user.debemail`. If either is missing, ask the maintainer before
  proceeding.

## What you produce

A minimal but complete `debian/` directory with:

- `debian/control` — house-style fields, accurate `Source:`,
  `Section:`, `Build-Depends:` derived from the build system, one
  or more binary stanzas. **Omit the `Priority:` field unless the
  package is `required`/`important`/`standard`** — Policy 4.7.3
  deprecated specifying the default (`optional`). `Standards-Version`
  is set from `${CLAUDE_PLUGIN_ROOT}/docs/house-style.md` (currently
  4.7.4). Long `Description:` is left as `<INSERT LONG DESCRIPTION HERE>`
  for the maintainer to fill in — NEVER invent prose marketing copy.
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
  `${CLAUDE_PLUGIN_ROOT}/docs/references/salsa-ci.md`.
- `debian/gbp.conf` — DEP-14 layout (`debian-branch = debian/latest`,
  `upstream-branch = upstream/latest`, `pristine-tar = True` if the
  package will use it).

Templates live in `${CLAUDE_PLUGIN_ROOT}/skills/bootstrap/templates/`.
Render each through the house-style rules; do not copy verbatim.

## What you do NOT produce

- A finished long-form `Description:`. Leave the placeholder.
- A populated `debian/$pkg.install` unless the build system makes
  the install layout unambiguous (e.g. a single binary going to
  `/usr/bin/`). Otherwise, defer to the maintainer.
- A populated `debian/tests/` — that's `/debutant:autopkgtest`.
- `debian/upstream/metadata` — propose it as a follow-up; do not
  bootstrap it (DEP-12 fields need maintainer verification).
- `debian/upstream/signing-key.asc` — you cannot verify a key;
  the maintainer must place this.
- Lintian overrides at bootstrap time. If a tag fires, fix it,
  don't suppress.

## Process

1. **Load context.**
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
7. **First verify.** Call `${CLAUDE_PLUGIN_ROOT}/scripts/verify.sh`
   to run sbuild (or fall back to dpkg-buildpackage) + lintian.
   Enter the iteration-budget loop (see
   `${CLAUDE_PLUGIN_ROOT}/shared-context.md` § "Iteration-budget
   envelope"). If neither builder is available, report and stop —
   the maintainer must set up a build environment before bootstrap
   verification can complete.
8. **Report.** Print:
   - Files created.
   - First build result.
   - First lintian result.
   - Any unresolved questions the maintainer must answer
     (Description, ambiguous license, unknown build-dep mappings).

## Hard rules

Suite-wide (apply to every debutant skill):

- Never invoke `dput`, `debrelease`, `dgit push`, or any upload
  command.
- Never `git push` to any remote.
- Never edit `debian/changelog` distribution from `UNRELEASED` to
  anything else.
- Never edit `Maintainer:` or `Uploaders:` fields.
- Never edit upstream source files directly (use `debian/patches/`
  with DEP-3 headers).
- Never run `rm -rf` or `git clean -fdx` on the workspace.
- Never write a lintian override without a `# reason` comment.
- Never set `Multi-Arch: same` without verifying file paths.

Phase-specific (bootstrap):

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
- First build fails AND you cannot identify a recovery path
  within budget.

Use the bail-out summary format from
`${CLAUDE_PLUGIN_ROOT}/shared-context.md` § "Bail-out summary
format".
