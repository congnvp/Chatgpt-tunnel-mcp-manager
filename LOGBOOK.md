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

### CTMM-20261016-020 — CONFIRMED DISCOVERY CHOICES AND PROPOSED ADR PACK
- Time: 2026-10-16 04:18 Asia/Bangkok; actor: ChatGPT, Requirements/Architecture workstream using authorized GitHub/Ubun connectors.
- Scope/action: Received user answers Q03A/Q04A/Q05A/Q06A/Q07A/Q08A; CONFIRMED product requirements (system-boot service before login; pinned verified tunnel with approved upgrade; AI propose + human approval per mutation; GUI/tray Ubuntu a V1 release MUST; one aggregated ChatGPT-facing endpoint; isolated E2E only after Discovery plus separate authorization). Analyzed conflict with early GUI-optional release map, boot secret-store, multi-MCP and 2026-07-28 protocol boundary. Drafted seven ADR candidates as PROPOSED, no implementation technology accepted and no runtime access granted.
- Git paths changed: `docs/requirements/V1-Q03-Q08-CONFIRMED.md`, `docs/adr/PROPOSED-ADR-CTMM-001-007.md`, `LOGBOOK.md`.
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Requirements/Discovery-Q03-Q08-2026-10-16.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Requirements/Questions.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Project-State.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Related: V1-R01..R09; Q03..Q08 product choices only; ADR-CTMM-001..007 PROPOSED; CTMM-TASK-P00-01C; Ubuntu V1 release now gates on both P06 GUI and P07 operations; early headless P01–P05 do NOT depend on GUI.
- Pre-state/evidence: inspected current Git/Obsidian logbooks through CTMM-20261016-012, Git research for proxy/GUI/registry, USES project rules and official OpenAI Secure MCP Tunnel + MCP 2026-07-28 docs. Verified pre-existing source notes; technical ADRs remain OPEN. Requirements and ADR proposal document Git commits: `b08766f6c4deabfd3bd4f24eb4eb261948498461`, `a77149994072ce0ded773e2de0395b65cc80384d`.
- Tests: documentation reasoning and file-read checks only; build/service, account entitlement, tunnel/mock E2E, proxy, GUI, auth, security and rollback tests NOT RUN. No credentials recorded or active tunnel/proxy/service changed.
- Status: Q03A..Q08A REQUIREMENTS CONFIRMED, details/architecture OPEN; ADRs PROPOSED NOT ACCEPTED; product NOT IMPLEMENTED; P01+ BLOCKED. Git commit: this commit.
- Pending/next: mirror exact event and resulting Git SHA to Obsidian, read back both logbooks and notes, review service identity/secret trust, user approve ADR alternatives and separately approve isolated tests before execution.

