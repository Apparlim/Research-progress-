# OpenAI Codex CLI: Storage & Data Flow

## Deliverables
- **[`Source-detail.pdf`](Source-detail.pdf)**: Storage persistence table and 1-page architecture diagram[cite: 6, 7].
- **`report.tex`**: LaTeX source code.

## Storage Persistence Map

| Store | What it Stores | Written By | Auto-Loaded? | How Deleted |
| :--- | :--- | :--- | :--- | :--- |
| `~/.codex/config.toml` | User configuration, sandbox policy, and feature flags[cite: 6]. | User / Harness[cite: 6] | YES[cite: 6] | Manual file deletion[cite: 6]. |
| `~/.codex/auth.json` | API keys, OAuth tokens, and account identity[cite: 6]. | Harness[cite: 6] | YES[cite: 6] | `codex logout` or manual deletion[cite: 6]. |
| `~/.codex/state_5.sqlite` | Active threads, git metadata, and token counts[cite: 6]. | Harness[cite: 6] | YES[cite: 6] | `codex delete` or database reset[cite: 6]. |
| `~/.codex/sessions/...` | Full append-only conversation transcript and tool outputs[cite: 6]. | Harness[cite: 6] | NO (Yes on resume)[cite: 6] | `codex delete` or manual purge[cite: 6]. |
| `~/.codex/memories_1.sqlite` | Extracted long-term agent memories and facts[cite: 6]. | Harness[cite: 6] | YES[cite: 6] | Thread or row deletion[cite: 6]. |
| `~/.codex/AGENTS.md` | Global user instructions and behavior guidelines[cite: 6]. | User[cite: 6] | YES[cite: 6] | Manual file deletion[cite: 6]. |
| `<project>/AGENTS.md` | Project-specific rules and build instructions[cite: 6]. | Developer[cite: 6] | YES[cite: 6] | Manual deletion or `git rm`[cite: 6]. |

## Architecture & Data Flow Diagram

```mermaid
flowchart TD
    config["~/.codex/config.toml<br/>~/.codex/auth.json"]:::store
    agents["~/.codex/AGENTS.md<br/>&lt;project&gt;/AGENTS.md"]:::store
    
    init(["<b>Core Initialization</b><br/>(codex-rs Launch)"]):::process
    runtime(["<b>Runtime Execution</b><br/>(Active Context & Prompts)"]):::process
    
    state["~/.codex/state_5.sqlite<br/>(Thread & Token Metadata)"]:::store
    sessions["~/.codex/sessions/...<br/>(Rollout Transcripts)"]:::store
    memories["~/.codex/memories_1.sqlite<br/>(Extracted Long-Term Facts)"]:::store

    config -->|Auto-loads settings/auth| init
    agents -->|Hierarchical merge| init
    init --> runtime

    runtime <-->|Reads/Writes state| state
    runtime <-->|Appends turns / Compacts| sessions
    runtime <-->|Extracts & injects memories| memories

    classDef store fill:#f3f4f6,stroke:#374151,stroke-width:1.5px,font-family:monospace;
    classDef process fill:#ffffff,stroke:#111827,stroke-width:2px;
```

## Source Code Findings
- **File Writes (`~/.codex`)**: Grepped `codex-rs` to map all persistent stores, identifying whether each is written mechanically by the harness or by the user, auto-loaded on startup, and how it is deleted[cite: 6, 7].
- **Compaction Prompt**: Located `SUMMARIZATION_PROMPT` and `SUMMARY_PREFIX` in the source code. When context limits are reached, the harness prompts the model to summarize prior messages and replaces older turns with the compacted summary.
- **AGENTS.md Merging**: Inspected `codex-rs/core/src/agents_md.rs`. The harness discovers global instructions (`~/.codex/AGENTS.md`) and project-level instructions (`<project>/AGENTS.md`), merging them hierarchically into the initial context[cite: 6, 7].

## PowerShell Verification & Source Inspection

```powershell
# 1. Search source code for ~/.codex paths and persistence stores
Select-String -Path "codex-rs\**\*.rs" -Pattern "\.codex|config\.toml|auth\.json|sqlite|AGENTS\.md"

# 2. Locate the conversation compaction prompt
Select-String -Path "codex-rs\**\*.rs" -Pattern "SUMMARIZATION_PROMPT|SUMMARY_PREFIX"

# 3. Inspect AGENTS.md discovery and hierarchical merging
Get-Content "codex-rs\core\src\agents_md.rs"

# 4. Verify local runtime directory structure (API key masked in .env)
# GEMINI_API_KEY="........."
codex exec "list the files in this repo"
Get-ChildItem -Path "$HOME\.codex" -Force
```
