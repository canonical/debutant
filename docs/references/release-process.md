# Release process — what workers must not do

Releasing a Debian package is a human action with social and
technical consequences. Workers participate in *preparation* only.

## UNRELEASED → release

- `debian/changelog` distribution `UNRELEASED` means the entry is
  in progress.
- The maintainer runs `dch -r` (or hand-edits) to set the
  distribution and finalise the timestamp.
- Workers MUST leave `UNRELEASED` in place. The orchestrator
  reminds the maintainer at the end of a run.

## Target distributions

- **`unstable`** — Debian development; default target.
- **`experimental`** — for transitions and library bumps that
  need staging.
- **`<release>-backports`** (e.g. `bookworm-backports`) —
  backports to stable.
- **`<release>-security`** — coordinated with the security team
  only.
- **`<release>` (e.g. `bookworm`)** — stable updates via the
  release team's process.
- **`noble`, `oracular`** — Ubuntu primary archive.
- **`noble-proposed`** — Ubuntu staging.

Workers neither pick nor change these. If the maintainer asks the
worker to do so, refuse and explain.

## NMUs (Non-Maintainer Uploads)

If you (the maintainer running the skill) are not the
`Maintainer:` of a package, your upload is an NMU. Conventions:

- Version suffix: `+nmuN` for native, or `-N.1` for non-native
  (where `-N` was the previous Debian revision).
- Diff sent to the BTS first (`bts attach`, MR, or
  delayed-NMU queue).
- DELAYED queue: `dput delayed-7-day` etc.

Workers DO NOT perform NMUs. They produce the changes; a human
chooses to ship as NMU and follows the etiquette.

## RC bugs

- Severity `serious`, `grave`, `critical` block the next stable
  release.
- `bts severity 123456 serious`.
- An RC-bug fix is a legitimate reason to upload urgently; even
  then, workers prepare, humans upload.

## Freezes

Twice per release cycle, the archive enters increasingly strict
freezes (`testing` migration slows, then halts). During freeze,
non-trivial uploads need release-team approval.

Workers should warn the maintainer if the current date is
during a known freeze window. (The list is volatile; check
https://release.debian.org/.)

## See also

- https://www.debian.org/doc/developers-reference/ — devref,
  chapter on uploading.
- https://release.debian.org/ — current freeze status.
- https://www.debian.org/Bugs/server-control — `bts` commands.