### CTMM-20261016-021 — TUNNEL ACCESS PREFLIGHT AND PM RELEASE-GATE RECONCILIATION
- Time: 2026-10-16 04:21 Asia/Bangkok. Actor: ChatGPT, Integration/PM, continuing assigned CTMM-TASK-P00-01A (documentation-only Tunnel preflight).
- Scope/action: Re-read both logbooks, Git HEAD, USES, PM task DAG and new CTMM-20261016-020 confirmed Q03A..Q08A + proposed ADR pack; researched official public Secure MCP Tunnel, current custom MCP/ChatGPT access docs; documented org-level Read+Manage vs Read+Use and separate ChatGPT workspace permissions, org/workspace associations, outbound HTTPS and private MCP reachability, isolated E2E negative preconditions. Reconcile PM DAG: Ubuntu GUI/tray REQUIRED for V1 RELEASE after Q06A, but service/early core independent from graphical login. No ADR accepted or code.
- Git paths: `docs/research/TUNNEL-ACCESS-PREFLIGHT-2026-10-16.md`, `LOGBOOK.md`.
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Research/Tunnel-Access-Preflight-2026-10-16.md`, `Planning/Task-Dependencies.md`, `Project-State.md`, `Logbook.md` (relative paths within same project folder).
- Requirements/task: V1-R02/R03/R07/R08/R09; Q03A/Q04A/Q05A/Q06A/Q07A/Q08A; CTMM-TASK-P00-01A; ADR-CTMM-001..007 PROPOSED. No new user selection, permission or approval.
- Pre-state: Git main latest synchronized event CTMM-20261016-020 at `0c5df2e9f7aeb108c0333cb0a46acce1f597b924`, paired Obsidian event 020 present after readback. Earlier PM graph (event 005) stale for V1 GUI release gate; update addendum. Existing Tunnel preflight research file did not exist. Source document commit `5bc3bbba0759a3029054758ee3df1c66f895f1b2`.
- Evidence: official references https://developers.openai.com/api/docs/guides/secure-mcp-tunnels , https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt and https://developers.openai.com/api/docs/guides/tools-connectors-mcp (inspected 2026-10-16). Document checks only. Live account entitlement and actual workspace permission UNKNOWN; temporary credentials NOT PROVISIONED. Rust, tunnel, mock MCP, GUI, security and E2E tests NOT RUN; no secret, binaries, configuration or active service/tunnel/proxy modified.
- Acceptance: P00-01A public-document research complete when Git/Obsidian paired readback verified; status RESEARCH_DOCUMENTED, not account VERIFIED, not test PASS. Parent P00-01 and P01+ implementation gate remain OPEN/BLOCKED. Rollback: correction event in both append-only logbooks; do not alter production.
- git_commit: this commit (Git research doc earlier committed separately as above).
- Next: pair Obsidian note, DAG amendment, Project-State and logbook with SHA of this commit; verify readback. P00-01B official tunnel binary provenance/license remains READY, next unassigned candidate. ADR approvals and isolated execution authorization still required.

### CTMM-20261016-022 — RUST OTUNNEL VS OFFICIAL V0.0.16 SOURCE AUDIT
- Time: 2026-10-16 04:26:42 Asia/Bangkok; actor: ChatGPT / Tunnel workstream; scope: CTMM-TASK-P00-01B, source comparison only.
- Action: inspected official openai/tunnel-client v0.0.16 release/diff/docs and zetaloop/otunnel v0.2.0 Rust source/release. Identified optional v0.0.16 resp_timing/forward_trace_context gaps, strict profile schema, embed-health feature boundary, release provenance differences and README semver-pin mismatch. Prepared ADR-CTMM-001 addendum PROPOSED and eight test gates; no ADR accepted.
- Git paths: `docs/research/OTUNNEL-V020-VS-OFFICIAL-V0016-2026-10-16.md`, `LOGBOOK.md`.
- Obsidian paired paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Research/Rust-Tunnel-Compatibility-2026-10-16.md`, `Logbook.md`, `Project-State.md`.
- Requirements/tasks: V1-R01/R02/R03/R08/R09, Q03A/Q04A/Q08A, CTMM-TASK-P00-01B, ADR-CTMM-001 (candidate addendum).
- Evidence/tests: official immutable release 0.0.16 and repo diff vs 0.0.15; otunnel v0.2.0 tagged source + release metadata; static audit only. Build, cargo tests, signature/artifact-byte verification, account permission, tunnel or ChatGPT E2E: NOT RUN.
- Security: no key, binary download, external program run, active tunnel/proxy/service or existing runtime modified.
- Git commit: document commit; paired Git LOGBOOK commit is this commit.
- Status: DOC RESEARCH ONLY / Rust E2E compatibility UNVERIFIED / ADR PROPOSED; paired Obsidian readback pending.
- Next: independent review 01B findings; decide the ADR after sandbox compatibility tests are approved. No P01/P02 implementation before approvals.


### CTMM-20261016-030 — SELF-MANAGEMENT MCP CONTRACT AND NEGATIVE TEST PLAN
- Time: 2026-10-16 04:24 Asia/Bangkok; actor: ChatGPT / Self-management MCP workstream via GitHub and Obsidian.
- Scope/action: inspect baseline+confirmed Q03A/Q05A/Q07A, proposed ADR-CTMM-002/004/005/006, Registry/Settings and official MCP 2026-07-28/tunnel documentation; propose P03 read-only tools, P05 staged propose/local-approve/apply, formal JSON Schema, privacy/permission matrix, offline fallback, fault cases SM-01..24. No ADR accepted, application code or service action.
- Git paths: `docs/architecture/SELF-MANAGEMENT-MCP-PROPOSAL.md`, `docs/contracts/self-management-mcp-v1.schema.json`, `LOGBOOK.md`.
- Obsidian paired paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Architecture/Self-Management-MCP-Proposal.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Requirements/tasks: V1-R04/R06/R08/R09; Q03A/Q05A/Q07A; P03/P05; SM-D01..04 OPEN; ADR-CTMM-004/006 PROPOSED; no approval or E2E execution.
- Evidence: official spec/docs read and schema structure checked prior to commit; JSON Schema conformance suites, Rust build, mock E2E, authz/rollback/negative cases NOT RUN; no active tunnel/proxy/service and no secrets touched.
- Status: SYNC_PENDING — proposed documentation committed and Obsidian architecture note created, but the mandatory Obsidian `Logbook.md` event append was blocked by intermittent connector initialization/safety failures; Git docs and JSON Schema readback succeeded. Product NOT IMPLEMENTED. Do NOT mark dual-log VERIFIED until the paired Obsidian event is appended and both logs are reread.
- git_commit: this commit.
- Next: mirror Obsidian note/event with actual SHA, re-read both logbooks; seek ADR-CTMM-004/006 selection before implementation.


