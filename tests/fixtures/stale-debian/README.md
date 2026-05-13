# stale-debian (stub)

Planned: a small upstream tree (probably the C autotools one) plus
a deliberately out-of-date `debian/` directory exercising the
refresh worker.

Stale features to include:
- `debhelper (>= 10)` instead of `debhelper-compat`.
- `Standards-Version: 4.3.0` (old).
- No `Rules-Requires-Root`.
- `dh $@ --with python3` instead of `dh-sequence-python3`.
- `debian/watch` v3.
- Unsorted `Build-Depends`.
- Freeform `debian/copyright` (not DEP-5).
- No `debian/salsa-ci.yml`.

The expected refresh diff should hit every house-style item
without touching `Maintainer:` or `debian/changelog` version.

TODO: populate with real stale source + `expected/refresh.diff`.
