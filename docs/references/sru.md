# Stable Release Updates (SRU)

**Last reviewed**: 2026-05-13

SRUs are package updates to a **released** Ubuntu series. They
follow a stricter process than dev-series uploads. Canonical
reference: Ubuntu docs `SRU/stable-release-updates.rst` and the
SRU site at https://documentation.ubuntu.com/sru/.

This kicks in when `target.distro == ubuntu` AND `target.pocket
!= dev` (i.e. `proposed`/`updates`/`security`/`backports`).

## What qualifies

The SRU team only accepts updates that:

- Fix a real-world bug, security issue, or hardware enablement
  regression.
- Have a clear regression potential paragraph in the changelog.
- Ship with a verifiable test plan in the LP bug.
- Don't introduce new features unless explicitly justified
  (e.g. microrelease exception, hardware enablement).

## Workflow shape

1. Land the fix in the **current dev release** first (no regression
   to a release that already has it).
2. Open or update a Launchpad bug with the SRU template:
   `Impact`, `Test Plan`, `Where problems could occur`,
   `Original description`.
3. Upload to `<series>-proposed` with version
   `<base>~<series>N` or `<base>+<series>N` (see versioning).
4. Wait for `proposed-migration` and verification by the bug
   reporter (`verification-needed-<series>` ⇒ `verification-done-<series>`).
5. SRU team copies to `-updates` after the aging period.

## Versioning

SRU version targeting a released series uses a tilde-suffix that
sorts **less than** the next release version. Common shapes
(consult Ubuntu version-strings doc for the authoritative list):

- `1.2.3-4ubuntu1.1` — first SRU on top of `1.2.3-4ubuntu1`.
- `1.2.3-4ubuntu0.24.04.1` — backport with explicit series anchor.

Workers MUST NOT invent SRU versions; this is maintainer
territory.

## What workers do (or don't)

- Refresh/lintian/autopkgtest workers MAY run, but they MUST
  treat the package as conservatively as possible: minimum
  changes, no opportunistic modernisation.
- The orchestrator MUST surface a confirmation gate noting that
  this is an SRU and require explicit `--yes` to proceed past
  any non-trivial refresh.
- Workers NEVER set the changelog distribution to
  `<series>-proposed`. The maintainer does that with `dch` (or
  the upload script).
- Workers NEVER write the regression-potential paragraph. The
  maintainer writes it.

## Phased updates

`Phased-Update-Percentage:` in `Release` files controls
gradual rollout of SRU packages. Refresh workers MUST NOT touch
it. See `how-ubuntu-is-made/concepts/phased-updates.md`.

## See also

- Ubuntu docs: `SRU/stable-release-updates.rst`.
- `maintainers/SRU/review-an-sru.md`.
- `docs/references/ubuntu-versioning.md`.
