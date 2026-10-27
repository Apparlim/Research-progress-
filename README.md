# OpenAI Codex CLI: Storage & Data Flow

## Deliverables
- **`report.pdf`**: Storage persistence table and 1-page architecture diagram.
- **`report.tex`**: LaTeX source code.
- **`diagram.png`**: Data flow diagram.

## Summary Diagram
<img width="500" src="https://github.com/user-attachments/assets/170a341e-ebee-43b0-b513-74e606c24a20" />

## Source Code Findings
- **File Writes (`~/.codex`)**: Grepped `codex-rs` to map all persistent stores (`config.toml`, `auth.json`, `state_5.sqlite`, `sessions/`, `memories_1.sqlite`), identifying whether each is written by the harness or user, auto-loaded on startup, and how it is deleted.
- **Compaction Prompt**: Read `SUMMARIZATION_PROMPT` and `SUMMARY_PREFIX` in the source code. When a conversation gets too long, the harness asks the model to summarize old messages and replaces them with that summary.
- **AGENTS.md Merging**: Inspected `codex-rs/core/src/agents_md.rs`. The harness finds global rules (`~/.codex/AGENTS.md`) and project rules (`<project>/AGENTS.md`), merging them from top to bottom into the starting prompt.
