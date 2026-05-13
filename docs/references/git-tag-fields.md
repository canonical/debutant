# Git-Tag-Tagger and Git-Tag-Info

**Last reviewed**: 2026-05-13

Two new source-control fields added in Debian Policy 4.7.3
(December 2025), §5.6.32 & §5.6.33. They record the git tagger
identity and the annotation that produced the release tag.

## Shape

```
Git-Tag-Tagger: Some Maintainer <maint@example.org> 1700000000 +0000
Git-Tag-Info: <tag annotation text, possibly multi-line>
```

Both fields are optional; they appear in the `.dsc` and
`.changes` when the upload tooling supplies them. `gbp
buildpackage` and modern `dpkg-source` know how to populate them.

## Where they show up

- `.dsc` (Debian source control file).
- `.changes` (upload control file).
- NOT in `debian/control` — the maintainer does not hand-edit
  them.

## What workers do

- **Bootstrap** does not write these fields — the release
  tooling will, once the maintainer cuts a tag.
- **Refresh** MUST NOT strip them if found in `.dsc` /
  `.changes` artefacts during an audit.
- **Lintian** is the authoritative checker — workers MUST NOT
  override any tags about these fields without a citation back
  to this doc.

## See also

- Debian Policy §5.6.32, §5.6.33 (Policy 4.7.3).
- `/usr/share/doc/debian-policy/upgrading-checklist.txt.gz`.
