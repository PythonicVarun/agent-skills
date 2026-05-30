# Agent Skills and Instructions

This repository contains shared instructions for AI coding agents to maintain consistent tool usage, code styling, and workflows.

## Configuration Directory Mapping

The shared rules are defined in `AGENTS.md`. Different coding agents load global instructions from specific files in your user profile or home directory. Below is the mapping of where the symbolic links point:

| Coding Agent    | Configuration Directory | Target Link Name |
| --------------- | ----------------------- | ---------------- |
| **Claude Code** | `.claude`               | `CLAUDE.md`      |
| **Codex**       | `.codex`                | `AGENTS.md`      |
| **Copilot**     | `.copilot`              | `AGENTS.md`      |
| **Gemini CLI**  | `.gemini`               | `GEMINI.md`      |

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

2. **Create symbolic links:**
    ```powershell
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\CLAUDE.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.codex\AGENTS.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.copilot\AGENTS.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.gemini\GEMINI.md" -Value "[PATH_TO_CLONED_REPO]\AGENTS.md"
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

2. **Create symbolic links:**

    ```cmd
    mklink "%USERPROFILE%\.claude\CLAUDE.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.codex\AGENTS.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.copilot\AGENTS.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    mklink "%USERPROFILE%\.gemini\GEMINI.md" "[PATH_TO_CLONED_REPO]\AGENTS.md"
    ```

### Linux / macOS (Bash or Zsh)

1. **Create target directories:**

    ```bash
    mkdir -p ~/.claude ~/.codex ~/.copilot ~/.gemini
    ```

2. **Create symbolic links:**

    ```bash
    ln -s "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.claude/CLAUDE.md
    ln -s "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.codex/AGENTS.md
    ln -s "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.copilot/AGENTS.md
    ln -s "[PATH_TO_CLONED_REPO]/AGENTS.md" ~/.gemini/GEMINI.md
    ```
