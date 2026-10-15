# Git Logbook — CTMM

**Mirror:** Obsidian vault `main` → `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`  
**Policy:** `docs/WORKING-AGREEMENT.md` — one event ID on both sides, append-only.

## 2026-10-16
### CTMM-20261016-001 — PROJECT INITIALIZATION
- Timezone: Asia/Bangkok; date: 2026-10-16; actor: ChatGPT via authorized connectors.
- Action: Tạo hồ sơ dự án mới hoàn toàn, thiết lập repo/tài liệu nền, quy định logbook song song; không thực thi app/tunnel/proxy.
- Git paths: `README.md`, `docs/PROJECT-CHARTER.md`, `docs/ARCHITECTURE-PRINCIPLES.md`, `docs/WORKING-AGREEMENT.md`, `docs/OPEN-QUESTIONS.md`, `docs/CHATGPT-PROJECT-INSTRUCTIONS.md`, `LOGBOOK.md`, `.gitignore`.
- Obsidian paths: folder note, `Logbook.md`, `Project-State.md`, `Requirements/Questions.md`, `Architecture/Principles.md`, `Documentation/ChatGPT-Project-Instructions.md`, `Sessions/2026-10-16.md`.
- Requirements: Rust-first core, Linux/Windows later; Git+Obsidian project; no inherited design.
- Status: docs initialized; runtime `NOT IMPLEMENTED`; tests: documentation/readback only; no runtime tests.
- git_commit: this commit (bootstrap `fa1bcd6e10a6f6e20520bef8464229aecbf0e4c3`).
- Next: verify mirrored Obsidian entry and repo contents; start fresh Discovery in ChatGPT Project.


### CTMM-20261016-002 — BASELINE READBACK VERIFICATION
- Timezone: Asia/Bangkok; date: 2026-10-16; actor: ChatGPT via authorized connectors.
- Action: Read back the initial files on both sides and confirm project identity, instructions, mirror event ID and repository/Obsidian paths; mark documentation initialization verified (not app/runtime).
- Git paths: `LOGBOOK.md` (verification event); inspected `docs/WORKING-AGREEMENT.md`, `docs/CHATGPT-PROJECT-INSTRUCTIONS.md`.
- Obsidian paths: `Logbook.md`, `Project-State.md`; inspected folder note and Project Instructions.
- Evidence: Git fetch_file and Obsidian obsidian_read_note succeeded; event CTMM-20261016-001 present in both logs; no compilation, runtime/service or integration tests.
- Status: documentation baseline `VERIFIED`; software `NOT IMPLEMENTED/NOT TESTED`.
- git_commit: this commit.
- Next: new discovery sessions in ChatGPT Project; maintain paired logs.
