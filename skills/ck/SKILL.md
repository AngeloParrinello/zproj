---
name: ck
description: "Use ck to find anything in any file in the repository: identifiers, types, patterns, concepts, build files, config files. Replaces grep, rg, ripgrep, awk, and sed for searching files — never use those. Triggers: find, search, locate, where is, usages, references, depends on, grep, rg, ripgrep, awk, sed."
license: MIT
compatibility: opencode
metadata:
  category: search
  replaces: grep, rg, Glob
---

# ck — Semantic Code Search

## Setup

```bash
ck --index .     # build index (required for --sem, --lex, --hybrid)
ck --status .    # check index status
ck --clean .     # remove entire index
```

Auto-indexes on first `--sem`. Run `--index` explicitly before using `--lex` or `--hybrid`.

**`.ckignore`** excludes files from all search modes — use `grep`/`rg` for excluded types (`.json`, `.yaml`, etc.). Auto-created on first `--index`; check it if expected files aren't appearing.

**Gitignore:** Run `echo "*" > .ck/.gitignore` after first indexing — `ck` does not create this automatically.

**Git worktrees (zproj):** never cold-index a new worktree — copy from a sibling:

```bash
ls -dt ../*/. | head -5 && cp -r ../main/.ck ./.ck && echo "*" > .ck/.gitignore && ck --index .
```

For call graph analysis (who calls this, what breaks if I change it), use `cqs` instead.

## Search Modes

| Mode | Flag | Use for |
|---|---|---|
| Semantic | `--sem` | Concepts, behavior, intent — "retry logic", "input validation" |
| Lexical | `--lex` | Ranked full-text — requires a prior `--index` run |
| Hybrid | `--hybrid` | Balance precision and recall |
| Regex | _(default)_ | Exact identifiers and patterns — `fn authenticate`, `class.*Handler` |

## Critical Syntax Rules

**Quoting:** Use single quotes for patterns containing `$` or `[` — double quotes allow shell expansion that silently corrupts the pattern.

```bash
ck 'Scope\.$'   # correct
ck "Scope\.$"   # wrong — $ may be expanded by the shell
```

## Common Invocations

Default `--sem` threshold is `0.6`. Too few results → lower; too many → raise. Range: `0.3`–`0.9`. `--hybrid` uses RRF — useful range is `0.01`–`0.05`.

```bash
ck --sem "error handling" .
ck --sem --threshold 0.8 "authentication logic" .   # raise precision
ck --sem --threshold 0.5 "retry" .                  # lower precision
ck --sem --limit 10 "caching strategy" .
ck --sem --rerank "authentication logic" .
ck --hybrid --threshold 0.02 "async function" .     # RRF range: 0.01–0.05
ck 'fn authenticate' src/                           # regex, no index needed
ck --jsonl --sem "error handling" .                 # JSONL for agent consumption
ck --jsonl --no-snippet --sem "error handling" .
ck --json --sem "error handling" .                  # JSON (fields: file, preview, lang)
```

## Do / Don't

**Do:**
- Use single quotes for patterns containing `$` or `[`
- Run `echo "*" > .ck/.gitignore` after first indexing
- Run `ck --index .` before using `--lex` or `--hybrid`
- Use `--jsonl --limit 20` when processing results programmatically

**Don't:**
- Use `grep`, `rg`, or Glob for source code — use them only for excluded file types
- Use double quotes for patterns with `$` or `[`
- Use `--lex` without a prior `--index` run
