# VCS workflows for Debian packaging

## DEP-14 layout

Authoritative branch names:

- `debian/unstable` — current packaging work targeting unstable.
  Default branch for new packages.
- `debian/<release>` (e.g. `debian/bookworm`) — release-specific
  branches for stable updates / backports.
- `upstream/latest` — imports of upstream releases.
- `pristine-tar` — pristine-tar binary deltas to reconstruct
  upstream tarballs.

Tag pattern: `debian/<version>` (e.g. `debian/1.2.3-1`).

## gbp buildpackage

`gbp` is the canonical UI for DEP-14-style workflows.

- `gbp import-orig --uscan` — fetch upstream via `debian/watch`
  and merge into `upstream/latest`, optionally regenerate
  `pristine-tar` branch.
- `gbp import-orig <tarball>` — same, for a local tarball.
- `gbp buildpackage` — build from current branch via configured
  builder.
- `gbp dch` — generate `debian/changelog` entries from git log.

Config in `debian/gbp.conf`:

```ini
[DEFAULT]
debian-branch = debian/unstable
upstream-branch = upstream/latest
pristine-tar = True
sign-tags = True
```

## pristine-tar

Stores the exact bytes of upstream tarballs as a delta against
the imported upstream tree, so anyone can regenerate the original
tarball from the git repo.

- `pristine-tar commit <tarball> upstream/<version>`
- `pristine-tar checkout <tarball>`
- Branch `pristine-tar` holds the deltas.

Workers don't manage pristine-tar directly; the maintainer's
`gbp import-orig --pristine-tar` does.

## dgit

`dgit` exposes the entire Debian archive as a git repository,
including binary uploads.

- `dgit clone <package>` — clone the archive view of a package.
- `dgit push-source` — upload from git, recording the git history
  in the source package.
- Compatible with DEP-14 but adds its own constraints.

Workers do not invoke `dgit push-source` (it's an upload command).

## Signed tags

`git tag -s debian/1.2.3-1 -m "Release 1.2.3-1"`.

The maintainer's key must be configured; `gbp buildpackage
--git-tag --git-sign-tags` automates this.

Workers do not create signed tags — release is a human action.

## See also

- https://wiki.debian.org/PackagingWithGit
- https://salsa.debian.org/help — Salsa user guide.
