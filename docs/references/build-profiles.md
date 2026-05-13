# Build profiles

**Last reviewed**: 2026-05-13

Debian Policy 4.7.4 formally documents build profiles. A profile
is a named tag that can switch parts of the build on or off,
typically to break bootstrap cycles or skip slow optional steps.

## Syntax

Annotate dependencies in `debian/control` with `<profile-name>`:

```
Build-Depends:
 debhelper-compat (= 13),
 libfoo-dev,
 python3-pytest <!nocheck>,
 doxygen <!nodoc>,
 libgcc-dev <stage1>,
```

`<!nocheck>` means "drop this dep when the `nocheck` profile is
active". `<stage1>` means "include this dep only when `stage1` is
active". Profiles compose with `,` (AND) and ` ` (OR) inside
angle brackets.

## Common profiles

| Profile     | What it disables                                |
|-------------|-------------------------------------------------|
| `nocheck`   | Build-time test suites                          |
| `nodoc`     | Documentation generation                        |
| `cross`     | Native-only build steps when cross-compiling    |
| `stage1`    | First bootstrap stage of a circular dep chain   |
| `stage2`    | Second bootstrap stage                          |
| `pkg.<pkg>.<name>` | Package-private profile (rare)           |

`DEB_BUILD_PROFILES=nocheck dpkg-buildpackage` activates one.

## What workers do

- **Refresh** MAY propose adding `<!nocheck>` / `<!nodoc>` when
  the audit shows long test runs or large doc deps, but only as
  a confirmation-gated suggestion. Cite Policy.
- **Bootstrap** does NOT add profiles. Profiles are added later,
  in response to a concrete need (a long test suite, a bootstrap
  cycle). Adding them preemptively obscures the build graph.
- **Lintian** MUST NOT add a profile annotation as a fix for any
  tag. The right answer there is a quilt patch or an override.

## See also

- Policy 4.7.4 — build-profiles section.
- https://wiki.debian.org/BuildProfileSpec
