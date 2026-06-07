# Agent Skills and Instructions

This repository contains shared instructions for AI agents to maintain consistent tool usage, code style, and workflows.

## Repository Structure

```
.
├── AGENTS.md                        # Universal agent rules (tools, workflow, logging, communication)
├── CLAUDE.md                        # Claude Code entry point - imports AGENTS.md
├── skills/
│   └── coding-style/
│       └── SKILL.md                 # Code-specific style rules (Python, JS/TS, general practices)
└── README.md
```

- **`AGENTS.md`** - Universal rules that apply to all agent interactions. References `skills/coding-style/SKILL.md` for coding tasks.
- **`CLAUDE.md`** - Thin wrapper for Claude Code that `@`-imports `AGENTS.md`.
- **`skills/coding-style/SKILL.md`** - Language-specific style guide with examples. Loaded automatically via the `@` directive in `AGENTS.md` when the agent is working on code.

## Configuration Directory Mapping

Different agents load global instructions from specific files in your user profile or home directory. Each agent's config directory needs:

1. **The agent's main instruction file** (varies by agent)
2. **The `skills/` directory** (so `@`-directives in `AGENTS.md` can resolve)

| Agent           | Configuration Directory | Links needed                        |
| --------------- | ----------------------- | ----------------------------------- |
| **Claude Code** | `.claude`               | `CLAUDE.md`, `AGENTS.md`, `skills/` |
| **Codex**       | `.codex`                | `AGENTS.md`, `skills/`              |
| **Copilot**     | `.copilot`              | `AGENTS.md`, `skills/`              |
| **Gemini CLI**  | `.gemini`               | `GEMINI.md`, `AGENTS.md`, `skills/` |

> [!NOTE]
> Claude Code reads `CLAUDE.md` (which `@`-imports `AGENTS.md`), so both files must be present. `AGENTS.md` then `@`-imports skills, which is why every agent directory needs the `skills/` symlink.

## Environment Variables

The persistent command logging feature requires a global environment variable named `LLM_LOGGING_PATH` to be set on your system. This variable specifies the directory where agents will log command execution history.

---

## Setup Instructions

Clone this repository to a local directory, then follow the instructions below for your operating system to create the necessary directories and symbolic links.

Replace `[PATH_TO_CLONED_REPO]` in the commands below with the absolute path to your cloned repository.

### Windows (PowerShell)

Run PowerShell as an **Administrator** or ensure Windows Developer Mode is enabled to create symbolic links.

1. **Create target directories:**

    ```powershell
    New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude", "$env:USERPROFILE\.codex", "$env:USERPROFILE\.copilot", "$env:USERPROFILE\.gemini"
    ```

2. **Create symbolic links (instruction files):**

    ```powershell
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\AGENTS.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\CLAUDE.md" -Value "[PATH_TO_CLONED_REPO]\CLAUDE.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.codex\AGENTS.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.copilot\AGENTS.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.gemini\GEMINI.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    ```

3. **Create symbolic links (skills directory):**
    ```powershell
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills" -Value "[PATH_TO_CLONED_REPO]\skills"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.codex\skills" -Value "[PATH_TO_CLONED_REPO]\skills"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.copilot\skills" -Value "[PATH_TO_CLONED_REPO]\skills"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.gemini\skills" -Value "[PATH_TO_CLONED_REPO]\skills"
    ```

### Windows (Command Prompt - cmd.exe)

Run Command Prompt as an **Administrator**.

1. **Create target directories:**

    ```cmd
    mkdir "%USERPROFILE%\.claude"
    mkdir "%USERPROFILE%\.codex"
    mkdir "%USERPROFILE%\.copilot"
    mkdir "%USERPROFILE%\.gemini"
    ```

2. **Create symbolic links (instruction files):**

    ```cmd
    mklink "%USERPROFILE%\.claude\AGENTS.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.claude\CLAUDE.md" "[PATH_TO_CLONED_REPO]\CLAUDE.md"
    mklink "%USERPROFILE%\.codex\AGENTS.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.copilot\AGENTS.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.gemini\GEMINI.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    ```

3. **Create symbolic links (skills directory):**

    ```cmd
    mklink /D "%USERPROFILE%\.claude\skills" "[PATH_TO_CLONED_REPO]\skills"
    mklink /D "%USERPROFILE%\.codex\skills" "[PATH_TO_CLONED_REPO]\skills"
    mklink /D "%USERPROFILE%\.copilot\skills" "[PATH_TO_CLONED_REPO]\skills"
    mklink /D "%USERPROFILE%\.gemini\skills" "[PATH_TO_CLONED_REPO]\skills"
    ```

### Linux / macOS (Bash or Zsh)

1. **Create target directories:**

    ```bash
    mkdir -p ~/.claude ~/.codex ~/.copilot ~/.gemini
    ```

2. **Create symbolic links (instruction files):**

    ```bash
    ln -sf "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.claude/AGENTS.md
    ln -sf "[PATH_TO_CLONED_REPO]/CLAUDE.md" ~/.claude/CLAUDE.md
    ln -sf "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.codex/AGENTS.md
    ln -sf "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.copilot/AGENTS.md
    ln -sf "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.gemini/GEMINI.md
    ```

3. **Create symbolic links (skills directory):**

    ```bash
    ln -sfn "[PATH_TO_CLONED_REPO]/skills" ~/.claude/skills
    ln -sfn "[PATH_TO_CLONED_REPO]/skills" ~/.codex/skills
    ln -sfn "[PATH_TO_CLONED_REPO]/skills" ~/.copilot/skills
    ln -sfn "[PATH_TO_CLONED_REPO]/skills" ~/.gemini/skills
    ```
