# CTMM V1 — incremental vertical slices
Status: proposed plan, no approved build task. Event: CTMM-20261016-004.
Rule: finish gate and demonstrate a real observable artifact after **every** slice. No changes to running legacy tunnel/proxy/services, including restart; test with separately named isolated profiles/ports/tunnel IDs.

| Gate | Narrow scope / tiny task | Observable output | Passing and failing evidence | Rollback |
|---|---|---|---|---|
| P00 DISCOVERY | Choose MVP, threat model, official tunnel-client distribution, service mode, MCP versions, UI criteria; accept ADRs | requirements/ADR and operation plan | reviewed scope, decisions, dependency/permission check; NO runtime PASS | revert docs only with correction log |
| P01 CORE DIAGNOSTICS | Create minimal Rust state snapshot + fake tunnel/MCP statuses with isolated local read-only API/CLI; no network | `status` output, optional thin panel consumer **only after GUI ADR** | success + injected errors, zero-display startup | stop only test app; delete isolated data |
| P02 TUNNEL + FAKE MCP | Run pinned verified official tunnel client with isolated tenant/profile + deterministic fake MCP echo/health | ChatGPT invokes mock echo; local status shows tunnel transitions | init/tools/list/tools/call, timeout, offline/reconnect, startup no GUI; evidence capture | stop isolated tunnel-client only, revoke temporary key |
| P03 SELF-MANAGEMENT MCP | Expose manager status, versions, health, recent sanitized error and tool invocations | ChatGPT reads CTMM and tunnel state, even when proxy has error | authorize all endpoints, read-only + denial cases, tunnel failure local fallback | disable experimental management endpoint |
| P04 PROXY/ROUTING | Smallest proxy routing 2 fake servers then representative real servers; `stdio` and Streamable HTTP; legacy SSE only if required | One ChatGPT connection discovers namespaced tools and calls both MCPs | capability/version negotiation, schema mismatch, collision, crash, stale server, policy deny | rollback registry/test routes, don't touch old proxy |
| P05 SETTINGS/REGISTRY | Persist basic tunnel/proxy and per-MCP config; validation, changes through local interface; gated AI add/enable/disable | ChatGPT shows proposed change, operator approves, tool reflects new status | invalid config, secret redaction, idempotent retry, rejected authorization and rollback | restore prior atomic config snapshot |
| P06 UI/TRAY/HEADLESS | Unified status/settings GUI for Ubuntu, connect to exact status source, separate from long-running service | Tray badge/panel updates with status from actual P02–P05, same states in CLI | click/manual visual, Wayland limitations, no DISPLAY boot, service survives UI close | remove UI client, retain service |
| P07 UBUNTU OPERATIONS | Boot/test isolated user/system service after privilege ADR, reconnect/backoff, journal/recovery, packaging | Fresh boot automatic connect; safe offline degraded state; retriable operations | service reboot, network loss, crash/restart, restart with no GUI, log audit | disable *new CTMM* service only; preserve existing |
| P08 WINDOWS / RELEASE | Windows service+tray reuse cross-platform contracts, compare behavior; explicit later release gate | Same snapshot and actions across OS | service/headless/GUI/proxy test matrix; signed artifact as approved | uninstall new app, import saved snapshot |
| CUTOVER OPTIONAL | Only after V1 requirements and risk review: approved transfer from old tunnel/proxy | verified parity, explicit before/after diff | sign-off, smoke tests, rapid rollback to old services | documented supervised revert; not automatic |

## Success checkpoints
- At each phase, compare `state == observed behavior` with a real test record; never present UI green for unexecuted check.
- P02 connection tests require user-provisioned **temporary** tunnel_id, runtime API key and required permissions in host secret store; never request those values in chat.
- P03 manager MCP exposes safe read-first tools such as `manager_status`, `list_servers`, `recent_failures`, `run_safe_selftest`; mutations `propose_server_change` then explicitly approved `apply_server_change`.
- P04 compatibility matrix specifies MCP protocol revision, transport, initialize/capabilities, tools/resources/prompts, auth, notifications, cancellation, error handling, independent server start/stop. Compatibility is measured, not assumed.
- No automatic 'latest' binary download at runtime without verified provenance + user-approved update/rollback policy.
- P06 GUI can be advanced earlier after ADR, but no service work may depend on it.
- Success requires clean test environment AND unchanged existing production processes/services by before/after inspection.

## Candidate first work item
`CTMM-TASK-P00-01`: survey official Secure MCP Tunnel docs, supported profile/mode on target ChatGPT account and OpenAI organization, rust SDK/protocol versions; produce comparison table and ADR choices. Acceptance: verified citations + isolated test design + blocking decisions; negative: privilege/version incompatibility documented; rollback: docs-only. Gate: explicit approval before executable creation.
