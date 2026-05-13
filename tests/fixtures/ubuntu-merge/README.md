# ubuntu-merge (stub)

Planned: a small upstream tree shipped both as a Debian package
and an Ubuntu package with a deliberate Ubuntu delta, exercising
the (future) merge worker and the Ubuntu overlay rules in
`docs/house-style.md`.

Shape to include:

- Two changelog snapshots:
  - **debian/**: `1.2.3-2` (Debian unstable)
  - **ubuntu/**: `1.2.3-1ubuntu1` (Ubuntu, one commit behind
    Debian, carries a single trivial delta)
- A non-trivial delta in `debian/rules` (e.g. an `override_dh_*`
  target Ubuntu added on top).
- `Maintainer: Ubuntu Developers …` with `XSBC-Original-Maintainer:`
  set, so `update-maintainer` round-trips cleanly.

The expected merge output should:

- Pick `1.2.3-2ubuntu1` as the next version
  (`docs/references/ubuntu-versioning.md`).
- Re-apply the Ubuntu rules-file delta on top of Debian's `1.2.3-2`.
- Preserve `Maintainer:` / `XSBC-Original-Maintainer:`.
- Leave the changelog distribution `UNRELEASED`.
- Surface (not write) the regression-potential paragraph as a
  TODO for the maintainer.

TODO: populate with real Debian + Ubuntu source snapshots and an
`expected/merge.diff`.
