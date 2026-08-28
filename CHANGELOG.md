# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-08-21

Initial public release of **StormByte-GitHub**: a unified Bash CLI for day-to-day
work across many local GitHub clones under one working root (Actions caches, CI
runs, git ops, pin-aware submodules, source dumps, releases, forks, search/clone
and sanitizer test drivers).

This entry describes the behaviour of the **1.0.0 script**, not the older README
wording that was left behind during silent re-tags of the same version.

### Added

#### Commands

- **`help`** — full usage overview (also shown when invoked with no arguments).
- **`cache show|list|delete`** `[dir-spec] [--name <pattern>]` — list or delete
  GitHub Actions caches; `--name` is a bash glob against the cache key
  (e.g. `*ccache*`).
- **`ci stop|start|restart|status`** `[dir-spec] [--failed]` — cancel active
  runs, re-run completed workflows (optionally only failed/cancelled/timed_out),
  restart = stop then start, status lists recent runs on the current branch.
- **`submodules init|deinit|update|list`** `[dir-spec] [--confirm]` —
  initialise, deinit, list, or update first-level modules.
- **`submodules add`** `<dir> <URL> <path> --name <name>` — add a submodule
  without init/update afterwards. Path must be relative and must not contain `..`.
- **`submodules delete`** `<dir> (--name <name> | --url <url> | --path <path>)`
  — remove the submodule and its folder. Exactly one selector. No commit, no push.
- **`git reset|pull|sync`** `[dir-spec]`
  - `reset` — `git reset --hard` + `clean -fd`, then recursive submodule
    reset/clean and `submodule update --init`.
  - `pull` — `fetch` + `pull --rebase --no-recurse-submodules`, then nested
    modules sync to the SHAs recorded by the parent.
  - `sync` — alias for `pull` (does **not** re-pin first-level modules).
- **`repos list`** — non-recursive folders under `$ROOT`: name, remote, latest
  tag (or HEAD SHA), current branch.
- **`repos dump`** `<directory> [chunk_size_kb] [type=c++|type=cmake]` — dump
  sources into `$OUT`, optionally split into chunks.
- **`repos status`** `[dir-spec]` — working tree + upstream:
  `OK`, `DIRTY`, `UNPUSHED`, or `DIRTY+UNPUSHED`.
- **`repos push`** `[dir-spec]` — `git push` (no `--force`). Skips if no
  upstream. Honours `DRY_RUN`.
- **`repos release`** `[dir-spec] [version] [--list | --delete | --skip-ci]` —
  list GitHub Releases, delete tag+release, or create a release from
  `CHANGELOG.md` on `master`/`main` after an optional CI wait.
- **`clone-all`** — clone every public repository of `$OWNER` missing under
  `$ROOT`.
- **`search <text>`** `[dir-spec]` — search selected clones (`rg` if present,
  else `grep`), excluding `.git` / `build` / `thirdparty` / `dist`.
- **`fork status|sync|branch-status`** `[dir-spec]` — uses `FORK_ROOT` when set,
  otherwise `ROOT`.
  - `status` — behind/ahead vs upstream (forks only).
  - `sync` — update **only** `master` when behind upstream.
  - `branch-status` — local branches ≠ `master`: `CLEAN` / `CONFLICTS` against
    merge into `master` via `merge-tree` (notes local-only / unpushed tips).
- **`test asan|tsan`** `[dir-spec]` plus `--builddir`, `--testdir`, `--outfile`,
  `--cmake-options`, `--executable`, `--test`, `--asan-options` / `--tsan-options`,
  `--verbose`. Build directory is created by the tool and always removed; an
  existing build dir aborts that repo for safety.

#### Configuration

- First-run interactive wizard writes `~/.StormByte-GitHub.conf`.
- Keys: `OWNER`, `ROOT`, `OUT`, optional `FORK_ROOT`.
- Per-run environment overrides: `DRY_RUN`, `ROOT`, `OUT`, `OWNER`, `FORK_ROOT`,
  `PAGE_LIMIT` (default 100), `MAX_PAGES` (0 = until empty), `RUN_LIMIT`
  (0 = all), `ONLY_FAILED` (legacy alias for `--failed`).

#### Directory selection (dir-spec)

