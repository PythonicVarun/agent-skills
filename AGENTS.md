# AGENTS instructions

## Tools

Prefer these over alternatives:

> **Python rule:** Always use `uv`/`uvx` instead of `python` or `pip`. No exceptions.
>
> - Run scripts: `uv run script.py` not `python script.py`
> - Install packages: `uv pip install` not `pip install`
> - Sync dependencies from lockfile: `uv sync` instead of installing manually
> - Run tools: `uvx ruff`, `uvx yt-dlp`, etc. not `pip install` + run

| Purpose                   | Use                                 |
| ------------------------- | ----------------------------------- |
| Python runtime & packages | `uv run`, `uv pip`, `uvx`           |
| Linting / formatting      | `uvx ruff check`, `uvx ruff format` |
| File search               | `fd` (find), `rg` (grep)            |
| Media / docs              | `uvx yt-dlp`, `uvx markitdown`      |
| Databases                 | `duckdb`, `sqlite3`                 |
| HTTP                      | `curl`                              |
| JSON processing           | `jq`                                |
| VCS                       | `git`                               |

## Code Style

@./skills/coding-style/SKILL.md

## Workflow

1. **Read before writing.** Understand the existing structure before adding or editing files.
2. **Targeted edits.** Change only what the task requires - don't refactor unrelated code.
3. **Verify before marking done.** Run the relevant script or test; don't declare success without evidence.
4. **Use the right tool for search.** `rg` for content, `fd` for filenames - don't `ls` recursively or `cat` whole directories.
5. **Prefer in-place fixes.** Edit files directly rather than creating copies or backups unless asked.
6. **No direct `.env` file access.** Never read or write `.env` files. If environment variables are required, ask the user for the specific variables rather than handling the file directly.

## Persistent Command Logging

If you run commands that make permanent system changes (Registry edits, shell profile/startup changes, service/scheduled-task changes, or persistent environment-variable changes), log the exact command and its result to:

```bash
$LLM_LOGGING_PATH/<agent_currently_user_talking_to_like_copilot_codex_etc>/cmds/<date>_<model>.log
```

## Output & Communication

- Be concise. Explain _what_ changed and _why_, not a line-by-line narration.
- If the prompt is unclear or underspecified, ask for clarification before proceeding - don't make assumptions that could lead the work in the wrong direction.
- Surface blockers immediately (missing dependency, permission error, unclear requirement) instead of working around them silently.
