# zproj — Agent Instructions

## What this repository is

`zproj` is a single-file bash script (`zproj`) that manages parallel git
worktrees with tmux sessions. Each worktree gets its own tmux window with a
3-pane layout (coding agent, editor, shell/ccm).

The repository also contains:
- `README.md` — user-facing documentation

## Commands

```bash
zproj --test       # run the full test suite (must pass before committing)
zproj --version    # print the current version
zproj --help       # list all subcommands
```

## Working with the codebase

The entire implementation lives in the single file `zproj`. It is a bash
script — read and edit it directly. Do not create additional source files.

**Before committing:**
- Edit `zproj` in this directory (not via any symlink)
- Bump `readonly VERSION=` — patch (x.y.Z) for fixes, minor (x.Y.0) for features
- Run `zproj --test` and confirm: `All N tests passed (0 skipped)`

**Do not:**
- Create additional source files — everything lives in `zproj`
- Commit with failing or skipped tests

## Architecture

`zproj` is organized into these layers, top to bottom:

1. **Constants and helpers** — `VERSION`, color vars, `die`/`warn`/`info`, git and tmux utilities
2. **Tool/editor/agent resolution** — `_resolve_ide`, `_resolve_ai`, `_is_claude_agent`, etc.
3. **Subcommands** — `cmd_<name>` functions, one per user-facing command (`init`, `clone`, `delete`, `list`) plus internal helpers (`cmd_create`, `cmd_launch`, `cmd_open`, `cmd_default`)
4. **Usage functions** — `usage_<name>`, one per user-facing subcommand
5. **Self-test** — `cmd_test()`, containing all tests

## Code conventions

- Subcommand functions: `cmd_<subcommand>`
- Helper functions: `_<name>` (underscore prefix)
- Fatal errors: `die`; non-fatal: `warn`; success: `info`
- Tests: `_t_check <desc> <cmd>` (pass if exit 0), `_t_grep <desc> <pattern> <cmd>` (pass if output matches)
- Test section headers: `_section "XX1 — short description"` where `XX` is a 2-letter prefix unique to that section

## Tests

Every new feature or behaviour change must be accompanied by tests in
`cmd_test()` — no exceptions. Add tests in a new or existing `_section` block
using the appropriate prefix. Choose the next available number within the
prefix (e.g. if `OL7` exists, add `OL8`).
