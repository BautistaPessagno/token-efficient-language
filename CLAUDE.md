# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code skill distribution repository** for TEL (Token Efficient Language) — a source-to-source compiled DSL that represents C programs in compact syntax optimized for LLM token efficiency. It contains no compiler implementation; it is a skill that teaches AI agents to read, write, and translate TEL.

## Repository Structure

```
skills/tel-language/        # Canonical skill source
  SKILL.md                  # Skill entrypoint — TEL syntax, keyword table, examples
  references/grammar.md     # Full formal grammar and edge-case reference
.agents/skills/tel-language/ # Copy used by agent runtimes
.claude/skills/tel-language  # Symlink → .agents/skills/tel-language
skills-lock.json             # Tracks skill source (GitHub: BautistaPessagno/token-efficient-language)
```

The `.claude/skills/` symlink is how Claude Code discovers and loads the skill. `skills-lock.json` pins the source for reproducible skill installs.

## Working on the Skill

All skill content lives in `skills/tel-language/`. The `.agents/` directory is a runtime copy — keep it in sync with `skills/` when making changes. `skills-lock.json` records the source repo and a content hash; update the hash when the skill content changes significantly.

**SKILL.md** is the primary entrypoint. It must contain the `description:` frontmatter field (used by the skill loader for trigger matching) and the `---` YAML front matter block.

**grammar.md** is the exhaustive reference linked from SKILL.md. It documents edge cases, error conditions, and the complete keyword mapping table.

## TEL Language Summary

TEL compiles to C. Core rules:
- Indentation replaces `{ }` braces; newlines replace `;`
- `fn name params -> type` for functions; `main` expands to full C main signature
- `fd i 0 n` → `for(int i=0; i<n; i++)` (most common pattern)
- `name:type = value` for variable declarations
- Abbreviated types: `uint` = `unsigned int`, `uli` = `unsigned long int`, `v` = `void`
- Abbreviated keywords: `w` = `while`, `dw` = `do`, `sw` = `switch`, `df` = `default`, `brk` = `break`, `cnt` = `continue`, `elif` = `else if`
- Standard I/O/stdlib/string includes are injected automatically when their functions are used
- Standard `#include` headers omit `<>` and `.h` (e.g. `#include stdio`)

Reserved words that cannot be used as identifiers: `fn ret str en td stc fd dw sw df brk cnt uint uli v null w main elif`
