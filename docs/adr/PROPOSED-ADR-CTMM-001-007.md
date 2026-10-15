# CTMM — Architecture Decision Pack ADR-CTMM-001..007
Status: ALL `PROPOSED`, NONE ACCEPTED · CTMM-20261016-020 · 2026-10-16 Asia/Bangkok
Owner: Requirements/Architecture. This is an ADR proposal pack to split into individual reviewed ADR files if/when operator approves. No technology selected by the six user answers. No implementation, service modifications or credentials.

## Common constraints and approval checklist
- New clean-room Rust core; Ubuntu first; later Windows. System boot service before login; GUI/tray is V1 release MUST but not runtime dependency. One ChatGPT MCP endpoint aggregates multiple MCP servers. Pinned tunnel versions, approved upgrades, model proposes and operator approves each mutation. Isolated E2E only after explicit separate permission.
- For EACH ADR: operator explicitly selects alternative and scope; cross-check licensing/security, platform contract, tests and rollback; record `ACCEPTED` date and paired logbook event, or keep `PROPOSED`. None is implicitly chosen below.
- Research inputs: `docs/requirements/V1-Q03-Q08-CONFIRMED.md`, `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`, `docs/research/PROXY-INTEROP-AND-FAKE-TEST-MATRIX.md`, `docs/research/GUI-TRAY-DISCOVERY-2026-10-16.md`; official sources linked at end.

### ADR-CTMM-001 — Official tunnel integration and delivery
- Context: R02/R03; Q04A/Q08A; bind isolated private MCP into ChatGPT without changing currently running stack.
- A: supervise independently released/pinned official `tunnel-client` process (upgrade/rollback independently, supervision and lifecycle handling needed). B: embed official runtime by supported API/FFI if actually provided (tight coupling/cross-language portability). C: reimplement tunnel transport in Rust (control, undocumented compatibility/security maintenance).
- Unresolved choice: `OPEN`. Candidate to research first: A, contingent on redistribution rights, provenance, supported profile/config and official compatibility.
- Security/OS: outbound-only tunnel-client account with least privilege; do not expose local admin UI; verify artifact provenance and secret handling at boot; Windows distribution scope OPEN.
- Positive test: pinned artifact doctor/health and ChatGPT isolated fake tool call. Negative test: bad checksum, unsupported version, wrong workspace entitlement, disconnected network, profile isolation.
- Rollback: stop only test tunnel process, restore previously pinned binary/profile, revoke temporary credential; never modify existing tunnel.
- Prereqs: tunnel access research (P00-01A/B), ADR-CTMM-003/004/006, separate operator test permission.

### ADR-CTMM-002 — MCP SDK/runtime/protocol compatibility
- Context: R03/R05 and one endpoint Q07A; 2026-07-28 stateless vs older initialize/initialized MCP eras.
- A: pinned upstream Rust `rmcp` with explicit compatible protocol adapters (faster conformance, SDK API/dependency risk). B: custom Rust JSON-RPC+MCP stack (full control, significantly larger audit/conformance burden). C: legacy-only MCP (smaller, insufficient if current clients require new revision).
- Unresolved: exact SDK version, Tokio/runtime, supported transports/revisions, capabilities beyond tool lists/calls, old SSE handling; do not promise full backward compatibility.
- Security/OS: untrusted tool schemas and prompts, request limits, auth boundaries, no unsafe token forwarding, headless-compatible adapters.
- Tests: modern server/discover, legacy initialize, stdio+Streamable HTTP, version mismatch, malformed JSON, cancellation, timeouts, protocol semantic mismatch; adapt PX matrix.
- Rollback: revert isolated dependency pin/adapter and disable unsupported feature route, with truthful degraded state.
- Prereqs: P00 protocol research, ADR-CTMM-005.

### ADR-CTMM-003 — Boot-time Rust service identity and secrets
- Context: R01/R08; Q03A implies starts before user login; desktop keyrings often unavailable before interactive unlock.
- A: isolated `systemd` system unit with dedicated unprivileged service account and boot-accessible protected credentials (reliable boot, more careful ACL/secret provision). B: `systemd --user` service with linger (per-user isolation, needs proof of headless boot and secret availability). C: privileged root process (convenient, unacceptable attack surface unless separately justified).
- Unresolved choice: no unit kind/ownership or secret provider selected. Windows Service adapter must not be assumed Linux-equivalent.
- Security: separate auth identity from logged-in GUI user; least privilege, protected secret references, local IPC peer validation, no inherited unrestricted environment.
- Tests: cold boot without login/DISPLAY/network, system recovery, locked keyring/unavailable credential, offline retry and clean shutdown; ensure GUI absence does not block.
- Rollback: disable only new isolated CTMM test unit/profile and restore service-specific config; do not touch old services.
- Prereqs: threat model, ADR-CTMM-004/006 before P01/P07 installation tests.

### ADR-CTMM-004 — Local IPC and authentication boundary
- Context: R04/R06/R08; GUI/tray and AI management MCP read same authoritative service snapshot but have DIFFERENT trust paths.
- A: Unix socket with OS peer credentials + Windows named pipe with SID ACL (strong local scope, different platform adapters). B: loopback HTTP with explicit authenticated transport (reusable clients, extra CSRF/origin/auth surface). C: plain local TCP unauthenticated (not viable for mutation).
- Unresolved: schema/versioning, service-user access matrix, remote/local principal translation, read vs write endpoints and approvals.
- Tests: valid/denied OS principal, forged UI caller, disconnected IPC, stale revision, loopback cross-origin if chosen, GUI crash; local diagnostics available when tunnel down.
- Rollback: disable isolated endpoint and restore prior ACL/policy; service core remains unaffected.

