---
name: cqs
description: "Use cqs for call graph analysis: callers, callees, impact of changing a function, dead functions, refactoring safety, diff risk. Requires a callable function name — does not find text, types, or identifiers. Triggers: who calls this, what breaks if I change, impact analysis, callers, callees, dead code, refactor function, rename function, review diff."
license: MIT
compatibility: opencode
metadata:
  category: code-intelligence
  complements: ck
---

# cqs — Code Intelligence CLI

## Setup (required before first use in any project)

```bash
cqs init        # Download ML model (~547MB, one-time per machine)
cqs index       # Index the codebase (builds embeddings + call graph + HNSW)
```

Run `cqs watch` afterward to keep the index fresh automatically.

**Re-index when:** new files were added outside of `watch`, or call graph shows 0 entries.

## Git Worktrees (zproj)

**Never cold-index a new worktree.** Copy the index from a sibling and delta-index instead:

```bash
ls -dt ../*/. | head -5        # find most recently indexed sibling
cp -r ../main/.cqs ./.cqs      # copy its index
cqs index                      # delta-index
```

For finding code by text, pattern, or concept, use `ck` instead — `cqs` does not search source files.

## Critical Syntax Rules

**`-q` is a global flag — it must come before the subcommand:**
```
cqs -q <command> <args> --json     ✓
cqs <command> <args> --json -q     ✗ (error: unexpected argument)
```

**`notes add/remove/update` do not accept `-q` at all.**

**`trace`, `impact`, `review`, `ci` use `--format json`, not `--json`:**
```
cqs impact "encode" --format json     ✓
cqs -q impact "encode" --json         ✗
```

## Do / Don't

**Do:**
- Use single quotes for queries containing `$` — in double quotes, `$letter` is silently expanded by the shell, changing the query without any error
- Run `cqs init` + `cqs index` at the start of every new project
- Start with `scout` or `task` before implementing
- Use `impact` before any refactor — even "small" changes can have transitive callers
- Use `--format json` for `trace`, `impact`, `review`, and `ci`
- Use `-q` before the subcommand, never after
- Use `cqs watch` in the background during long coding sessions

**Don't:**
- Skip `cqs index` — without it, call graph and type graph will be empty
- Use `--json` with `trace` or `impact` — it will error; use `--format json`
- Append `-q` after the subcommand — it will error
- Run `cqs index` repeatedly — it's expensive; `watch` handles incremental updates

---

## Search & Discovery

**There is no `search` subcommand.** The query is a bare positional argument.
```bash
cqs -q "error handling" --json          # correct
cqs -q search "error handling" --json   # WRONG — search is not a subcommand
```

```bash
cqs -q "<query>" [--lang L] [-n N] [-t N] [--name-only] [--semantic-only] [--rerank]
    [--path glob] [--chunk-type T] [--pattern P] [--tokens N] [--expand]
    [-C N] [--no-content] [--no-stale-check] --json

cqs -q similar "<name>" --json
cqs -q gather "<query>" [--expand N] [--direction both|callers|callees] [-n N] [--tokens N] --json
cqs -q where "<description>" --json
cqs -q scout "<task>" [-n N] [--tokens N] --json
cqs -q task "<description>" [-n N] [--tokens N] --json
cqs -q onboard "<concept>" [-d N] [--tokens N] --json
cqs read <path> [--focus <function>] --json
cqs context <path> [--compact] [--summary] --json
cqs explain "<name>" --json
```

## Call Graph

```bash
cqs -q callers "<name>" --json
cqs -q callees "<name>" --json
cqs trace "<source>" "<target>" --format json       # --format json, not --json
cqs -q deps "<name>" --json                         # forward: who uses this type?
cqs -q deps --reverse "<name>" --json               # reverse: what types does this use?
cqs -q related "<name>" --json
cqs impact "<name>" [--depth N] [--suggest-tests] [--include-types] --format json
cqs impact-diff [--base <ref>] [--stdin] [--json] [--tokens N]
cqs -q test-map "<name>" --json
cqs blame "<name>" [--callers]
```

## Quality & Review

```bash
cqs -q dead [--include-pub] [--min-confidence low|medium|high] --json
cqs -q stale --json
cqs -q health --json
cqs -q suggest [--apply] --json
cqs -q gc --json
cqs -q stats --json
cqs review [--base <ref>] [--stdin] [--format json] [--tokens N]
cqs ci [--base <ref>] [--stdin] [--gate high|medium|off] [--format json] [--tokens N]
```

## Notes

Notes do not accept `-q`.

```bash
cqs notes add "<text>" [--sentiment N] [--mentions a,b,c]
cqs notes list [--json]
cqs notes update "<exact text>" [--new-text "..."] [--new-sentiment N]
cqs notes remove "<exact text>"
cqs audit-mode [on|off] [--expires 30m] --json
```

Sentiment: -1 (serious pain), -0.5 (notable pain), 0 (neutral), 0.5 (notable gain), 1 (major win).

## Infrastructure

```bash
cqs ref add <name> <path> [--weight 0.8]
cqs ref list
cqs ref update <name>
cqs ref remove <name>
cqs watch [--debounce <ms>] [--no-ignore]
cqs convert <path> [-o <dir>] [--overwrite] [--dry-run] [--clean-tags <tags>]
```
