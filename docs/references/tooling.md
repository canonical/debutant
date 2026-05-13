# Packaging tooling cheatsheet

Short notes on tools workers use; not a replacement for `man`.

## debputy

Newer declarative packaging helper, complementary to debhelper.

- `debputy lint` — fast static checks on `debian/` (yaml
  validation, manifest checks).
- `debputy check-manifest` — validates `debian/debputy.manifest`.
- Not a replacement for debhelper yet; workers use it as a
  *cheap check* in the tiered-verification path.

## decopy

Automatic `debian/copyright` generator.

- `decopy` — scans the source tree for licenses and copyright
  notices, then writes (or updates) `debian/copyright` in DEP-5
  format.
- Re-parses an existing `debian/copyright` to produce a more
  complete output.
- Useful when bootstrapping or after a large upstream refresh
  that changes licence headers.

## lrc

`lrc` (from the `licenserecon` package) cross-checks `debian/copyright` against
`licensecheck` output.

- `lrc` — runs `licensecheck` on the source tree and compares
  each file's detected licence with the DEP-5
  `debian/copyright`; reports mismatches.
- Catches drift between actual file licences and the declared
  copyright file.
- Workers SHOULD run `lrc` after any refresh that touches
  `debian/copyright`.

## wrap-and-sort

Canonicalises whitespace and sorting in `debian/control`,
`debian/copyright`, `debian/*.install`, etc.

- `wrap-and-sort -ast` (the house default).
- `-a` wrap, `-s` short paragraph (puts each dep on its own line),
  `-t` trailing comma.
- Add `-b` for blank-line groups if the maintainer prefers.

Run after every refresh that touches control fields.

## cme

`cme` (Config Model Engine) validates and modifies Debian
control files programmatically.

- `cme check dpkg` — sanity-check `debian/control` etc.
- `cme update dpkg` — auto-fix some issues (with confirmation).
- Workers MAY use `cme check`; `cme update` requires maintainer
  approval (it can be opinionated).

## devscripts highlights

- `uscan` — fetch upstream releases via `debian/watch`.
- `licensecheck -r .` — bulk license discovery (use as hint, not
  oracle).
- `dch` — edit `debian/changelog`. `--create`, `-i`, `-r`.
- `dget` — fetch a Debian source package.
- `debuild` — wraps `dpkg-buildpackage` + lintian.
- `bts` — interact with the BTS.
- `rmadison` — query archive presence of a package.

## blhc

Build-Log Hardening Check. Parses a build log for missing
hardening flags.

- `blhc /path/to/build.log` — flags PIE, fortify, bindnow, etc.
- House-style rule: clean output (relies on dpkg-buildflags
  defaults).

## hardening-check

Inspects a binary for hardening features.

- `hardening-check /usr/bin/foo` — reports PIE, fortify,
  bindnow, RELRO, stack protector.

## sbuild auxiliaries

- `mk-sbuild --arch=amd64 unstable` — create a chroot.
- `sbuild-update -udcar u` — update all chroots.
- `sbuild --chroot=unstable-amd64-sbuild` — explicit chroot.

## See also

- `apt-cache search ^devscripts` — many small utilities here.
- https://wiki.debian.org/PackagingTutorial — the gentle intro.
