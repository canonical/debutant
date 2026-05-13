# Build tools — sbuild vs pbuilder vs gbp buildpackage vs dpkg-buildpackage

## TL;DR

| You want… | Use |
|---|---|
| Archive-quality builds in a clean chroot | **sbuild** |
| Build from a git tree honouring DEP-14 layout | **gbp buildpackage** (wraps the above) |
| A quick local rebuild from already-extracted source | **dpkg-buildpackage** |
| The "old" clean-room builder (still works) | **pbuilder** |

Workers default to `sbuild` for the verification loop.

## sbuild

The reference clean-room builder used by Debian buildds.

- Uses schroot (or `unshare`) for isolation.
- `mk-sbuild` to create a chroot per release/architecture.
- Honours `Build-Depends` strictly — catches missing deps that
  build locally because you happen to have the package installed.
- `sbuild --no-arch-all` skips arch-all binaries when iterating.
- Pairs with `lintian` automatically: `lintian` runs at the end
  by default; configure in `~/.sbuildrc`.

When to use: every time you intend to upload, and every time a
worker enters the verification loop.

## gbp buildpackage

Wraps `sbuild` (or `pbuilder`, or `dpkg-buildpackage`) with
git-buildpackage's DEP-14 awareness.

- Builds from `debian/latest` (or whatever `debian-branch =` says
  in `debian/gbp.conf`).
- Imports orig tarball from `pristine-tar` if configured.
- Signs git tags after a successful release build.
- `--git-builder='sbuild ...'` to wire to sbuild.

When to use: from a git checkout following DEP-14. Workers prefer
this when `source.debian_branch_layout == "dep14"`.

## dpkg-buildpackage

The low-level build command. No isolation.

- `dpkg-buildpackage -us -uc -b` for an unsigned binary build.
- Builds in the source tree itself; pollutes the working
  directory.
- Fast iteration, but trusts your installed `Build-Depends`.

When to use: never for verification; only for very fast iteration
during authoring. Workers fall back to this only when sbuild is
unavailable.

## pbuilder

The original clean-room builder. Still maintained.

- Tarball-based chroots (slower setup than sbuild's schroot).
- `pbuilder --create`, `pbuilder --build`.
- Cookbook works well for cross-architecture builds via
  `qemubuilder`.

When to use: legacy environments, cross-arch testing,
maintainer-preference. Otherwise prefer sbuild.

## See also

- https://wiki.debian.org/sbuild
- https://wiki.debian.org/PackagingWithGit
- `man dpkg-buildpackage`
