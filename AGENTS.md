# zproj — Agent Instructions

## What this repository is

`zproj` is a single-file bash script (`zproj`) that manages parallel git
worktrees with tmux sessions. Each worktree gets its own tmux window with a
3-pane layout (coding agent, editor, shell). It also provides a review
workflow for annotating diffs and dispatching notes to a coding agent.

The repository also contains:
- `skills/` — bundled skill files installed by `zproj integrate` into the
  coding agent's global skills directory
- `rules/` — rule files installed by `zproj integrate` into the agent's
  global instructions file (e.g. `~/.config/opencode/AGENTS.md`)
- `README.md` — user-facing documentation

## Working with the codebase

The entire implementation lives in the single file `zproj`. It is a bash
script — read and edit it directly. Do not create additional source files.

When making changes:
- Edit `zproj` in this directory (not via any symlink)
- Bump `readonly VERSION=` (semver patch for fixes, minor for features)
- Every new feature or behaviour change must be accompanied by tests in
  `cmd_test()` — no exceptions
- If you change the reference editor integration (the Lua code bundled in
  `_integrate_build_prompt`), you must also apply the same change to the
  live editor config at `~/.config/nvim/lua/plugins/diffview.lua` so they
  stay in sync
- Run `zproj --test` to execute the built-in test suite before committing
- All tests must pass (0 failures) before committing

## Code conventions

- Functions are named `cmd_<subcommand>` for top-level subcommands
- Helper functions are prefixed with `_`
- Tests live inside `cmd_test()` and use `_t_check` / `_t_grep` helpers
- Test sections are named with a short prefix (e.g. `OL1`, `SK5`, `IN2`)
  followed by a description; add new tests in the relevant section
- Use `die` for fatal errors, `warn` for non-fatal, `info` for success

## Search

Use the `ck` skill to search this codebase. Never use `grep` or `rg`.
