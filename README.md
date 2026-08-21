# StormByte-GitHub

Unified Bash CLI for day-to-day work across many local GitHub clones under one
root directory: Actions caches, CI runs, git sync, pin-aware submodules, source
dumps, releases, forks, search/clone helpers, and test drivers.

**Version 1.0.0** · License: MIT

---

## Features

| Area | Capabilities |
|------|----------------|
| **Cache** | List and delete GitHub Actions caches (optional `--name` with wildcards) |
| **CI** | Stop, start, restart, and status of workflow runs (`--failed` supported) |
| **Git** | Status, sync, and related operations over one repo or the whole tree |
| **Submodules** | `init` / `deinit` / `update` / `list` / `add` — root modules follow `branch=` when set; nested modules stay on the SHAs recorded by the parent (pins preserved) |
| **Repos** | List, dump sources, push, status, release (create / list / delete, `--skip-ci`, …) |
| **Fork** | Status and sync under optional `FORK_ROOT` |
| **Search / clone** | Search and clone-all for the configured owner |
| **Test** | Drive ctest or a binary with sanitizer presets and extra CMake options |
| **Dir-spec** | All repos under root, a single path, or exclusions (`!name`, `!a+!b`) |

Per-run environment overrides (`DRY_RUN`, `ROOT`, `OUT`, `OWNER`, `FORK_ROOT`, …)
apply without editing the config file.

---

## Requirements

- **bash** (modern)
- **[GitHub CLI](https://cli.github.com/)** (`gh`), authenticated for API operations
- **jq**
- **StormByte helpers**
  - `StormByte-functions-bash` ≥ 1.1.0
  - `StormByte-functions-git` ≥ 1.1.0  
  Default locations: `/lib/StormByte/functions.sh` and `/lib/StormByte/git.sh`  
  (or the same files next to the script)

Optional: network access for `gh` and remotes; git with submodule support.

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

Install the StormByte function libraries so the script can source them (see Requirements).

### Gentoo

If you use the StormByte overlay package `dev-util/StormByte-GitHub`, emerge it
as usual; the ebuild installs the binary, completion, man page, and dependencies.

---

## Configuration

On first run the tool creates `~/.StormByte-GitHub.conf` (interactive prompts):

| Key | Meaning |
|-----|---------|
| **OWNER** | GitHub username that owns the repositories |
| **ROOT** | Working root (immediate subdirs = local clones for most commands) |
| **OUT** | Output directory for source dumps |
| **FORK_ROOT** | Optional root used only by `fork` (if unset, fork uses `ROOT`) |

Override for a single run:

```bash
ROOT=~/Software OWNER=StormBytePP StormByte-GitHub repos list
DRY_RUN=1 StormByte-GitHub cache delete
FORK_ROOT=~/Forks StormByte-GitHub fork status
```

Other useful variables:

| Variable | Meaning |
|----------|---------|
| `DRY_RUN=1` | Simulate actions without applying them |
| `PAGE_LIMIT=N` | Items per API page (default: 100) |
| `MAX_PAGES=N` | Max cache-delete passes; `0` = until empty |
| `RUN_LIMIT=N` | Max workflows to re-run; `0` = all |
| `ONLY_FAILED=1` | Legacy alias for `--failed` |

---

## Directory selection (dir-spec)

Used by many commands:

| Spec | Meaning |
|------|---------|
| *(omitted)* | Every immediate git repository under the effective root |
| `<dir>` | That directory only (absolute, relative, `~/…`, or basename under root) |
| `!<name>` | All under root **except** that basename |
| `!a+!b` | Exclude several basenames |

Examples:

```bash
StormByte-GitHub repos status
StormByte-GitHub repos status multimedia
StormByte-GitHub git sync '!multimedia'
StormByte-GitHub test asan database --cmake-options "-DENABLE_FOO=ON"
```

Quote exclusions in interactive shells so history expansion does not consume `!`:

```bash
StormByte-GitHub repos push '!multimedia'
```

---

## Command overview

```text
StormByte-GitHub <command> [subcommand] [arguments…]
```

Run with no arguments, or `help`, for full usage. `help <command>` shows details
for one area when available.

### Cache

```bash
StormByte-GitHub cache list
StormByte-GitHub cache delete
StormByte-GitHub cache delete --name '*ccache*'
StormByte-GitHub cache delete '!multimedia'
```

### CI

```bash
StormByte-GitHub ci status
StormByte-GitHub ci stop
StormByte-GitHub ci start --failed
StormByte-GitHub ci restart multimedia
```

### Git

```bash
StormByte-GitHub git status
StormByte-GitHub git sync
ROOT=~/Software StormByte-GitHub git sync
```

### Submodules

```bash
StormByte-GitHub submodules list
StormByte-GitHub submodules update
StormByte-GitHub submodules init
StormByte-GitHub submodules deinit
StormByte-GitHub submodules add <dir> <URL> <path> --name <name>
```

**Update policy**

- **Root** submodule with `branch=` in `.gitmodules` → track that branch (e.g. `origin/master`).
- **Root** submodule **without** `branch=` → keep the pinned commit (gitlink SHA).
- **Nested** submodules → `update --init --recursive` so pins recorded by the parent are respected.

### Repos

```bash
StormByte-GitHub repos list
StormByte-GitHub repos dump ./base type=cmake 200
StormByte-GitHub repos status
StormByte-GitHub repos push
StormByte-GitHub repos push '!multimedia'
```

### Releases

```bash
StormByte-GitHub repos release --list
StormByte-GitHub repos release database --list
StormByte-GitHub repos release database 1.0.0
StormByte-GitHub repos release database 1.0.0 --skip-ci
StormByte-GitHub repos release database 1.0.0 --delete
DRY_RUN=1 StormByte-GitHub repos release database 1.0.0
```

### Fork

```bash
FORK_ROOT=~/Forks StormByte-GitHub fork status
FORK_ROOT=~/Forks StormByte-GitHub fork sync
```

### Search and clone

```bash
StormByte-GitHub search "mimalloc"
StormByte-GitHub clone-all
```

### Test

Execution priority per repository:

1. `--executable` → run that binary  
2. `--test` → `ctest -R REGEX`  
3. *(none)* → `ctest` (all tests)

```bash
StormByte-GitHub test asan
StormByte-GitHub test asan multimedia
StormByte-GitHub test tsan '!multimedia'
StormByte-GitHub test asan database --cmake-options "-DENABLE_FOO=ON" --verbose
StormByte-GitHub test asan multimedia --executable bin/my_test --outfile /tmp/report.txt
```

---

## Bash completion

After installing the completion file, Tab completes commands, subcommands,
dir-specs (root basenames, `./` paths, absolute paths), flags, and common
arguments (e.g. release version hints from `CHANGELOG.md`).

---

## Notes

- Repositories must belong to the configured **OWNER** (or `OWNER` env override).
- Omitting the dir-spec walks every **immediate** subdirectory of the effective root.
- Prefer pinning third-party submodules by commit (no `branch=` in `.gitmodules`)
  so recursive updates do not float unexpectedly.
- `DRY_RUN=1` is recommended before bulk `cache delete`, `repos release --delete`,
  or mass git operations.

---

## License

MIT License. See [LICENSE](LICENSE) for the full text.
