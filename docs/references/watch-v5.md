# debian/watch — version 5 syntax

**Last reviewed**: 2026-05-13

Watch v5 (the RFC822-style format) replaced the v3/v4
`opts=key=value` syntax. Workers emit v5 by default per
`docs/house-style.md`. The canonical reference is `man uscan`
shipped with `devscripts`; verify against the version installed
on the build host when in doubt.

## Field shape

```
Version: 5
[Field]: [value]
[Field]: [value]
...
```

Fields use `Hyphen-Case` names, one per line, no `opts=` wrapper.
Multi-source watch files separate stanzas with a blank line.

## Common fields

| Field | Purpose |
|---|---|
| `URL`              | Where to look for new versions. |
| `Match-Pattern`    | Regex extracting the version from a filename or HTML page. Use `(\d[\d.]+)` to capture. |
| `Match-Tag-Pattern`| Regex on git tag names (for `Template: git`). |
| `Pgp-Mode`         | `auto`, `mangle`, or `none`. Requires `debian/upstream/signing-key.asc` when not `none`. |
| `Template`         | One of `github`, `gitlab`, `pypi`, `sourceforge`, `cpan`, `git`. Selects a built-in pattern. |
| `Repack-Suffix`    | If upstream tarball must be repacked, e.g. `+dfsg`. |
| `Mangle`           | Version-string post-processing rules. |
| `Filename-Mangle`  | Rewrite the downloaded filename to Debian convention. |

## Three idiomatic examples

### 1. GitHub release tags

```
Version: 5
Template: github
URL: https://github.com/owner/project
Match-Tag-Pattern: v(\d[\d.]+)
Pgp-Mode: none
```

`Template: github` derives the tag-listing URL from `URL`; the
worker doesn't have to construct the `/tags` path by hand.
`Pgp-Mode: none` because GitHub release tags are typically
unsigned; if the project signs them, switch to `auto` and place
the key.

### 2. Classic upstream tarball with signature

```
Version: 5
URL: https://download.example.org/project/
Match-Pattern: project-(\d[\d.]+)\.tar\.(?:gz|xz|bz2)
Pgp-Mode: auto
```

`Pgp-Mode: auto` requires `debian/upstream/signing-key.asc` to be
present — `uscan` fetches `.asc` alongside the tarball and
verifies. Without the key file, downgrade to `Pgp-Mode: none` and
flag for the maintainer.

### 3. Git-only upstream (no tarball releases)

```
Version: 5
Template: git
URL: https://git.example.org/project.git
Match-Tag-Pattern: v?(\d[\d.]+)
Pgp-Mode: none
```

`Template: git` produces a `pristine-tar`-compatible tarball from
the matched tag. Pair with `pristine-tar = True` in
`debian/gbp.conf` so future imports are reproducible.

## Migration from v4

| v4                                       | v5                            |
|------------------------------------------|-------------------------------|
| `version=4`                              | `Version: 5`                  |
| `opts=pgpmode=auto, ...`                 | `Pgp-Mode: auto` (one line each) |
| `opts="pgpsigurlmangle=s/$/.sig/"`       | `Pgp-Url-Mangle: s/$/.sig/`   |
| `opts=repacksuffix=+dfsg,dversionmangle=s/-rc/~rc/` | `Repack-Suffix: +dfsg` + `Mangle: s/-rc/~rc/` |
| Trailing `\` line-continuation           | One field per line; no continuation. |

## Bootstrap worker — render flag mapping

The bootstrap template uses two booleans and three string
placeholders (see `skills/bootstrap/SKILL.md` § "Template render
flags"):

- `has_signing_key` → gates `Pgp-Mode: auto` vs `Pgp-Mode: none`.
- `has_template`    → gates the `Template:` field path.
- `template_name`   → one of the values in the `Template` table.
- `template_specific_flags` → the extra fields the template
  needs (`Match-Tag-Pattern:` for `github`/`git`,
  `Filename-Mangle:` for `sourceforge`, …).
- `watch_source_fields` → the non-template fallback set
  (`URL:`, `Match-Pattern:`, etc.).

When the upstream release pattern doesn't fit any built-in
template **ask the maintainer** rather than emit a guess — a
broken watch file blocks uscan and salsa-CI's auto-update job.

## Tooling caveats

- `uscan` from `devscripts ≥ <pin-current-version>` is required
  to read v5; older versions silently ignore the file.
- Salsa-CI's pipeline images bundle their own `devscripts`; check
  `docs/references/salsa-ci.md` for the pinned ref and confirm
  the pinned image is recent enough.
- `debian/watch` is consumed by `qa.debian.org` and Debian's
  archive infrastructure as well as `uscan`; both should follow
  upstream `devscripts` quickly, but a brand-new field may take
  a release cycle to propagate.

## See also

- `man uscan` on the build host.
- `docs/house-style.md` § "debian/watch".
- `skills/bootstrap/templates/watch.tmpl`.
- https://wiki.debian.org/debian/watch (community examples).
