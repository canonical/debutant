---
name: debutant-lintian
description: Resolve lintian -EvIL +pedantic output by classifying each tag and producing the right fix — packaging change, DEP-3 quilt patch, or justified override with a comment. Bails to the maintainer after 3 attempts on the same tag. Never blanket-suppresses.
---

# debutant-lintian

Drive a package toward `lintian -EvIL +pedantic` clean, or
justified-overrides-only.

## Preconditions

- A context JSON exists.
- `source.has_debian_dir == true`.
- `tooling.lintian.available == true`. If not, ask the maintainer
  to install lintian.

## Tag classification

For every tag lintian emits, classify it into exactly one bucket:

| Bucket | Use when | How to fix |
|---|---|---|
| **fix in packaging** | Tag is fixable by a change in `debian/` | Smallest patch that resolves the tag without side effects. |
| **fix upstream via patch** | Tag points to an upstream source issue | DEP-3 quilt patch under `debian/patches/`, add to `series`, with `Forwarded:` header (URL or `no` + reason). |
| **justified override** | Tag is a false positive, or the right answer is to suppress for this package | `lintian-overrides` file with a `# reason:` line directly above the override. |
| **won't fix** | Tag is real but maintainer accepts the cost | Report in summary; do NOT silently override. |

Default: prefer **fix in packaging** > **fix upstream** > **override**
> **won't fix**. Use override only when fix isn't possible or has
worse trade-offs.

## Hard rules

In addition to the suite-wide rules in
`../debutant/shared-context.md`:

- **Every override needs a `# reason:` comment immediately above
  it.** No exceptions. If you cannot write a clear reason, the
  tag is not eligible for an override — it's "won't fix" until
  the maintainer decides.
- **Never override a tag class wholesale.** No `*` wildcards in
  override files unless the maintainer explicitly approves.
- **Place overrides correctly.** Source-level tags go to
  `debian/source/lintian-overrides`. Binary-level tags go to
  `debian/$pkg.lintian-overrides`. Don't mix.
- **Never edit upstream source files directly to fix a tag.**
  Always a quilt patch.
- **Never bump the package version.** A lintian-fix run is not a
  release. The maintainer will `dch -i` if they want to record
  the change.

## Process

1. **Load context.**
2. **Run lintian.** `lintian -EvIL +pedantic --info` on the
   built `.changes` (preferred) or `--source` on the `.dsc`. If
   no build artefacts exist, perform a quick `dpkg-buildpackage
   -us -uc -b -nc` first; abort if the build itself fails (that's
   not your problem — bail to the maintainer).
3. **Classify each tag.** Print the classification table to the
   maintainer.
4. **Resolve, tag by tag.** Process in order of severity (E → W → I
   → P). For each:
   - Apply the chosen fix.
   - Re-run lintian on the affected file/scope.
   - If the tag persists, count an attempt.
   - After `budget.max_attempts_per_error_class` attempts on the
     same tag, bail out for that tag with a structured summary
     and move on to the next.
5. **Final pass.** Re-run lintian on the full package. Confirm
   only justified overrides remain. Report.

## Override file format

```
# reason: <one-line justification, ideally citing policy/devref/DEP>
<package-or-source>: <tag> [<extra context>]
```

Example:
```
# reason: upstream ships a pre-built README.html for offline use;
# regenerating it would require a Sphinx Build-Depends just to
# delete the file, which is worse than the tag.
mypkg: privacy-breach-generic [usr/share/doc/mypkg/README.html]
```

## Bail-out conditions

- Same tag persists after `max_attempts_per_error_class` fix
  attempts.
- Tag classification is ambiguous (could be packaging fix OR
  upstream patch and the right answer needs domain knowledge).
- Fix introduces a NEW lintian error.
- Build fails as a precondition for running lintian.

Use the bail-out summary format from
`../debutant/shared-context.md`. Include:
- The unresolved tag and lintian's `--info` text for it.
- Each fix attempt and why it didn't work.
- Concrete proposed options for the maintainer.
