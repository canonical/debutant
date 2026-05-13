# hello-rust (stub)

Planned: a minimal Cargo workspace (one library crate + one binary
crate) to exercise bootstrap against the Cargo build system.

Workers will produce `debian/` with `dh-sequence-cargo` (or
`dh-cargo`), `Architecture: any`, separate `lib*-dev` and binary
packages.

TODO: populate with real upstream source + `expected/debian/`.
