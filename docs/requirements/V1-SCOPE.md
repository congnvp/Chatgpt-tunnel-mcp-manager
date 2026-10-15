# CTMM V1 — scope baseline (user intent; technical details OPEN)
Status: REQUIREMENTS BASELINE, not implemented. Event: CTMM-20261016-004.

## Product objective
A new Rust-based application for managing MCP servers, using the supported OpenAI Secure MCP Tunnel path. The first target is Ubuntu, then Windows. Replaces the operational role of the current tunnel and MCP proxy **only after independently verified and explicit cutover approval**. Never modify or stop the existing stack during planning/isolated validation.

## Functional requirements
- V1-R01: persistent background service on Ubuntu (later Windows); if configured, authorized, and networking is available, start connection automatically at boot and retry with bounded exponential backoff/jitter on failure; report OFFLINE/CONNECTING/READY/DEGRADED precisely. No claim of instantaneous successful network connection.
- V1-R02: evaluate the official, latest *verified compatible* OpenAI `tunnel-client` release for Secure MCP Tunnel; version pin + checksum/provenance + compatibility evidence + controlled upgrade/rollback; do not reinvent undocumented wire protocol by default.
- V1-R03: expose a minimal mock MCP server first, validate initialize, tools/list, tools/call through an isolated tunnel in ChatGPT. User will arrange temporary credentials and tunnel permissions **later**.
- V1-R04: built-in management MCP interface for read-only tunnel/proxy/service status, sanitized diagnostics, test results. Write tools (adding/enabling/disabling MCP servers) only behind authenticated, authorized, auditable and confirmation-gated operations. Local read-only diagnostics survive upstream failure.
- V1-R05: MCP proxy/router supports multiple managed MCP servers; discover and negotiate protocol versions/capabilities; explicit compatibility matrix for `stdio`, Streamable HTTP, and legacy SSE if needed; never claim universal support without fixtures/tests.
- V1-R06: basic tunnel/proxy configuration with validation, persistence and restart/rollback; no plaintext secrets in repo, Obsidian, chat or logs. Separate per-server profiles, enable/disable, connection diagnostics, sanitized errors.
- V1-R07: consistent desktop UX on Ubuntu and Windows, tray state + settings where feasible. Optional ChatGPT-side form/dashboard using MCP Apps standard; underlying tools work in headless clients without UI.
- V1-R08: core/service starts and works without graphical session or DISPLAY; GUI is a separate optional process. No startup errors merely from absence of GUI.
- V1-R09: every meaningful research/session/action/decision/operation creates a durable trace with task ID, timestamp Asia/Bangkok, actor, input classification (no secret), outcome, evidence, status and next action; two logbooks receive same CTMM event for each change-set. Implement runtime operation journal separately in a later task.

## Boundaries and safety
- Service is authoritative; panel is a read-only/status client initially, never controls process lifecycle implicitly.
- Tunnel/proxy/managed MCP need non-circular dependency: local status remains queryable if tunnel fails.
- Automatic reconnect is allowed only for the **new isolated profile** after approval and provision. Do not connect a live service automatically merely because binary exists.
- Do not write tokens to GitHub, Obsidian, ChatGPT Project or unredacted debug logs. Credentials must be provisioned into a protected host secret store or restricted temporary runtime secret channel; log only secret reference identifiers. After test: revoke/rotate and securely remove the temporary local material, with auditable confirmation.
- Publishing/packaging, updater, service privileges, daemon/IPC implementation, Rust MCP SDK, tunnel-client redistribution rights, UI framework, and auth model remain OPEN until ADR.

## V1 acceptance intent
Successful V1 must demonstrate a fresh Ubuntu boot with no GUI, connection to an authorized isolated tunnel, mock MCP invocation from ChatGPT, authenticated management MCP read and approved write, multi-server proxy interoperability matrix, basic config persistence, safe failure/retry/rollback, same-state GUI on a graphical session, and explicit unchanged-state check for legacy production stack.

## Authoritative research
- https://developers.openai.com/api/docs/guides/secure-mcp-tunnels
- https://developers.openai.com/plugins/build/mcp-server
- https://developers.openai.com/plugins/build/chatgpt-ui
- https://rust.sdk.modelcontextprotocol.io/