### ADR-CTMM-005 — Multi-MCP aggregation, namespace and routing
- Context: R05/Q07A mandates ONE ChatGPT-facing endpoint exposing multiple independently managed servers.
- A: official tunnel native multichannel if it empirically produces required single endpoint catalog/routing (minimal custom code, less control; NOT YET PROVEN). B: explicit Rust aggregator/router with one outward MCP endpoint (control over namespace/auth/reconciliation, higher complexity). C: multiple independent ChatGPT endpoints (does NOT satisfy Q07A).
- Unresolved: A-vs-B after isolated proof; namespace strategy, tool ID stability, transports, capability translation and resources/prompts MUST coverage.
- Security: per-profile principal filtering, no credential passthrough, stable alias mapping, collision fail closed, bounded child execution and SSRF controls.
- Tests: ChatGPT from one connection discovers/calls echo A + echo B; same tool names do not collide; one backend crash/timeout does not poison sibling; protocol version mismatch cleanly denied; no cross-tenant leak.
- Rollback: revert only isolated routing generation/config, preserving current production stack.
- Prereqs: ADR-CTMM-001/002/004/006, P02 E2E auth.

### ADR-CTMM-006 — Registry, secrets and per-change AI approval
- Context: R04/R06/R09; Q05A confirms model MAY propose but must NEVER self-approve and apply a change.
- A: atomic validated TOML/JSON registry + protected OS secret store (inspectable, simpler, needs concurrency/transaction design). B: SQLite desired-state store + OS secret store (transaction, migration/backup complexity). C: plaintext secrets or model-enforced approval (REJECT: violates security).
- Approval transport alternatives: authenticated local GUI operator confirmation vs separate privileged local CLI vs external identity/approval service; NONE SELECTED. Each accepted change MUST bind actor, scope, canonical diff hash, base revision, expiry and one-time confirmation, then re-authorize atomically at apply.
- Security: no secret values in Git/Obsidian/chat/logs; audit independent from CTMM dual project logbooks; no implicit model authority, default-deny.
- Tests: proposal no side effects; valid approved apply once; replay/stale revision/forged approval/expired token/unauthorized principal rejected; atomic rollback and redaction tested; reference RS matrix.
- Rollback: restore previous isolated registry generation; if apply reconciliation fails, report DEGRADED and preserve evidence; never silently mark READY.
- Prereqs: threat model, service identity, IPC and auth/approval policy.

### ADR-CTMM-007 — Ubuntu GUI/tray and packaging
- Context: R07/R08/Q06A: GUI/tray is Ubuntu V1 release MUST, but service requires no graphical login.
- A: Slint/Rust-native tray (Rust integration, license/distro/manual GUI risks). B: Tauri 2 webview (rich settings, Linux click event limitations and GTK/WebKit dependencies). C: GTK4/gtk-rs (Ubuntu native focus, Windows parity packaging concern). D: Qt6/CXX-Qt or Rust immediate-mode toolkit + separate tray adapter (ecosystem tradeoffs).
- Unresolved: toolkit, license, exact GNOME Wayland click/menu fallback, packaging, per-user tray session startup, multi-monitor behavior, future Windows port.
- Security: GUI is unprivileged authenticated client only; no direct service lifecycle control or secret rendering.
- Tests: manual Ubuntu Wayland icon/menu and launcher, settings/status snapshot vs CLI, close/crash GUI, absent DISPLAY, denied config mutation, DPI/multi-monitor; GUI-01..12. No test passes from old isolated C PoC.
- Rollback: uninstall/close only new GUI test artifact and restore GUI prefs; service stays up.
- Prereqs: service typed state, IPC, approval policy, operator accepted UX expectations.

## Cross-ADR conflict register and gates
1. Q03A boot vs Q06A GUI: GUI separate process; release requires both, runtime must not depend on GUI.
2. Q03A vs Q05A approval: approval is impossible pre-login absent operator; only previously-approved, persisted desired config may auto-connect at boot; pending mutation waits.
3. Q04A vs latest: capture latest available source for *research* only; runtime pinned until explicitly approved upgrade. Never silent latest upgrade.
4. Q07A vs tunnel multichannel: native channel feature requires evidence of aggregated single ChatGPT catalog; avoid premature custom proxy lock-in.
5. 2026-07-28 vs legacy handshake: transparent JSON-RPC relaying alone cannot guarantee semantics; capability matrix selects compatible subset.
6. Q08A isolated permission vs project policy: planning allowed, actual E2E execution blocked until tenant/workspace entitlement and credentials are provided to protected host secret store and approval recorded.
7. Windows parity later, but Ubuntu V1 GUI/tray is a release gate. Existing task DAG marking P06 optional FOR RELEASE is superseded by confirmed Q06A, while P06 is still optional for early headless P01–P05.

## Acceptance/approval status
All choices remain PROPOSED; all execution tests NOT RUN. Acceptance requires explicit option selection per ADR, permission/threat model assessment, positive/negative tests, rollback and paired Git/Obsidian logbook update. P01–P08 remain blocked by respective gates.

Source references:
- https://developers.openai.com/api/docs/guides/secure-mcp-tunnels
- https://blog.modelcontextprotocol.io/posts/2026-07-28/
- https://rust.sdk.modelcontextprotocol.io/
- https://docs.slint.dev/latest/docs/slint/reference/window/systemtrayicon/
- Internal: `docs/research/PROXY-INTEROP-AND-FAKE-TEST-MATRIX.md`, `docs/research/GUI-TRAY-DISCOVERY-2026-10-16.md`, `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`.