### CTMM-20261016-040 — REGISTRY V1 STRUCTURAL SCHEMA AND EVENT-ID COLLISION CORRECTION
- Time: 2026-10-16T04:31:55+07:00; actor: ChatGPT / Registry-Settings workstream via authorized GitHub+Ubun connectors.
- Scope/action: Continued RS-D01. Authored *proposed only* JSON Schema Draft 2020-12 for desired Registry state and inert disabled stdio+HTTP example; documented structural-vs-trusted-semantic validation, approval/revision/generation contract, import/export secret isolation and Q03A/Q05A/Q06A/Q07A/Q08A alignment. No runtime code and no ADR acceptance.
- Event-ID conflict/correction: initial Registry source commits inadvertently used CTMM-20261016-030 in commit titles while concurrent Self-management MCP workstream independently reserved/wrote the official CTMM-20261016-030 log entry. That event belongs to Self-management and was NOT modified. This Registry change-set is exclusively CTMM-20261016-040; original immutable Git commit messages remain as provenance, with corrected document header and explicit audit trail. No force push or history rewrite.
- Git paths changed: `docs/architecture/schemas/registry-v1.proposed.schema.json`, `docs/architecture/examples/registry-v1.inert-example.json`, `docs/architecture/REGISTRY-V1-SCHEMA-VALIDATION.md`, `LOGBOOK.md`.
- Obsidian paired paths: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Architecture/Registry-V1-Schema-Validation.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Requirements/tasks: V1-R04/R06/R07/R08/R09, RS-D01; Q03A..Q08A confirmed product requirements; ADR-CTMM-002/004/005/006/007 still PROPOSED.
- Evidence: `Draft202012Validator.check_schema` succeeded against exact Git schema; evaluated exact Git example and 16 mutated negatives on Ubuntu, output `SCHEMA_CHECK_VALID; CASES=17; MATCH=17; MISMATCH=0`. This verifies *only structural schema cases*; trusted semantic checks V-01..V-14, RS-01..RS-18, authorization, secret store, Rust, tunnel and E2E NOT RUN. No credentials, binaries, services, active tunnel or proxy altered.
- Git commits: schema `8d738779354703de98a539263167dd607f467824`; example `101a1dbd0ab6bb7949cd40776b01d3d8eeb38952`; original validation doc `8187db02d5337de9bee86884d9e9fa80b1fb6f87`; corrective doc `9694e2b1138026e659cc4db1d213667f8787fa29`; Git logbook: this commit.
- Status: STRUCTURAL DOCUMENT SMOKE VERIFIED; Registry contract PROPOSED, NOT ACCEPTED/IMPLEMENTED; Obsidian paired documentation sync pending readback.
- Next: mirror documentation and event to Obsidian with same CTMM-20261016-040 and SHA; verify both Git/Obsidian logs. Then RS-D02 permission matrix and RS-D03 sanitized export envelope pending ADR-CTMM-006 security decisions.

