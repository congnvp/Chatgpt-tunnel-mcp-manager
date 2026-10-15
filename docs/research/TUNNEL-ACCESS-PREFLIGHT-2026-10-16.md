# CTMM — Secure MCP Tunnel access and permissions preflight
Status: SOURCE-REVIEWED / ACCOUNT-ACCESS UNKNOWN / NO RUNTIME TESTS
Task: CTMM-TASK-P00-01A (P00-01 / Tunnel); Event: CTMM-20261016-021
Time: 2026-10-16 04:21 Asia/Bangkok; actor: ChatGPT Integration/PM performing assigned desk-only Tunnel preflight.
Related: V1-R02/R03/R08/R09; Q04A/Q05A/Q08A; ADR-CTMM-001/003/004/006 PROPOSED; NO ADR ACCEPTED.

## Evidence policy
All evidence below is published OpenAI documentation, checked on 2026-10-16. `DOCUMENTED` means **public documentation describes a requirement**, NOT this user's account has that access. `UNKNOWN` means a live operator verification remains necessary. `BLOCKED` means precondition to a *future* execution has not been authorized or demonstrated. Research does not grant permission to create a tunnel, authenticate, install, or modify active services.

Sources:
- S1: https://developers.openai.com/api/docs/guides/secure-mcp-tunnels — before-you-start, permissions/access, workspace association, networking, setup, troubleshooting.
- S2: https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt — custom MCP connections, account/workspace permission and owner controls. UI terminology and plan-specific beta availability subject to change; verify at operator test time.
- S3: https://developers.openai.com/api/docs/guides/tools-connectors-mcp — remote MCP vs tunnel-backed MCP and Responses API.
- S4: https://help.openai.com/en/articles/12584461-developer-mode-and-full-mcp-connectors-in-chatgpt — legacy developer-mode wording and plan/security constraints; cross-check with newer S2; do not treat a cached UI path or account eligibility as established.

## Evidence and access matrix
| Requirement/check | Published evidence | Design implication | Documentation status | Real-world verification owner/status |
|---|---|---|---|---|
| Tunnel endpoint identity | S1 requires `tunnel_id` from Platform tunnel settings | Allocate a NEW isolated tunnel ID, never reuse production tunnel | DOCUMENTED | Platform org operator: UNKNOWN / not requested |
| Creating/editing a tunnel | S1: org-level `Tunnels Read + Manage` | Separate provisioning role from runtime account | DOCUMENTED | Platform org owner/RBAC administrator: UNKNOWN |
| Running tunnel-client / selecting tunnel | S1: `Tunnels Read + Use` | Restrict runtime identity; do not infer Manage or owner | DOCUMENTED | Platform org RBAC admin: UNKNOWN |
| Organization/workspace mapping | S1: tunnel association includes intended Platform organization and ChatGPT workspace; other org association as needed | A tunnel visible in Platform may be invisible in target ChatGPT workspace | DOCUMENTED | Platform org owner and ChatGPT workspace admin: UNKNOWN |
| ChatGPT custom MCP permission | S1 + S2: separate workspace permissions / security restrictions apply | ChatGPT operator access cannot be inferred from API Platform RBAC | DOCUMENTED | ChatGPT workspace admin: UNKNOWN |
| Runtime credential | S1 requires runtime API key for `tunnel-client` | Provision later only through protected host secret channel and separate test authorization; never store literal | DOCUMENTED | Security/operator: BLOCKED for test, not provisioned |
| Private MCP reachability | S1: `tunnel-client` must reach MCP via stdio or private HTTP | Fixture must be under separately scoped profile/port; no production paths | DOCUMENTED | Fixture implementer: NOT TESTED |
| Egress networking | S1: outgoing HTTPS to `api.openai.com:443` on `/v1/tunnel/*`, or `mtls.api.openai.com:443` with mTLS; no inbound public port needed | Firewall allowlist and egress proxy decision for isolated test host | DOCUMENTED | Test host operator: NOT TESTED |
| Client liveness | S1: running `tunnel-client` and local `doctor`, `/healthz`, `/readyz` surfaces are prerequisites for reliable discovery | Only later prove E2E with local readiness + ChatGPT tool call; health != E2E alone | DOCUMENTED | Tunnel test owner: NOT RUN |
| Private connection vs publication | S1 describes private developer/testing; public plugin submission requires public HTTPS endpoint | V1 tests private ChatGPT connection; do not claim public marketplace publishing | DOCUMENTED | Product PM: no release submission planned |
| Runtime account & ChatGPT plan eligibility | S1/S2 describe requirements; actual account/workspace and enabled capabilities not accessible from public docs | Must check operator's intended plan, org, workspace, permission, UI at future approved checkpoint | UNKNOWN | Authorized human, NOT VERIFIED |

## Isolated E2E authorization gate — NOT GRANTED
Before any test: (1) identify target Platform org and ChatGPT workspace *without copying IDs into public artifacts*; (2) confirm role split: creator Read+Manage, runtime/use Read+Use, separate workspace permission; (3) request and receive explicit authorization for NEW isolated tunnel, fixture, profile/port and test period; (4) provision temporary least-privilege runtime secret via host secret store, not chat/Git/Obsidian/CLI history; (5) capture before-state of existing tunnel/proxy/services; (6) check approved artifact provenance and fixture readiness; (7) after authorization, record local health/doctor and ChatGPT initialize/discovery + safe fake tool call evidence; (8) revoke/rotate test credential and compare unchanged production before/after. No step in this list has been executed in this task.

## Negative/failure checklist (DESIGN ONLY — NOT RUN)
- No `Read + Manage` → deny creation/edit; no `Read + Use` → deny client/use.
- Wrong organization/workspace association → tunnel absent in ChatGPT even when Platform view works.
- Custom MCP disabled/not permitted in workspace → ChatGPT connection blocked, not assumed plan capability.
- Outbound HTTPS blocked, missing proxy trust/mTLS, or MCP unreachable → no false READY.
- Absent/revoked runtime key or stopped client → no claims of ChatGPT E2E success.
- Doctor/health appears healthy but ChatGPT `tools/list` or fake `tools/call` fails → distinguish transport readiness from E2E.
- Secret in command arguments, logs, screenshot, public repo or notes → reject evidence, redact and investigate before recording.
- Version, UI label or entitlement documentation drift → `UNKNOWN`, re-check official current product surface, never convert into permission claim.

## Handoff and acceptance
Acceptance of **desk research only**: official source URLs captured; permission categories separated; workspace association and egress recorded; account-dependent items explicitly UNKNOWN; negative cases and future gate articulated; no real credentials, IDs, client binaries, tunnel executions or permission changes. Evidence: official pages inspected; no automated integration/unit, runtime, security or E2E tests; `NOT RUN`. Rollback for documentation: superseding correction event with dual logbook, no deletion of history. Mark `CTMM-TASK-P00-01A` `RESEARCH_DOCUMENTED` after paired readback (not permission-verified). Parent P00-01 remains OPEN.

## Open inputs for subsequent P00/ADR and operator
- Exact intended Platform organization and ChatGPT workspace mapping and entitlement; no IDs/credentials in chat.
- Appropriate permission grant procedure by authorized org/workspace admins.
- Test-only host and secret store usable BEFORE login (Q03A) without user desktop keyring, and least-privilege runtime principal.
- Run-time artifact provenance/version and license belong to P00-01B, not established here.
- Aggregation one ChatGPT MCP endpoint (Q07A) is NOT proven by mere existence of tunnel channels; requires distinct isolation test (P02/P04).
