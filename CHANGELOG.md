# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-08-21

Initial public release of **StormByte-GitHub**: a unified Bash CLI for managing
GitHub Actions caches, CI workflow runs, git operations, pin-aware submodules,
source dumps, releases, forks, search/clone helpers and test drivers across many
local repository clones under one working root.

### Added

#### Commands
- **`help`** — full usage overview
- **`cache show|list|delete`** — manage Actions caches; optional `--name` with wildcards
- **`ci stop|start|restart|status`** — manage workflow runs; `--failed` for failed/cancelled/timed_out runs
- **`submodules init|deinit|update|list|add`** — submodule lifecycle and listing
- **`git status|sync`** — status and sync over one repo or the whole tree
- **`repos list|status|push|dump|release`** — inventory, push, source dump, and release create/list/delete
- **`fork status|sync`** — fork tree operations under optional `FORK_ROOT`
- **`search`** — search repositories for the configured owner
- **`clone-all`** — clone missing repositories for the configured owner
- **`test`** — drive ctest or an executable with sanitizer presets and extra CMake options

#### Configuration
- First-run interactive config written to `~/.StormByte-GitHub.conf`
- Keys: `OWNER`, `ROOT`, `OUT`, optional `FORK_ROOT`
- Per-run environment overrides: `DRY_RUN`, `ROOT`, `OUT`, `OWNER`, `FORK_ROOT`,
  `PAGE_LIMIT`, `MAX_PAGES`, `RUN_LIMIT`, `ONLY_FAILED`

#### Directory selection (dir-spec)
- Omit argument → all immediate git repos under the effective root
- Path or basename → single repository
- `!name` / `!a+!b` → exclude basenames (quote in interactive shells)

#### Submodule update policy
- Root modules with `branch=` in `.gitmodules` track that branch
- Root modules without `branch=` stay on the pinned gitlink SHA
- Nested modules updated with `--init --recursive` so parent-recorded pins are preserved

#### Releases
- Create, list and delete releases via `repos release`
- Options such as `--list`, `--skip-ci`, `--delete`
- `DRY_RUN=1` support for dry-run releases

#### Test driver
- Priority: `--executable` → binary; `--test` → `ctest -R`; else full `ctest`
- Sanitizer presets (e.g. asan, tsan), `--cmake-options`, `--verbose`, `--outfile`

#### Packaging and docs
- Bash completion (commands, dir-specs, common flags)
- Manual page (`StormByte-GitHub.1`)
- README documenting CLI behaviour
- Depends on StormByte-functions-bash and StormByte-functions-git ≥ 1.1.0

### Notes

- This is the first stable public release of the StormByte-GitHub CLI.
- Requires `bash`, authenticated `gh`, `jq`, and the StormByte helper libraries.
- Repositories are expected to belong to the configured OWNER (or OWNER env override).
- Designed for developers who keep an organization of clones under one ROOT directory.

[1.0.0]: https://github.com/StormBytePP/StormByte-GitHub/releases/tag/1.0.0