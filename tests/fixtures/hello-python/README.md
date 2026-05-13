# hello-python (stub)

Planned: a minimal Python project (`pyproject.toml` with
`hatchling` or `setuptools` backend, one importable module) to
exercise bootstrap against PEP-517.

Workers will produce `debian/` with `dh-sequence-python3` and
`pybuild-plugin-pyproject`, `Architecture: all`, single binary
package.

TODO: populate with real upstream source + `expected/debian/`.