### CTMM-20261016-050 — PM DUAL-LOG RECONCILIATION AND NEXT SECURITY DISCOVERY HANDOFF
- Time: 2026-10-16T04:35:39+07:00 Asia/Bangkok; actor: ChatGPT / Integration-PM, authorized GitHub/Ubun read-and-document scope.
- Action: re-read USES, project state, both logbooks, task graph and Git HEAD; audit concurrent events 022/030/040. Confirm 022 Tunnel official v0.0.16 vs Rust otunnel v0.2.0 source audit is documentation only; no parity or E2E PASS. Confirm event-ID collision: Registry changes originally had erroneous 030 commit titles; Registry's own append-only corrective event 040 reattributes immutable commits to 040. Event 030 in Git remains exclusively Self-management MCP. PM appended the missing Obsidian 030 mirror *without editing existing entries*, then read back Obsidian log with events 030 and 040. No rewrite, force push or architecture decision.
- Git paths changed: `LOGBOOK.md` (this PM event).
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`, `Project-State.md`, `Planning/Task-Dependencies.md`.
- Related: event 022 (Tunnel research), event 030 (Self-management MCP), event 040 (Registry corrective record), CTMM-TASK-P00-01B research snapshot, CTMM-TASK-P00-01D1 handoff, Q03A/Q05A/Q06A/Q07A/Q08A, ADR-CTMM-001..007 still PROPOSED.
- Baseline evidence: Git main HEAD at `7b0ac6f6c05b6751e9fccda47f7b43b17d7afcd6` before PM write. Git/Obsidian each contain exactly one heading for event 030 (Self-management) and event 040 (Registry correction); Obsidian event 030 late mirror was applied with read-note etag guard and re-read. No open PR/issues and only main branch in previous inspection; not a lock against future parallel writes.
- Decisions: no technology ADR acceptance or runtime authorization. Select one new smallest READY document-only subtask `CTMM-TASK-P00-01D1` (security: map boot-before-login identity/secret-store and AI-approval trust boundary), assign role QA/Security CHAT-08 via written handoff, status READY_ASSIGNED/NOT STARTED. Related P00-01D broader threat model OPEN. Acceptance: model actors/assets/trust boundaries, compare boot-safe host secret options with pros/cons, least privilege, human approval binding and server-side fail-closed, positive/negative test design including keyring unavailable, stale revision, replay, and rollback; no secrets. Failure: do not assume desktop keyring works before login; no model-self-approval or exposed public admin. All ADRs remain PROPOSED pending user's choice.
- Tests/evidence: documentation/source/git history/readback verification only. Registry event 040 reports isolated Python schema structural smoke 17/17; PM did NOT rerun this test and does not promote semantic/runtime validation. CTMM build, real boot, compiler, privileged auth, tunnel, ChatGPT E2E and rollback tests NOT RUN. No service/tunnel/proxy changes performed in this PM session; no credentials handled.
- Status: PM planning PROPOSED; correction pairs recognized; implement P01+ BLOCKED pending accepted ADR and separate isolated test authorization. Git commit: this commit.
- Next: mirror 050 with actual SHA to Obsidian and read back; delegated Security/CHAT-08 prepares only P00-01D1 proposal; then PM reviews blocker ADR-CTMM-003/004/006 with user. Do not start P01/P02.

### CTMM-20261016-060 — RECOVERY READBACK AND ADR-003/004/006 SECURITY GATE REVIEW
- Time: 2026-10-16 04:35 Asia/Bangkok; actor: ChatGPT / Requirements/Architecture via authorized GitHub and Obsidian connectors.
- Preflight/recovery: reread both Git/Obsidian logbooks and original artifacts. Self-management event CTMM-20261016-030 now has the late Obsidian mirror and matches the Git-only scope; Registry's independent misused 030 commit titles were corrected in event CTMM-20261016-040 with an append-only provenance trail. Historical Git SYNC_PENDING at 030 reflects state when it was written, not current status; this event verifies the later recovery checkpoint without modifying previous history.
- Action: created detailed PROPOSED review for ADR-CTMM-003 boot service / boot credential availability, ADR-CTMM-004 local IPC/authenticated principal boundaries, and ADR-CTMM-006 registry/secret/audit/per-change approval. Compared alternatives, threat boundaries, security conditions, positive/negative/rollback test proposals, and P01/P02/P05 gates. No ADR accepted or technology implemented.
- Git paths changed: `docs/architecture/ADR-003-004-006-SECURITY-GATE-REVIEW.md`, `LOGBOOK.md`.
- Obsidian paths in paired change-set: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Architecture/ADR-003-004-006-Security-Gate-Review.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Project-State.md`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.
- Related: V1-R01/R04/R06/R08/R09; Q03A/Q04A/Q05A/Q06A/Q07A/Q08A confirmed, other detail choices OPEN; ADR-CTMM-003/004/006 PROPOSED; CTMM-TASK-P00-01D and P00 gates. Project 030 and 040 disjoint workstreams verified at readback.
- Evidence: CTMM event Git/Obsidian record comparisons; canonical docs `SELF-MANAGEMENT-MCP-PROPOSAL.md`, `REGISTRY-V1-SCHEMA-VALIDATION.md`; official systemd.io credentials and man7 Unix socket peer-credential documentation checked. Document/source readback only; service boot, OS identity, secret provider, IPC authorization, mutation, Rust build, negative cases, tunnel and E2E tests **NOT RUN**. Registry 040 separately records `CASE=17,MATCH=17` structural-only smoke; not a security PASS.
- Safety: no credentials, secret provider, service/tunnel/proxy process, port, systemd unit, installer or production configuration changed; no elevated actions, no product code.
- Git document commit: `05aae9bab249e1b18a207b7e6e001e1e819c4fa5`; Git logbook commit: this commit.
- Status: cross-workstream event collision clarified; dual logbook readback/checkpoint needs mirrored Obsidian event and final verification; ADRs PROPOSED; code/runtime NOT IMPLEMENTED/NOT TESTED; P01+ gate still BLOCKED.
- Next: append paired Obsidian note/log/state with actual Git SHA and read both sides; submit ADR-003/004/006 alternatives to operator for explicit choice, not automatic approval. Isolated E2E requires separate execution authorization.

