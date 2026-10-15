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

### CTMM-20261016-003 — UBUNTU TRAY PANEL DISCOVERY SPIKE
- Time: 2026-10-16 03:03 Asia/Bangkok; actor: ChatGPT via authorized Ubun and GitHub connectors.
- Scope/action: User-authorized isolated GNOME tray/AppIndicator + GTK3 floating panel PoC, built to AppImage on Ubuntu 26.04.1 LTS (x86_64, Wayland). This is a discovery spike only; not Rust core, accepted GUI ADR, or production implementation.
- Git paths changed: `LOGBOOK.md` (this event only; experimental code deliberately remains outside main project repository).
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`, `Project-State.md`, `Research/Ubuntu-Tray-AppImage-Spike-2026-10-16.md` (paths below project root where abbreviated).
- Ubuntu PoC paths: `/home/congnvp-su/Projects/CTMM-Tray-Panel-Spike/src/main.c`, `build.sh`, `README.md`, `build/ctmm-tray-demo`, `build/CTMM-Tray-Demo.AppDir/`, `build/live-demo.log`, `tools/appimagetool-x86_64.AppImage`, `dist/CTMM-Tray-Demo-x86_64.AppImage`; distributable copy `/home/congnvp-su/Applications/CTMM-Tray-Demo-x86_64.AppImage`.
- Requirement/task: Discovery on Ubuntu tray UX and AppImage feasibility; Q01-Q08 and product GUI/packaging decisions remain OPEN. Rust core requirement unchanged.
- Evidence/tests: GCC `-Wall -Wextra -Werror` build successful; native GTK+indicator smoke exit=0; packaged AppImage smoke exit=0; GNOME Ubuntu AppIndicators extension ACTIVE; `org.kde.StatusNotifierWatcher.RegisteredStatusNotifierItems` contains `ctmm_tray_spike`; live process PID 16064 at observation. AppImage 1116664 bytes, SHA256 `778067cbab2065a8ca23daa7bed62619f98cdbf4d85a25bd7b33c9a1a067e91c`. Menu clicking and rich panel visual placement: NOT YET MANUALLY TESTED. Cross-distro portability: NOT TESTED.
- Security/limits: no tunnel, MCP Proxy, daemon, service or system configuration changed; no package installation. Uses deprecated GTK Ayatana interface, host GTK3; GNOME native menu shown on icon click, separate GTK panel selected from menu; Wayland does not guarantee arbitrary window coordinates.
- Status: experimental build/registration smoke VERIFIED; production gate remains DISCOVERY, NOT READY FOR IMPLEMENTATION.
- git_commit: this commit.
- Next: paired Obsidian event + research note + state readback; continue Q01–Q08 and evaluate modern SNI/API options before ADR.

### CTMM-20261016-004 — V1 REQUIREMENTS AND REPOSITORY PLANNING BASELINE
- Time: 2026-10-16 03:34 Asia/Bangkok; actor: ChatGPT via GitHub/Ubun. Requested and authorized: document-only repository tree, module rules, user-stated V1 product baseline, workflow prompts and incremental execution plan.
- Git paths: `LOGBOOK.md`; `docs/requirements/V1-SCOPE.md`; `docs/architecture/REPOSITORY-LAYOUT-PROPOSAL.md`; `docs/engineering/MODULE-RULES.md`; `docs/plans/V1-MILESTONES.md`; `docs/workflows/CHAT-PROMPTS.md`; `docs/workflows/RECOVERY-AND-LOGGING.md`; `docs/adr/README.md`; `crates/README.md`; `apps/README.md`; `tests/README.md`; `fixtures/README.md`; `packaging/README.md`.
- Obsidian paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`, `Project-State.md`, `Requirements/Questions.md`, `Requirements/V1-Baseline.md`, `Architecture/Repository-Layout-Proposal.md`, `Planning/V1-Milestones.md`, `Documentation/Chat-Workstreams-V1.md`.
- Requirement/ADR/task: V1-R01..V1-R09; Discovery Q01..Q08 updated with new inputs; all implementation-related ADRs still OPEN; next P00/CTMM-TASK-P00-01 research/gate.
- Evidence: checked official OpenAI Secure MCP Tunnel, Rust SDK, Cargo layout, MCP Apps and OpenAI MCP server documents; Git and Obsidian initial readbacks; documentation consistency/readback only. Rust build/integration/service/tunnel tests NOT RUN; no executable created.
- Protection: no credentials provided, recorded or requested; no changes to existing tunnel, MCP Proxy, Ubuntu service or previous tray PoC. Docs and placeholder README files only; code folders reserved for post-ADR gate.
- Decision: Rust core, Ubuntu first then Windows, headless service plus optional unified GUI, self-management MCP, compatibility-focused proxy, progressive visible test slices constitute new user requirements; specific GUI, SDK, IPC, service mechanisms and distribution remain unapproved.
- Status: doc plan PROPOSED, product NOT IMPLEMENTED; git_commit: this commit.
- Pending/next: mirror notes and current SHA to Obsidian, readback paired logs, confirm blocking ADRs and begin P00 Discovery prior to Rust coding.
