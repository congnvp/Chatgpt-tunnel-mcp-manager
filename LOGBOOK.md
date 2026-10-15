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

### CTMM-20261016-005 — PM DEPENDENCY TRIAGE AND TUNNEL PREFLIGHT HANDOFF
- Time: 2026-10-16 04:02 Asia/Bangkok; actor: ChatGPT, Integration/PM, using authorized GitHub/Ubun connectors.
- Action: read current USES protocol, Project-State, paired logbooks, V1 milestone/task plan, ADR README and workstream prompts; inspect Git branch/PR/issue surface and Ubuntu project path. Build proposed task dependency DAG and assign one smallest documentation-only READY task, CTMM-TASK-P00-01A, to workstream Tunnel/CHAT-02 (handoff through Obsidian dossier, not a new chat or an active execution). No implementation/architecture ADR acceptance.
- Git paths changed: `LOGBOOK.md` only.
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Project-State.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Planning/Task-Dependencies.md`.
- Requirements/tasks: V1-R02, V1-R03, V1-R09, Q04, Q05, Q08; parent CTMM-TASK-P00-01; subtask CTMM-TASK-P00-01A (READY_ASSIGNED, NOT STARTED). All technology ADRs still OPEN.
- Dependencies: independent P00 desk research -> separate core/tunnel/GUI ADR and user approval gates -> P01 core -> P02 isolated tunnel+mock MCP -> parallel P03 manager MCP / P04 routing -> P05 registry -> P06 optional UI and P07 Ubuntu operations -> P08 Windows; cutover only by separate authorization. Canonical DAG, acceptance/negative checks, owner-role and rollback recorded in Obsidian Planning/Task-Dependencies.md.
- Preflight: last both logbooks CTMM-20261016-004; Git main head d4b5d308e083d044ee3f628e75ade4802e570817; one visible main branch, no open PR/issue returned; planned Ubuntu source path NOT_FOUND; no workstream update files found in project Obsidian tree. No observed branch or event collision; Git main may change, recheck prior to subsequent writes.
- Conflict/risk register: acceptance gates and user confirmation for tech ADR pending; account/org tunnel entitlement UNKNOWN; runtime credentials not provisioned for test; public repo permits NO secrets; no active tunnel/proxy/service touched. Documentation review/readback only, no runtime/security/permission verification, no compilation/tests (NOT RUN).
- Git commit: this commit.
- Status: PM plan PROPOSED, task delegation RECORDED ONLY; project Intake/Discovery, NOT READY FOR IMPLEMENTATION. Dual-log mirror to verify after Obsidian write.
- Next: Tunnel/CHAT-02 researches official public tunnel eligibility/permissions in a one-page source-backed checklist, recording VERIFIED/UNKNOWN/BLOCKED with citation and date; no login, credentials or tunnel mutation. PM rechecks event/branch/acceptance before next task.


### CTMM-20261016-010 — REGISTRY / SETTINGS DISCOVERY CONTRACT (PROPOSED)
- Time: 2026-10-16 04:04 Asia/Bangkok; actor: ChatGPT, Registry/Settings workstream, via authorized GitHub/Ubun connectors.
- Scope/action: Wrote new clean-room draft defining per-server profiles, versioned desired-state contract, transport-specific validation, secret references, controlled enable/disable, sanitized portable import/export, transactional rollback, audit, read-only/local-editable/AI-proposed field matrix, optional MCP Apps UI, 18 negative/positive test cases, and dependency/ADR gates. No implementation or accepted ADR.
- Git paths changed: `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`, `LOGBOOK.md`. Obsidian paired paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Architecture/Registry-Settings-Proposal.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Requirements/Tasks: V1-R04/R06/R07/R08/R09; P05; proposed RS-D01..RS-D04. D-ARCH-02/04/06 remain OPEN. No product code.
- Tests/evidence: official MCP server and MCP Apps reference documents inspected; existing V1 scope/plan/logbooks read; documentation readback checked. Schema validation, tests RS-01..RS-18, runtime, security and ChatGPT E2E: NOT RUN. No secrets captured or service/tunnel/proxy touched.
- Git commit: document commit `242efb27e2df9efe380b535a9fcb780a7b81c925`; paired Git-logbook commit is this commit.
- Status: PROPOSED DOCUMENT ONLY; pending Obsidian mirror/readback; product NOT IMPLEMENTED.
- Next: paired Obsidian note/event, verify both logs and draft, route dependency decisions to Requirements/Architecture workstream before P05 implementation.

### CTMM-20261016-011 — GUI/TRAY DISCOVERY COMPARISON AND TEST GATES
- Time: 2026-10-16 04:09:54 Asia/Bangkok; actor: ChatGPT GUI/Tray workstream via GitHub + Ubun.
- Scope: official-doc comparison Slint/Tauri/GTK4/Qt6/egui/iced, status/tray UX, headless IPC contract, negative acceptance matrix and rollback; NO ADR accepted, code or runtime change.
- Git paths: `docs/research/GUI-TRAY-DISCOVERY-2026-10-16.md`, `LOGBOOK.md`.
- Obsidian paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Research/GUI-Tray-Discovery-2026-10-16.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Requirements/ADR: V1-R07/R08/R09, Q03/Q06/Q07/Q08, D-ARCH-02/05/06/07 OPEN; P00/P06.
- Tests/evidence: current vendor/API docs URLs in research note; no GUI build/test, service or tunnel E2E (NOT RUN). Paired logs consistent through previous entry before this write.
- Status: RESEARCH / PROPOSED; Rust service unaffected; production NOT IMPLEMENTED.
- Git commit: this commit.
- Next: obtain accepted GUI+IPC ADR and isolate mock snapshot tests; no importing old C PoC.

### CTMM-20261016-012 — PROXY PROTOCOL RESEARCH AND FAKE TEST MATRIX
- Time: 2026-10-16 04:10:43 Asia/Bangkok; actor: ChatGPT / Proxy workstream through authorized GitHub/Ubun connectors.
- Action: Source-backed MCP stable 2026-07-28 and handshake-era survey; propose version/capability negotiation, namespace/collision, tools/resources/prompts routing, authentication boundary, stdio supervision, timeout/cancellation/error contract, six fake fixture types and PX-01..PX-30 test matrix. No ADR approved.
- Git paths: `docs/research/PROXY-INTEROP-AND-FAKE-TEST-MATRIX.md`, `LOGBOOK.md`.
- Obsidian paths paired: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Research/Proxy-Interop-and-Fake-Test-Matrix-2026-10-16.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Related: V1-R03/R04/R05/R06/R08/R09; CTMM-TASK-P00-01E (research candidate); P04 blocked until ADR approval and P02 isolated test.
- Evidence: official MCP releases/specification, official Rust rmcp and OpenAI tunnel references in research doc. Baseline Git/Obsidian logs checked. Documentation research only; all 30 fixtures/cases NOT RUN; no compilation, runtime or network evidence.
- Protection: no API keys/credentials, no existing tunnel/MCP Proxy/service/config changed; public repo contains only synthetic fixture descriptions.
- Git commit: this commit.
- Status: PROPOSED; application NOT IMPLEMENTED, interoperability NOT VERIFIED; dual-log Obsidian mirror to follow.
- Next: PM/Architecture decide protocol/security/SDK and P00 scope; approve ADR before coding and isolated test fixtures.
