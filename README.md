# StormByte-GitHub

[![Release](https://img.shields.io/github/v/release/StormBytePP/StormByte-GitHub?sort=semver&display_name=tag&label=release)](https://github.com/StormBytePP/StormByte-GitHub/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Bash](https://img.shields.io/badge/bash-5%2B-4EAA25?logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![GitHub CLI](https://img.shields.io/badge/gh-required-2088FF?logo=github&logoColor=white)](https://cli.github.com/)
[![helpers bash](https://img.shields.io/badge/functions.sh-%E2%89%A5%201.3.0-555)](https://github.com/StormBytePP/StormByte.Repository)
[![helpers git](https://img.shields.io/badge/git.sh-%E2%89%A5%201.3.1-555)](https://github.com/StormBytePP/StormByte.Repository)
[![Changelog](https://img.shields.io/badge/changelog-Keep%20a%20Changelog-E05735)](CHANGELOG.md)
[![Pages](https://img.shields.io/github/actions/workflow-status/StormBytePP/StormByte-GitHub/jekyll-gh-pages.yml?label=docs)](https://github.com/StormBytePP/StormByte-GitHub/actions/workflows/jekyll-gh-pages.yml)

Unified Bash CLI for day-to-day work across many local GitHub clones under one
root directory: Actions caches, CI runs, git operations, pin-aware submodules,
source dumps, releases, forks, search/clone helpers, and sanitizer test drivers.

**Version 1.1.0** · License: MIT

---

## What it does

| Area | Capabilities |
|------|----------------|
| **Cache** | `show` / `list` / `delete` GitHub Actions caches; optional `--name` glob |
| **CI** | `stop` / `start` / `restart` / `status` of workflow runs; `--failed` |
| **Git** | `reset` (hard + clean + submodules), `pull` / `sync` (rebase **without** re-pinning first-level modules) |
| **Submodules** | `init` / `deinit` / `update` / `list` / `details` / `add` / `delete`. Update follows A / B1 / B2; `--latest` may cross major; `--module` / `--tag` filter or pin one gitlink; `details` reports PIN vs newest release |
| **Repos** | inventory, source dump, status, push, release (create / list / delete) |
| **Fork** | `status` / `sync` / `branch-status` under optional `FORK_ROOT` |
| **Search / clone** | code search and `clone-all` for the configured owner |
| **Test** | `asan` / `tsan` via CMake + ctest or a binary |
| **Dir-spec** | parent clones under the root, a single path, or exclusions (`!name`, `!a+!b`). Never a submodule name (`--module` does that) |

Per-run environment overrides (`DRY_RUN`, `ROOT`, `OUT`, `OWNER`, `FORK_ROOT`, …)
apply without editing the config file.

---

## Requirements

- **bash** (modern: `set -euo pipefail`, associative arrays)
- **[GitHub CLI](https://cli.github.com/)** (`gh`), authenticated for API work
- **jq**
- **git** with submodule support
- **StormByte helpers**
  - `StormByte-functions-bash` ≥ **1.3.0** (`functions.sh`)
  - `StormByte-functions-git` ≥ **1.3.1** (`git.sh`)  
  Default locations: next to the script, or `/lib/StormByte/functions.sh` and
  `/lib/StormByte/git.sh`

Optional: `rg` (ripgrep) for `search` (falls back to `grep`).

---

## Installation

### From this repository

```bash
git clone https://github.com/StormBytePP/StormByte-GitHub.git
cd StormByte-GitHub

sudo install -m 755 StormByte-GitHub /usr/local/bin/StormByte-GitHub

# Optional: bash completion
sudo install -m 644 StormByte-GitHub.bash-completion \
  /usr/share/bash-completion/completions/StormByte-GitHub

# Optional: man page
sudo install -m 644 StormByte-GitHub.1 /usr/local/share/man/man1/
```

Install the StormByte function libraries so the script can `source` them.

### Gentoo

StormByte overlay package `dev-util/StormByte-GitHub`. The ebuild installs the
binary, completion, man page, and helper dependencies.

---

## Configuration

On first run the tool writes `~/.StormByte-GitHub.conf` (interactive prompts):

| Key | Meaning |
|-----|---------|
| **OWNER** | GitHub username that owns the repositories |
| **ROOT** | Working root (immediate subdirs = local clones for most commands) |
| **OUT** | Output directory for `repos dump` |
| **FORK_ROOT** | Optional root used **only** by `fork` (if unset, `fork` uses `ROOT`) |

Override for a single run:

```bash
ROOT=~/Software OWNER=StormBytePP StormByte-GitHub repos list
DRY_RUN=1 StormByte-GitHub cache delete
FORK_ROOT=~/Forks StormByte-GitHub fork status
```

Other variables:

| Variable | Meaning |
|----------|---------|
| `DRY_RUN=1` | Simulate without applying (checkouts, deletes, commits, pushes, clones…) |
| `PAGE_LIMIT=N` | Items per API page (default: 100) |
| `MAX_PAGES=N` | Max `cache delete` passes; `0` = until empty |
| `RUN_LIMIT=N` | Max workflows to re-run; `0` = all |
| `ONLY_FAILED=1` | Legacy alias for `--failed` |

---

## Directory selection (dir-spec)

Used by most multi-repo commands. Always a **local clone path**, never a
GitHub repo name and never a submodule name.

| Spec | Meaning |
|------|---------|
| *(omitted)* | Every **immediate** git repository under the effective root |
| `<dir>` | That directory only (absolute, relative, `~/…`, or basename under the root) |
| `!<name>` | All under the root **except** that basename |
| `!a+!b` | Exclude several basenames |

The effective root is `ROOT`, except `fork`, which uses `FORK_ROOT` (or `ROOT`
if unset).

Quote exclusions in interactive shells so history expansion does not consume `!`:

```bash
StormByte-GitHub repos push '!multimedia'
StormByte-GitHub git sync '!multimedia+!legacy'
```

---

## Commands

```text
StormByte-GitHub <command> [subcommand] [arguments…]
```

No arguments, or `help`, prints full usage.

---

### Cache

```bash
StormByte-GitHub cache show
StormByte-GitHub cache list
StormByte-GitHub cache delete
StormByte-GitHub cache delete --name '*ccache*'
StormByte-GitHub cache delete '!multimedia'
```

`show` prints id / size / key. `list` prints counts only. `delete` pages the
API until empty (or `MAX_PAGES`). With `--name`, it stops when a page has no
matches.

---

### CI

```bash
StormByte-GitHub ci status
StormByte-GitHub ci stop
StormByte-GitHub ci start --failed
StormByte-GitHub ci restart multimedia
```

`stop` cancels active runs (`in_progress` / `queued` / `pending` / `waiting` /
`requested`) on the current branch. `start` re-runs finished workflows
(deduplicated by workflow name). `restart` = stop + pause + start.

---

### Git

```bash
StormByte-GitHub git reset
StormByte-GitHub git pull
StormByte-GitHub git sync
ROOT=~/Software StormByte-GitHub git sync '!multimedia'
```

| Subcommand | Behaviour |
|------------|-----------|
| `reset` | `git reset --hard` + `clean -fd` on the superproject, the same inside submodules (`foreach --recursive`), then `submodule update --init --recursive` |
| `pull` | `fetch origin` + `pull --rebase --no-recurse-submodules`, then nested modules sync to SHAs **already recorded** by the parent |
| `sync` | Alias for `pull`. Does **not** re-pin first-level modules (that is `submodules update`) |

---

### Submodules

```bash
StormByte-GitHub submodules list
StormByte-GitHub submodules details
StormByte-GitHub submodules details network
StormByte-GitHub submodules details --module 'StormByte BuildMaster'
StormByte-GitHub submodules details network --module 'StormByte BuildMaster' --all
StormByte-GitHub submodules init
StormByte-GitHub submodules deinit
StormByte-GitHub submodules update
StormByte-GitHub submodules update multimedia --confirm
StormByte-GitHub submodules update network --latest
StormByte-GitHub submodules update --module 'StormByte BuildMaster'
StormByte-GitHub submodules update --module 'StormByte BuildMaster' --tag 2.0.0
DRY_RUN=1 StormByte-GitHub submodules update --module 'StormByte BuildMaster'
StormByte-GitHub submodules add <dir> <URL> <path> --name <name>
StormByte-GitHub submodules delete <dir> --name <name>
StormByte-GitHub submodules delete <dir> --url <url>
StormByte-GitHub submodules delete <dir> --path <path>
```

`add` does not init/update afterwards. `delete` requires **exactly one**
selector, removes the folder, and does not commit or push.

With no dir-spec, `update` walks `ROOT` in **dependency order** (topological
sort over `.gitmodules` URLs that resolve to other clones under the same
`ROOT`).

**dir-spec** = which parent clones. **`--module`** = which first-level gitlinks
inside each parent (exact `.gitmodules` name or path).  
`--module '!a+!b'` = every first-level module except those.

---

#### `submodules details`

Not an alias of `list`. First-level only. Read-only (shadow-fetch tags; no
checkout, no commit). `DRY_RUN` does not apply.

| Column | Meaning |
|--------|---------|
| **NAME** | `.gitmodules` name |
| **URL** | clone URL |
| **PIN** | current tag, `branch=`, or `sha:……` |
| **LATEST** | newest recognised release (`git_pin_latest_recognised`) |

A `(semver)` suffix is added only when the ref text is **not** already that
semver (`1.0.1` stays `1.0.1`; `v2.0.0` becomes `v2.0.0 (2.0.0)`).
Unclassified pins print the raw ref with no parentheses.

`--all` prints, under each row, tags that are ancestors of `origin/master` or
`origin/main` (wrapped) and origin / shadow branches. The current pin is
marked with `*` in those lists.

`--module` filters rows. Without it, every first-level module is shown.

---

#### `submodules update` policy (1.1.0)

Only **first-level** modules are re-pinned. Nested modules stay on the SHAs
the parent has just staged.

The decision uses `branch=` in the parent's `.gitmodules` **and** what is
checked out in the module:

| Case | Condition | Action |
|------|-----------|--------|
| **A** | `branch=master` | Float to `origin/master`. Already at that SHA → *No changes* |
| **B1** | no master; checkout is a recognised release tag | Jump to the **latest** tag in the same family. Same tag, different SHA → re-release |
| **B2** | no master; checkout is **not** a tag | Pin the latest tag of the inferred family. A downgrade is intentional. If no family, the newest recognised release |
| **UNRESOLVED** | pin not recognised | On a TTY you can type a ref; `--confirm` / non-TTY skips that row |

`--latest` keeps A / B1 / B2, but PLANNED is the newest recognised release
**in that module**, including a new major. Default stays in-family.

`--tag <tag>` requires a **positive** `--module` (not `!…`). That row pins the
exact tag (character match) and **ignores** family policy and `--latest`.

Rows not selected by `--module` still appear in the plan as *No changes*.

Families (prefix + grain), extended in `git.sh`:

- own libraries: `X.Y` / `X.Y.Z`
- PostgreSQL: `REL_16_4` → 16.4, family by major (`REL_16_*`)
- Crypto++: `CRYPTOPP_8_9_0` → 8.9.0, family by `X.Y` while the prefix matches

Before applying:

1. If the parent is dirty, stash it (restored at the end; a conflicted pop
   leaves the stash in place).
2. Print the `MODULE / CURRENT / PLANNED` table.
3. On a TTY, without `--confirm`, and if there is work, ask `[y/N]`.
   `--confirm` or non-TTY applies the plan (UNRESOLVED rows skipped).
4. Checkout (`origin/<branch>` or tag), stage the gitlink with
   `update-index --cacheinfo 160000`, **then** sync nested modules to those SHAs.
5. Local commit `chore: update dependencies`. **No push.**

`DRY_RUN=1` prints the plan and does not touch the tree.

---

### Repos

```bash
StormByte-GitHub repos list
StormByte-GitHub repos dump ./base type=cmake 200
StormByte-GitHub repos status
StormByte-GitHub repos push
StormByte-GitHub repos push '!multimedia'
```

`list` is not recursive: one row per immediate folder under `$ROOT`.
`dump` writes into `$OUT` (`type=c++` by default, or `type=cmake`); a numeric
argument splits the dump into chunks of that many KiB.

`status` reports `OK`, `DIRTY`, `UNPUSHED`, or `DIRTY+UNPUSHED`.
`push` is a plain `git push` (no `--force`); skips repos with no upstream.

---

### Releases

```bash
StormByte-GitHub repos release --list
StormByte-GitHub repos release database --list
StormByte-GitHub repos release --list 1.0.0
StormByte-GitHub repos release database 1.0.0 --list
StormByte-GitHub repos release database 1.0.0
StormByte-GitHub repos release database 1.0.0 --skip-ci
StormByte-GitHub repos release database 1.0.0 --delete
DRY_RUN=1 StormByte-GitHub repos release database 1.0.0
```

| Mode | Behaviour |
|------|-----------|
| `--list` | GitHub Releases only (not bare tags). Optional dir-spec and/or version. Tags without a Release are reported as a note |
| `--delete <dir> <version>` | Remove Release + tag (local and `origin`) after typing `yes`. No-op if neither exists. `--skip-ci` is ignored |
| Create (default) `<dir> <version>` | Clean tree on `master`/`main`, OWNER remote, non-empty `## [<version>] - YYYY-MM-DD` section in `CHANGELOG.md`. Notes come from that section. Waits for CI unless `--skip-ci` (Ctrl-C during the wait: no tag yet). Same version deletes and recreates. Pre-releases (`1.0.0-rc.1`) use `gh --prerelease`. Confirmation must be exactly `yes` |

---

### Fork

```bash
FORK_ROOT=~/Forks StormByte-GitHub fork status
FORK_ROOT=~/Forks StormByte-GitHub fork sync
FORK_ROOT=~/Forks StormByte-GitHub fork branch-status
```

| Subcommand | Behaviour |
|------------|-----------|
| `status` | behind/ahead vs upstream; non-forks are reported and skipped |
| `sync` | update **only** `master` when behind upstream; refuses if the remote is not a fork |
| `branch-status` | local branches ≠ `master`: `CLEAN` / `CONFLICTS` merging into `master` (`merge-tree`). Notes local-only or origin-divergent tips |

---

### Search and clone

```bash
StormByte-GitHub search "mimalloc"
StormByte-GitHub search "mimalloc" '!thirdparty'
StormByte-GitHub clone-all
```

`clone-all` clones every **public** `$OWNER` repository missing under `$ROOT`.
Existing folders are left alone.

---

### Test

Execution priority per repository:

1. `--executable` → run that binary (relative to the build dir or absolute)
2. `--test REGEX` → `ctest -R REGEX`
3. *(none)* → `ctest` (all tests)

```bash
StormByte-GitHub test asan
StormByte-GitHub test asan multimedia
StormByte-GitHub test tsan '!multimedia'
StormByte-GitHub test asan database --cmake-options "-DENABLE_FOO=ON" --verbose
StormByte-GitHub test asan multimedia --executable bin/my_test --outfile /tmp/report.txt
```

Options: `--builddir`, `--testdir`, `--outfile`, `--cmake-options`,
`--executable`, `--test`, `--asan-options`, `--tsan-options`, `--verbose`.

The build directory is created by the tool and **always** removed on exit
(success or failure). If it already exists, that repo is skipped for safety.
The report (`--outfile`, default `/tmp/<repo>_asan|tsan_report.txt`) is written
only when the run fails. Sanitizer flags are injected first; `--cmake-options`
come after so they can override. Default extra: `-DENABLE_TEST=ON`.

---

## Output colours

Section headers stay blue/cyan. Everything else uses a short palette:

| Colour | Use |
|--------|-----|
| Dark green | idle / *No changes* / OK with no work |
| Light green (bold) | planned or applied update; PIN ≠ LATEST in `details` |
| Yellow | warnings (`DIRTY`, `UNPUSHED`, unresolved prompts, …) |
| Red | failures, `DIRTY+UNPUSHED`, `CONFLICTS`, error rows |

---

## Bash completion

After installing the completion file, Tab completes commands, subcommands,
dir-specs (root basenames, `./…`, absolute paths), flags (`--confirm`,
`--latest`, `--all`, `--module`, `--tag`, …), and release-version hints read
from `CHANGELOG.md`.

---

## Notes

- Repositories must belong to the configured **OWNER** (or the `OWNER=` override).
- Omitting the dir-spec walks only **immediate** subdirectories of the effective
  root (not a recursive `find`).
- `dir-spec` selects parent clones; `--module` selects first-level gitlinks.
- `--tag` wins over `--latest` and over A/B1/B2 on that row.
- `submodules update` floats to `master` **only** when the parent declares
  `branch=master`. Removing that line is the signal to pin a tag (case B2).
- `git sync` / `git pull` do not change first-level pin policy.
- Use `DRY_RUN=1` before a bulk `cache delete`, `repos release --delete`, or
  a wide `git reset`.

---

## License

MIT License. See [LICENSE](LICENSE) for the full text.
