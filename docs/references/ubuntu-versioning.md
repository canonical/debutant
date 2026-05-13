# Ubuntu version-string conventions

**Last reviewed**: 2026-05-13

Applies when `target.distro == ubuntu`. The canonical reference
is the Ubuntu project docs at
`how-ubuntu-is-made/concepts/version-strings.md`.

## Shape

```
[upstream]-[debian_revision][suffix][N]
```

Suffix vocabulary (and only this vocabulary — workers MUST NOT
invent new suffixes):

| Suffix    | Meaning                                              |
|-----------|------------------------------------------------------|
| `ubuntu`  | Ubuntu carries a delta from Debian                   |
| `build`   | No-change rebuild (e.g. library transition)          |
| `willsync`| Placeholder — will be overwritten by next Debian sync |

`ubuntu` > `willsync` > `build` for ordering purposes
(`dpkg --compare-versions`).

## Common moves

| Previous            | Action                  | Next                  |
|---------------------|-------------------------|-----------------------|
| `2.0-2`             | Ubuntu change           | `2.0-2ubuntu1`        |
| `2.0-2ubuntu1`      | Ubuntu change           | `2.0-2ubuntu2`        |
| `2.0-3`             | Re-sync with Debian     | `2.0-3` (unchanged)   |
| `2.0-3`             | No-change rebuild       | `2.0-3build1`         |
| `2.0-3build1`       | Ubuntu change           | `2.0-3ubuntu1`        |
| `2.0-1`             | Reserved for next sync  | `2.0-1willsync1`      |

For native-in-Ubuntu packages and SRU-targeted versions, see the
upstream version-strings doc — those cases need maintainer
judgement.

## What workers do

- **Read** the current top changelog version; classify into
  `ubuntu_delta = true | false | null` (see
  `shared-context.md`).
- **Never** invent the next version. The maintainer runs `dch -i`
  (or git-ubuntu's equivalent) themselves.
- **Never** flip `ubuntuN` ↔ `buildN` ↔ `willsync`. Those carry
  semantic meaning a worker can't infer.

## See also

- `docs/references/ubuntu-merges-syncs.md` — when to sync vs merge.
- `docs/references/sru.md` — SRU versioning is different.
- Ubuntu docs: `how-ubuntu-is-made/concepts/version-strings.md`.
