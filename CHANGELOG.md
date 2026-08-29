# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `submodules update --latest`: plan the newest recognised release in the
  module (may cross major). Default stays in-family (B1 / B2).
- `submodules details` [dir-spec] [--all]: first-level NAME / URL / PIN /
  LATEST. LATEST is the newest recognised release (`git_pin_latest_recognised`).
  `(semver)` only when the ref text is not already that semver. `--all` lists
  tags on master/main and origin branches; current pin marked with `*`.

[Unreleased]: https://github.com/StormBytePP/StormByte-GitHub/compare/1.1.0...HEAD

## [1.1.0] - 2026-08-28

Rewrite of `submodules update` around the 1.3.0 helper libraries, plus a coherent
colour palette and help/docs that match the script that actually ships.

Requires **StormByte-functions-bash ≥ 1.3.0** and **StormByte-functions-git ≥ 1.3.0**.

### Added

- Pin policy **A / B1 / B2** for first-level `submodules update`:
  - **A** — parent `.gitmodules` has `branch=master`: float to `origin/master`.
    Already at that SHA → *No changes*.
  - **B1** — no master; checkout is a recognised release tag: move to the
    **latest** tag in the same family (same prefix and grain). Same tag,
    different SHA → re-release.
  - **B2** — no master; checkout is **not** a tag (e.g. `branch=master` was
    removed): pin the latest tag of the inferred family. A downgrade is
    intentional. If no family can be inferred, the newest recognised release.
  - Unrecognised pins → `UNRESOLVED`. On a TTY you can type a ref;
    `--confirm` / non-TTY skip those rows.
- Extensible family schemes in `git.sh` (used by the planner):
  - own libraries `X.Y` / `X.Y.Z`
  - PostgreSQL `REL_16_4` → 16.4, family by major (`REL_16_*`)
  - Crypto++ `CRYPTOPP_8_9_0` → 8.9.0, family by `X.Y` while the prefix matches
- Planner integration: `git_sm_plan`, `git_shadow_fetch` / `git_shadow_delete`,
  `git_pin_latest_in_family`, `git_sm_checkout_branch` / `git_sm_checkout_tag`,
  `git_sm_update_nested`, `git_peel_ref`, `git_list_tag_names`,
  `git_canonical_repo_id`, `git_gitmodules_list`.
- Shared colour helpers: `_clr_wrap`, `_clr_planned`, `_clr_status_label`,
  `_clr_merge_label`, `_clr_ci_status`, `_clr_ci_conclusion`, `_idle`.
- Palette used across the CLI (headers unchanged):
  - dark green — idle / *No changes* / OK with no work
  - light green (bold) — planned or applied update
  - yellow — warnings (`DIRTY`, `UNPUSHED`, unresolved prompts)
  - red — failures, `DIRTY+UNPUSHED`, `CONFLICTS`, error / UNRESOLVED rows
- `confirm_yes` for the apply-plan prompt (TTY only). `--confirm` or a non-TTY
  applies the plan without asking.
- Help text for `submodules update` documents A / B1 / B2, stash/restore,
  first-level vs nested, and the local `chore: update dependencies` commit.
- Help text for `git reset|pull|sync` documents the real side effects
  (hard reset + submodule reset; pull without re-pinning first-level modules).

### Changed

- `STORMBYTE_GITHUB_VERSION` `1.0.0` → `1.1.0`.
- `REQUIRE_STORMBYTE_FUNCTIONS` `1.1.0` → `1.3.0`.
- `REQUIRE_STORMBYTE_FUNCTIONS_GIT` `1.2.0` → `1.3.0`.
- Path input uses `path_normalize` from `functions.sh` instead of a private
  `_normalize_path_input`.
- `canonical_repo_id` is a thin wrapper over `git_canonical_repo_id`.
- `cmd_submodules_update_one` no longer contains the old in-script family
  planner. It consumes `git_sm_plan` (`action`, `kind`, `current`, `planned`,
  `target`, `sha`) and, on tag updates, prefers `git_pin_latest_in_family`
  so B1 jumps to the tip of the family rather than only the immediate next tag.
- Tag-family advance in 1.0.0 was “next ref in the same family”. In 1.1.0 it
  is “latest tag in the same family”.
- Unresolved-ref listing uses shadow refs (`refs/sb-gh/tags`, `refs/sb-gh/heads`)
  via `git_list_tag_names`.
- Idle successes (`No caches`, `No .gitmodules`, `No changes`, `Not a fork`,
  `Nothing to check`, already-up-to-date fork sync, …) print through `_idle`
  (dark green) instead of `ok`.
- `repos status` labels, `fork branch-status` `CLEAN`/`CONFLICTS`, and
  `ci status` STATUS/CONCLUSION cells are colourised with the same palette.
- The `MODULE / CURRENT / PLANNED` table colourises only the last column so
  alignment is preserved.

### Fixed

- Help and README described `git status|sync`, omitted `submodules delete`
  and `fork branch-status`, and documented a pin policy that the 1.0.0 script
  had already outgrown. Help now matches the binary.
- Family detection no longer falls over on PostgreSQL `REL_*` or Crypto++
  `CRYPTOPP_*` tags (prefix + grain live in `git.sh`, not ad-hoc regex in the
  CLI).
- Transition “had a tag, now `branch=master`” floats to the branch tip (A)
  instead of staying on the old tag.
- Transition “had `branch=master`, line removed” pins a family tag (B2)
  instead of keeping a floating SHA with no policy.
- Parent stash / pop, gitlink `update-index --cacheinfo 160000` **before**
  nested sync, topological walk of `ROOT`, `--confirm`, and
  `chore: update dependencies` (no push) are unchanged and still applied
  after the planner rewrite.

### Removed

- Private in-script path normaliser `_normalize_path_input`.
- Private in-script tag-family planner (`_sm_plan_tag` and related helpers).
  Planning lives in `git.sh` (`git_sm_plan` and the pin-scheme table).

### Notes

- Nested modules are still only synced to SHAs recorded by the parent after
  first-level gitlinks are staged. `git pull` / `git sync` still do **not**
  re-pin first-level modules.
- Removing `branch=master` from `.gitmodules` is the explicit signal for B2,
  including an intentional downgrade onto a release tag.
- README rewritten to match this release. The 1.0.0 changelog entry was
  restated against the 1.0.0 script (not the stale README of that day).

[1.1.0]: https://github.com/StormBytePP/StormByte-GitHub/releases/tag/1.1.0

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
