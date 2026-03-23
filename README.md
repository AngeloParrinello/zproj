# zproj

```
███████╗██████╗ ██████╗  ██████╗      ██╗
╚══███╔╝██╔══██╗██╔══██╗██╔═══██╗     ██║
  ███╔╝ ██████╔╝██████╔╝██║   ██║     ██║
 ███╔╝  ██╔═══╝ ██╔══██╗██║   ██║██   ██║
███████╗██║     ██║  ██║╚██████╔╝╚█████╔╝
╚══════╝╚═╝     ╚═╝  ╚═╝ ╚═════╝  ╚════╝
```

**Parallel workspaces with bare git worktrees and tmux.**

Each feature branch gets its own directory and a tmux window with a 3-pane
layout — coding agent, editor, and shell — all opened in the right place
automatically. Worktrees, branches, and windows are created and torn down
together as a unit.

## Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| bash | 3.2+ | Runtime (bash 4+ preferred) |
| git | 2.5+ | Worktree support |
| tmux | 3.0+ | Named pane support |

One of the following coding agents is auto-discovered (or set `$CODING_AGENT`):
`opencode`, `claude`, `codex`, `amp`, `aider`, `goose`, `gemini`

One of the following editors is auto-discovered (or set `$ZPROJ_EDITOR`):
`idea` (IntelliJ IDEA)

## Installation

```bash
# Download and make executable
curl -o ~/.local/bin/zproj https://raw.githubusercontent.com/jdegoes/zproj/main/zproj
chmod +x ~/.local/bin/zproj

# Verify
zproj --version
```

## Quick start

```bash
# New project from scratch
zproj init my-project
cd my-project
zproj                        # opens tmux session with main worktree

# Clone an existing repo
zproj clone git@github.com:you/my-project.git
cd my-project
zproj

# Work on a feature
zproj feature-auth           # creates worktree + branch + tmux window
zproj delete feature-auth    # tears it all down when done
```

## How it works

### Directory structure

`zproj init` or `zproj clone` creates a bare repo with one worktree per branch:

```
my-project/
  .bare/              bare git repository
  .git                pointer: "gitdir: ./.bare"
  main/               worktree for the main branch
  feature-auth/       worktree for feature-auth branch
  fix-bug/            worktree for fix-bug branch
```

Each worktree directory is a fully independent working tree. You can have
multiple branches open simultaneously without stashing or switching.

### Tmux layout

One tmux **session** per project, one **window** per worktree. Each window has
three panes, all opened in the worktree directory:

```
┌─────────────────┬──────────────────┐
│                 │                  │
│  coding agent   │     editor       │
│  (pane 1)       │     (pane 2)     │
│                 ├──────────────────┤
│                 │                  │
│                 │  shell / ccm     │
│                 │     (pane 3)     │
└─────────────────┴──────────────────┘
```

When the coding agent is `claude`, pane 3 runs `ccm --plan pro` instead of a
plain shell.

### Tool resolution

```bash
zproj --env          # show which editor and agent will be used
zproj --diagnostics  # check the full environment for problems
```

| Env var | Purpose |
|---------|---------|
| `CODING_AGENT` | Override coding agent (e.g. `export CODING_AGENT=claude`) |
| `ZPROJ_EDITOR` | Override editor (e.g. `export ZPROJ_EDITOR=goland`) |

## Command reference

```
zproj                                  Open project session (from repo root)
zproj <worktree-dir>                   Create (if needed) and launch
zproj init <dir> [--main <branch>]     Init, convert, or upgrade to bare worktree repo
zproj clone <git-url> [dir]            Clone remote repo as bare worktree structure
zproj delete <worktree-dir> [--force]  Remove worktree, branch, and window
zproj list [dir]                       Show worktrees with status
zproj --env                            Show resolved editor and coding agent
zproj --diagnostics                    Check environment for problems
zproj --test                           Run the built-in test suite
```

Run `zproj <command> --help` for details on any command.

### `zproj init`

Behavior depends on whether the directory already exists:

- **Does not exist** — creates a new empty bare worktree repo
- **Exists, no `.git`** — converts the directory (files moved into `<branch>/`)
- **Exists, has `.git/`** — upgrades the repo in-place (history preserved)

### `zproj <worktree-dir>`

The primary command for daily use. From the project root:

```bash
zproj feature-auth   # creates worktree + branch + opens tmux window
zproj feature-auth   # already exists → switches to the window
```

## Self-test

```bash
zproj --test
```

## License

MIT — see [LICENSE](LICENSE).
