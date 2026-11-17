# RTL819x patch store (quilt)

Out-of-tree patch series for the Realtek RTL8197F (RTL819x) platforms. The
patch files here are the source of truth, a git branch is not. The series
order is the numeric sort of the store-root file names (`NNN[-SS]-` with
zero-padded fields); there are no bucket directories and the `series` file
quilt works from is generated, not tracked. Kernel trees are disposable and
are extracted from the baseline commit named in `baseline`.

This file states only what is specific to this store - layout, class order
and numbering. quilt itself is documented by quilt.
checkpatch is a pre-submission spot-check tool, not a store gate. Cutting a
submission out of the series, writing its introduction and sending it are a
separate upstreaming process.

## Layout

```
series              the order patches apply in (generated from the filename sort, not tracked)
baseline            kernel version + commit every patch applies to
quilt.conf          --fuzz=0, -p ab refresh, and QUILT_PATCHES pointing here
*.patch             every patch, flat in this root
```

`quilt.conf` defaults `QUILT_PATCHES` to the directory it lives in, so a tree
only has to name the file:

```sh
quilt --quiltrc=<this directory>/quilt.conf push -a
```

`new` and `refresh` then write into this directory, `series` included - there
is no staging copy and nothing to sync back. `.pc/` stays in the tree, so an
arm tree can work against one store. Patches apply with `--fuzz=0`. Beware:
quilt caches the patches directory in `.pc/.quilt_patches` of the tree it
last ran in, and trusts that cache over `QUILT_PATCHES` - point a tree at a
different store by rewriting that file, not just by exporting the variable.

Get a tree by extracting `BASE_SHA` from `baseline`
(`git -C <kernel repository> archive <BASE_SHA> | tar -x -C <tree>`) and take
the kernel version and configuration from `baseline` too.

Renumbering or splitting patches is a file rename plus a `series` edit;
`quilt new` and `quilt delete` edit `series` in place.

Do not run `quilt init` in this directory: init would add a `.pc/` here, and
`.pc/` belongs in the tree.

## Numbering

`NNN[-SS]-<shortname>.patch`; the shortname is exactly what `git format-patch`
derives from the patch's `Subject:` line (original case kept, `_` and `.`
kept, truncated at 52 characters) - rename a patch by rewriting its subject,
never by hand-editing the file name.

| range | content | numbering |
|---|---|---|
| 0xx | patches taken from another kernel version (upstream backports, none yet) | one number each, no series grouping; named `0NN-vX.Y-<shortname>` for the kernel version they came from |
| 1xx-8xx | submittable patches | `NNN-SS`: **NNN is one submission** - the series that goes to one maintainer - and SS is the position of a patch inside it. A submission uses `NNN-SS` throughout, never bare `NNN` alongside `NNN-SS` |
| 9xx | local-only content (bring-up instrumentation, bench harness), never submitted | one number each, no series grouping |

Class order is dependency order (a provider always precedes its consumers):

| range | class |
|---|---|
| 1xx | shared ABI prerequisites: dt-bindings consumed by 2+ submissions, vendor prefixes, txt-to-YAML conversions (none yet) |
| 2xx | platform/arch and fixes to existing drivers: mach support, SoC dtsi, board dts creation, driver quirks |
| 3xx-7xx | new driver introductions (a driver's own binding stays in the same `NNN`) |
| 8xx | board integration tails: dts hunks that need a driver submission first |
| 9xx | hack (local-only, none yet) |

The hundreds digit of 3xx-7xx is a **size band**, banding by added lines across
the submission:

| band | limit |
|---|---|
| 3xx | 250 |
| 4xx | 800 |
| 5xx | 2,000 |
| 6xx | 5,000 |
| 7xx | 20,000 |

**Inserting a new group**: pick any free number between the neighbours - a
non-multiple of 5 in the middle is fine - and leave every other number
untouched. Renumbering back onto clean 5-multiple spacing happens only when
explicitly asked.