- Omit argument → every immediate git repository under the effective root.
- Path or basename → that clone only (absolute, relative, `~/…`, or
  `<root>/<name>`).
- `!name` / `!a+!b` → exclude basenames (quote in interactive shells to avoid
  history expansion).
- The argument is always a **local directory**, never the GitHub repo name.
- `fork` uses `FORK_ROOT` (falling back to `ROOT`) as its walk root.

#### Submodule update policy (1.0.0)

Applies to **first-level** modules only. Nested modules are then synced to the
SHAs recorded by the parent (`--init --recursive` after the gitlinks are staged).

- Fetch the parent first (`origin --prune`).
- If `.gitmodules` has `branch=<name>`, float that module to `origin/<name>`.
- If the checkout is a recognised **release tag**, move to the **next** ref in
  the same family (plain semver or PostgreSQL-style `REL_N_STABLE`). Same tag
  with a different SHA is treated as a re-release of that tag.
- A raw SHA pin is left alone. A missing recorded SHA is an error.
- If the next tag cannot be chosen, list available refs and prompt. Under
  `--confirm` or a non-TTY those rows are skipped (`UNRESOLVED`).
- Prints a `MODULE / CURRENT / PLANNED` table, then asks `[y/N]` on a TTY
  unless `--confirm` or `DRY_RUN=1`.
- Parent dirty worktree is stashed before the plan and restored afterwards
  (`git stash pop`; conflicts leave the stash in place).
- After checkouts, gitlinks are staged with `update-index --cacheinfo 160000`
  **before** the nested update, so children see the new parent pins.
- When no dir-spec is given, parents are walked in **dependency order**
  (Kahn topological sort over `.gitmodules` URLs that resolve to other clones
  under `ROOT`).
- Commits `chore: update dependencies` locally. Does **not** push.

#### Releases (`repos release`)

- `--list` — GitHub Releases only (not bare tags). Optional dir-spec and/or
  version filter. Tags without a matching Release are reported as a note.
- `--delete <dir> <version>` — remove GitHub Release and the tag (local +
  origin) after typing `yes`. No-op if neither exists. `--skip-ci` is ignored.
- Create (default) `<dir> <version> [--skip-ci]` — requires a clean tree on
  `master`/`main`, an OWNER remote, and a non-empty
  `## [<version>] - YYYY-MM-DD` section in `CHANGELOG.md`. Notes come from that
  section. Waits for CI unless `--skip-ci` (Ctrl-C during the wait leaves no
  tag). Same version deletes and recreates tag/release. Pre-releases
  (e.g. `1.0.0-rc.1`) pass `gh --prerelease`. Confirmation is exactly `yes`.
  `DRY_RUN=1` is supported for create and delete.

#### Test driver

- Execution priority per repository: `--executable` → that binary;
  `--test` → `ctest -R REGEX`; otherwise full `ctest`.
- Sanitizer flags are injected first; `--cmake-options` come after so they can
  override. Default CMake extra: `-DENABLE_TEST=ON`.
- Report file (`--outfile`, default `/tmp/<repo>_asan|tsan_report.txt`) is
  written only when the run fails.

#### Packaging and docs

- Bash completion (commands, dir-specs, common flags).
- Manual page (`StormByte-GitHub.1`).
- README (partially stale relative to this script; corrected in a later
  release).
- Depends on **StormByte-functions-bash ≥ 1.1.0** and
  **StormByte-functions-git ≥ 1.2.0**, sourced from next to the script or
  `/lib/StormByte/{functions,git}.sh`.

### Notes

- First stable public release of the StormByte-GitHub CLI.
- Requires `bash`, authenticated `gh`, `jq`, and the StormByte helper libraries.
- Repositories are expected to belong to the configured `OWNER` (or `OWNER`
  env override).
- Designed for a forest of clones under one `ROOT` directory.
- Prefer pinning third-party modules by commit (no `branch=` in `.gitmodules`)
  when floating to a branch tip is not wanted.
- `DRY_RUN=1` is recommended before bulk `cache delete`, `repos release --delete`,
  or mass git operations.

[1.0.0]: https://github.com/StormBytePP/StormByte-GitHub/releases/tag/1.0.0
