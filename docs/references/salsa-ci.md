# Salsa-CI

The shared GitLab CI pipeline used by most Debian packaging
repos on https://salsa.debian.org/.

## What it does

Default jobs (subset):

| Job | What it checks |
|---|---|
| `extract-source` | `dpkg-source -x` on the source package |
| `build` | Builds the package in a clean chroot |
| `build i386` / `arm64` / … | Multi-arch builds |
| `reprotest` | Reproducible-builds check |
| `lintian` | `lintian -EvIL +pedantic` on the build output |
| `autopkgtest` | DEP-8 tests in LXC |
| `blhc` | Build-log hardening check |
| `piuparts` | Install/upgrade/remove cycle |
| `missing-breaks` | Looks for missing `Breaks:` |

## Minimum config

```yaml
---
include:
  - https://salsa.debian.org/salsa-ci-team/pipeline/raw/master/salsa-ci.yml
  - https://salsa.debian.org/salsa-ci-team/pipeline/raw/master/pipeline-jobs.yml
```

## House-style: pin the template ref

`master` of `salsa-ci-team/pipeline` evolves. Pin to a
known-good ref once the package has a working pipeline:

```yaml
include:
  - https://salsa.debian.org/salsa-ci-team/pipeline/raw/<TAG-OR-COMMIT>/salsa-ci.yml
  - https://salsa.debian.org/salsa-ci-team/pipeline/raw/<TAG-OR-COMMIT>/pipeline-jobs.yml
```

Workers MAY suggest this as a refresh action, citing this
document; never pin silently.

## Common failures

- **lintian job red, local lintian green.** Often a flag/version
  drift. Check `lintian` version in the CI image vs local.
- **reprotest red.** Frequently caused by timestamps, sort order,
  or build-path leaks. Fix the underlying nondeterminism; do not
  paper over.
- **blhc red.** A package is overriding hardening flags
  somewhere. Check `debian/rules` overrides and the upstream
  build system.
- **autopkgtest skipped.** No `debian/tests/` present — run the
  autopkgtest worker.

## See also

- https://salsa.debian.org/salsa-ci-team/pipeline
- https://salsa.debian.org/salsa-ci-team/pipeline/-/blob/master/README.md
