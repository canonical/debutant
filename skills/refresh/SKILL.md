---
name: refresh
description: Modernise an existing debian/ directory to current house style — compat bump, Standards-Version, R³, dh-sequence migration, wrap-and-sort, watch v4, DEP-5 normalisation, M-A audit, Salsa-CI. Default dry-run. The most dangerous worker; treat the maintainer's prior choices with respect.
---

# debutant:refresh

Modernise an existing `debian/` directory to the active house style.

This is the **most dangerous worker** in the suite. The maintainer
made deliberate choices when they wrote the existing packaging;
your job is to *propose* updates, with citations, and let the
maintainer accept or reject each one. **Default mode is dry-run.**

## Preconditions

- A context JSON exists at `./.debutant/context.json`. If missing,
  build it: run `${CLAUDE_PLUGIN_ROOT}/scripts/detect-source.sh`
  and `${CLAUDE_PLUGIN_ROOT}/scripts/tooling-probe.sh`, merge their
  outputs (see `${CLAUDE_PLUGIN_ROOT}/shared-context.md` for the
  full schema).
- `source.has_debian_dir == true`. If `false`, refuse and suggest
  `/debutant:bootstrap`.

## Hard rules

Suite-wide (apply to every debutant skill):

- Never invoke `dput`, `debrelease`, `dgit push`, or any upload
  command.
- Never `git push` to any remote.
- Never edit `debian/changelog` distribution from `UNRELEASED` to
  anything else.
- Never edit `Maintainer:` or `Uploaders:` fields.
- Never edit upstream source files directly (use `debian/patches/`
  with DEP-3 headers).
- Never run `rm -rf` or `git clean -fdx` on the workspace.
- Never write a lintian override without a `# reason` comment.
- Never set `Multi-Arch: same` without verifying file paths.

Phase-specific (refresh):

- **Default to dry-run.** Produce a single diff against the
  existing tree; do not write until the maintainer says go.
- **Never change `Maintainer:` or `Uploaders:`.**
- **Never bump `debian/changelog`.** Refresh is not a release. If
  changes are accepted, the maintainer adds a `dch -i` entry
  themselves.
- **Never refactor `override_dh_*` targets without showing the
  maintainer first.** These exist for reasons the maintainer
  knows and you don't.
- **Never auto-set `Multi-Arch: same`.** Audit, propose, justify;
  let the maintainer decide.
- **Each modernisation is a separate, labelled hunk** in the diff
  with a one-line citation (Policy section / devref / DEP-NN /
  house-style line).

## Scope — opt-in per item via flags

The maintainer enables specific refreshes via flags. Without any
flag, run the full audit and report findings WITHOUT producing
changes. Flags:

- `--compat[=N]` — bump debhelper-compat to N (default: house-style value).
- `--standards-version[=X.Y.Z]` — bump Standards-Version (default: house-style value).
- `--rrr` — review `Rules-Requires-Root` for d/control file and remove
  it if it's set to `no` as it's the default one in the latest version
  of the policy.
- `--dh-sequence` — convert `dh $@ --with foo` to
  `dh-sequence-foo` Build-Depends.
- `--wrap-and-sort` — run `wrap-and-sort -ast`.
- `--watch-v5` — upgrade `debian/watch` to v5 syntax.
- `--dep5` — normalise `debian/copyright` to DEP-5.
- `--m-a-audit` — produce an advisory report on `Multi-Arch:`
  candidates; never writes.
- `--salsa-ci` — add or update `debian/salsa-ci.yml`.
- `--all` — enable everything above except `--m-a-audit` (which
  is always advisory).
- `--yes` — skip the confirmation gate before writing.

## Process

1. **Audit phase** (always runs). Read the existing `debian/`
   tree and produce a structured report:
   ```
   compat:           current=12, target=13         [eligible]
   standards:        current=4.6.0, target=4.7.1   [eligible]
   rrr:              present but not needed        [eligible]
   dh-sequence:      uses --with python3,golang    [eligible]
   wrap-and-sort:    unsorted Build-Depends        [eligible]
   watch:            v3                            [eligible]
   dep5:             freeform                      [eligible]
   m-a:              no annotations                [advisory]
   salsa-ci:         absent                        [eligible]
   override_dh_*:    3 targets (auto_configure,    [needs review]
                     auto_test, install)
   ```
2. **Plan phase.** For each enabled flag, produce the planned
   change as a hunk with a citation. For `override_dh_*` targets,
   list them and ask the maintainer how to proceed BEFORE
   modifying.
3. **Diff phase.** Emit a single unified diff covering all planned
   changes. Group hunks by file. Annotate each hunk with the
   citation as a comment in the diff header (using `# `).
4. **Confirmation gate.** Check `budget.diff_threshold_lines`. If
   exceeded, summarise and ask before writing. Otherwise, ask
   anyway when not `--yes`.
5. **Apply phase** (only if maintainer approves). Write the diff
   to the tree. Re-run `wrap-and-sort -ast` if it was enabled. Run
   `${CLAUDE_PLUGIN_ROOT}/scripts/verify.sh` to confirm the result
   still builds and lintian is no worse than before. Honour the
   iteration-budget envelope.
6. **Report.** Files changed, build/lint status, any items
   deferred to the maintainer.

## Bail-out conditions

- A planned change conflicts with an `override_dh_*` target that
  the maintainer hasn't approved touching.
- Result builds but introduces NEW lintian errors.
- Diff exceeds threshold and maintainer declines confirmation.
- DEP-5 normalisation produces ambiguity (any `Files:` stanza you
  can't classify with confidence).

Use the bail-out summary format from
`${CLAUDE_PLUGIN_ROOT}/shared-context.md` § "Bail-out summary
format".
