# CTMM V1 — Threat Model and Trust Boundaries (P00-01D)

Status: PROPOSED / DOCUMENTATION ONLY; no accepted implementation ADR. Event: CTMM-20261016-080 (Asia/Bangkok, 2026-10-16).
Owner: QA/Security/Release workstream. Task: CTMM-TASK-P00-01D, upstream PM dependency graph. Links: V1-R01–R09; Q03A–Q08A; ADR-CTMM-001–007 (all proposed); P01–P08 gates.

## 1. Scope, assumptions and non-goals

This is a clean-room threat model for the new Rust-based CTMM application, not the previous MCP Panel Manager. Ubuntu V1 requires a headless, pre-login background service, a GUI/tray release artifact, one ChatGPT-facing aggregated MCP endpoint, verified pinned tunnel delivery, and per-mutation independent human approval. Windows is a later target. Only new named, isolated test profiles may be used after separate execution authorization. No current-production tunnel, proxy, service, credentials, ports or identities may be changed by this document.

Model assumptions, not verified facts: an authorized OpenAI tunnel can route MCP requests to a private CTMM endpoint; local users and hosted AI callers have distinct identities; third-party MCPs may be malicious or compromised; UI is not a privileged source of truth; protected host secrets are not automatically available at loginless boot. Account/workspace entitlement, service identity, OS secret backend, approval transport, proxy adapter, toolkit and updater are UNKNOWN pending ADRs.

In scope: authenticated/unauthenticated MCP calls; tunnel-to-local boundary; manager control plane; local IPC; approval; per-profile config and secret references; third-party stdio/HTTP MCP; UI rendering; release binary supply chain; audit and recovery; isolated test escape. Out of scope for V1 P00: proving OS-kernel containment against root compromise, certification, penetration testing of OpenAI-hosted infrastructure, and claiming endpoint-to-endpoint compatibility without actual fixtures.

## 2. Protected assets and principals

| Asset | Integrity / confidentiality requirement |
|---|---|
| Host runtime credentials and MCP service keys | No plaintext in Git/Obsidian/chat/logs/portable exports, argv or arbitrary child environment; host-controlled provisioning and rotation |
| Registry desired-state and revisions | Atomic, scoped, versioned, tamper-resistant against unauthorized writes; configuration not equal to observed READY |
| Local service / control MCP privilege | Management read and write separated; no privilege granted by tunnel existence or model suggestions |
| Approval grants, identities and audit records | Short-lived one-time exact-diff authorization; durable denials and commits; no unlogged mutation on audit failure |
| Third-party tools, descriptions, resources and outputs | Untrusted data/code; no implicit authority; namespaced, bounded and isolated |
| Binary/toolchain/installer artifacts | Pinned version, checksum plus independent provenance/signature verification as applicable; approved updater and migration rollback |
| Production isolation | All tests constrained to dedicated identities/profiles/ports/directories and before/after unchanged-state evidence |

Threat actors: remote unauthenticated invoker; low-privilege authenticated MCP caller; malicious model-prompt/tool-output contributor; compromised or hostile third-party MCP; unprivileged local process or logged-in user; attacker with stolen limited runtime credential; malicious build/update dependency; accidental operator or CI misconfiguration. A fully compromised host administrator/root is a catastrophic trust-boundary loss, not an assumed preventable scenario.

## 3. Trust zones and boundaries

| Boundary | Source -> destination | Required control | Forbidden assumption |
|---|---|---|---|
| TB-01 | ChatGPT/workspace -> hosted tunnel -> local tunnel-client | Platform tunnel entitlement + ChatGPT workspace authorization; TLS, pinned tunnel ID; no public local admin ingress | Tunnel connectivity proves CTMM management authorization |
| TB-02 | Tunnel-client -> single CTMM MCP ingress | Explicit inbound client identity/auth scope, bounded JSON-RPC/protocol parsing, optional revision adapters, rate limits | Every discovered tool may be invoked or authorized |
| TB-03 | MCP management tools -> configuration writer | Independent server-side authz + immutable proposal + per-change local human approval + CAS; recheck on apply | AI-provided approved=true, UI hints or an MCP annotation is proof |
| TB-04 | GUI/tray/local operator -> headless service IPC | OS peer identity and ACL plus privileged step-up/reauth for approval, explicit role/scope | GUI and service share trust or secret access by default |
| TB-05 | Aggregating proxy -> third-party stdio/HTTP MCP | Per-server identity, process isolation, restricted env/fs/network, SSRF control, timeout, output bounds, namespace and schema validation | Tool descriptions/results, third-party stdout/stderr or redirected URLs are trusted |
| TB-06 | Headless service -> protected secret store | Dedicated least-privileged boot-capable identity, opaque secret references, narrow injection only to authorized target | Desktop keyring is unlocked before login |
| TB-07 | Git/CI/release -> installed binary/OS unit | Immutable dependency/action pins, least-privileged CI, SBOM/provenance, signature/hash and release approval; stage then rollback gate | SHA-256 alone authenticates who built a binary |
| TB-08 | Isolated fixture -> host/existing production | Dedicated namespace/profile/tunnel ID/ports/data root, allowlisted cleanup targets, before/after snapshot | Test run may discover/restart live services |

The tunnel's outbound HTTPS path to OpenAI does not remove the need to authenticate calls at CTMM's application boundary. OpenAI Platform Tunnels Read/Use/Manage are distinct from target ChatGPT workspace access; actual account entitlements remain NOT VERIFIED.

## 4. Abuse-case and risk register (initial pre-control priority)

Risk labels are planning priorities, not scored vulnerability findings. H=High, M=Medium. Re-score only after control design and tests.

| ID | Scenario / trust breach | Priority | Mandatory mitigation + future negative oracle |
|---|---|---|---|
| TM-01 | Stolen or overprivileged tunnel key reaches manager mutation | H | Per-tool CTMM principal/scope, no privilege inheritance; valid tunnel call alone denied |
| TM-02 | Approval forged via model text, metadata, UI flag or local callback | H | Approval from independently authenticated local channel, not model-controlled; forged request denied |
| TM-03 | Replay, expired, cross-profile or altered-diff approval | H | Bind actor + target + canonical diff digest + base revision + expiry + nonce; reject all variants |
| TM-04 | Concurrent AI/local applies overwrite one another | H | Registry revision CAS + single durable intent; one winner and no lost update |
| TM-05 | Tool description/output prompt injection triggers admin action | H | Treat as inert data; safe tool allowlist and independent policy; no privileged action |
| TM-06 | Malicious MCP executable reads manager secrets / filesystem | H | Separate child principal/env/data root, restricted fs/network; no inherited master secret |
| TM-07 | HTTP child redirects to metadata/private control addresses | H | Allowlisted endpoints, canonical URL/DNS/IP/redirect checks; SSRF attempt blocked |
| TM-08 | Local process calls unauthenticated IPC or impersonates tray | H | Unix peer credentials / Windows pipe SID ACL candidate, verify exact principal; deny |
| TM-09 | No boot-accessible secure credential but status reports READY | H | Secret retrieval must fail-closed, state NOT_CONFIGURED/FAILED/UNKNOWN; no false READY |
| TM-10 | Logging/audit persistence fails during an approved mutation | H | Audit-intent durable gate; mutation denied, no unaudited state transition |
| TM-11 | Proxy tool namespace collision or cross-profile route | H | Stable profile-scoped routing keys, collision rejection, no cross-tenant data |
| TM-12 | Malformed/oversized JSON, child hang/crash exhausts service | M | Length/depth/timeout/CPU/process limits, bounded retries; sibling route survives |
| TM-13 | Stale status from GUI/cache mistaken for live tunnel health | M | Live/snapshot source, observed_at, freshness, evidence, no remote success when tunnel offline |
| TM-14 | Hostile import contains shell command, plaintext key or executable path traversal | H | Strict schema + semantic lint, sanitized preview disabled by default, approval before enable |
| TM-15 | Malicious update/downgrade, unsigned installer, incompatible migration | H | Pin+provenance, signed release policy, compatibility/migration test, safe rollback or stop |
| TM-16 | CI/test cleanup selects existing production unit, ports or tunnel | H | Dedicated fixture manifest + explicit allowlist, independent before/after inventories |
| TM-17 | GUI renders unsafe MCP HTML or reveals secret through form/log | M | Escape inert text, strict webview/iframe policy, never render actual secret values |
| TM-18 | Attacker disables management/audit/tunnel via general registry change | H | Protected component denylist; out-of-band privileged recovery, no generic disable |

## 5. Security invariants / release-blocking oracles

- SEC-01: Unauthorized, absent, expired or wrong-audience caller produces no mutation and no confidential cross-profile data.
- SEC-02: Propose/validate/list never executes a new server, changes desired state or provisions secrets.
- SEC-03: Approval is never granted by a remote MCP tool; local approval authenticates a human and binds exact immutable change and current registry revision.
- SEC-04: Apply authorizes again, consumes one-time approval once, is idempotent per key and uses compare-and-swap to prevent lost updates.
- SEC-05: Denial, audit failure, stale revision, forbidden component, checksum mismatch and uncertain rollback fail closed without false READY.
- SEC-06: No raw secret values, auth headers, arbitrary child stderr or unsanitized tool payload appear in model-facing responses, diagnostics or release artifacts.
- SEC-07: Tunnel offline never blocks local trusted read-only diagnostics; tunnel liveness does not imply end-to-end MCP readiness.
- SEC-08: GUI absent, crashed or logged-out has no effect on headless service operation.
- SEC-09: A third-party MCP crash, malformed response or prompt injection cannot elevate management privileges, read another profile or terminate sibling servers.
- SEC-10: Imported configurations and updates are inert until independently validated and explicitly approved.
- SEC-11: Installer, service and fixture ownership remain separate from any pre-existing production service; no test can automatically cut over.
- SEC-12: A failed or non-reversible migration reports RECOVERY_REQUIRED and blocks automatic downgrade/rollback assertions.

## 6. Deterministic isolated test plan (NOT RUN)

| Test ID | Injected input / fault | Expected evidence |
|---|---|---|
| QA-TM-01 | Read-only principal requests apply | DENIED; revision and child counts unchanged; sanitized audit |
| QA-TM-02 | Model fabricates approval=true or fake privileged metadata | DENIED; approved grant absent |
| QA-TM-03 | Valid proposed diff then modifies one field before apply | Digest mismatch; DENIED |
| QA-TM-04 | Expired/consumed approval and cross-profile reuse | DENIED; at most one commit |
| QA-TM-05 | Two concurrent writers use same expected revision | One CAS winner, one conflict |
| QA-TM-06 | Audit sink unavailable | No commit, safe error, no false READY |
| QA-TM-07 | Prompt injection in child tool response / description | Inert data, no management action |
| QA-TM-08 | Child executes with fake secret sentinel / hostile env request | Manager master sentinel never reaches child/log |
| QA-TM-09 | HTTP MCP asks metadata IP / redirects from allowlist | Denied, no egress |
| QA-TM-10 | Unix socket or Windows pipe unauthorized peer (later OS fixture) | Denied before data/command |
| QA-TM-11 | No DISPLAY and desktop keyring locked at boot | Service reports safe failure, still locally diagnosable |
| QA-TM-12 | Child crash, hang, oversized output, invalid protocol revision | Bounded failure, sibling service intact |
| QA-TM-13 | Tunnel offline with cached READY snapshot | Local reports STALE/DEGRADED/UNKNOWN; remote inaccessible |
| QA-TM-14 | Cross-profile tool identity collision | Reject/namespace, no cross-profile route |
| QA-TM-15 | Import includes secret literal, unsafe executable or symlink escape | Reject, no disk/execution side effect |
| QA-TM-16 | Forged update digest, unsigned binary or failing migration | No activation, old isolated state retained if reversible |
| QA-TM-17 | Simulated crash between approval, durable intent, commit, reconcile | Durable operation outcome or RECOVERY_REQUIRED; never double-apply |
| QA-TM-18 | Test fixture manifest points to existing service/tunnel | Preflight refuses fixture without touching target |
| QA-TM-19 | Revoke auth token and attempt reconnect | No previous scope survives revoked credential |
| QA-TM-20 | GUI displays forged manager status from tool output | Only authoritative signed/authenticated local snapshot can drive state |

All tests above need deterministic fake identity, fake clock, fake store, fake audit, mock tunnel, two fake MCPs, revisioned registry and isolated no-network profiles first. Captured evidence MUST include test ID, assertion, actual observed result, run_id, artifact provenance/commit and isolation before/after diff. A test plan is not a PASS. Test execution, network use, binaries, credentials, tunnel/profile creation and OS service changes require separate approval.

## 7. ADR dependencies, blockers, ownership

| Decision | Blocking question | Owner |
|---|---|---|
| ADR-CTMM-003 | Boot service account/system vs lingered user service; boot-time host secret availability; Windows parity | Core/Service + Security |
| ADR-CTMM-004 | Local IPC auth, peer credentials, remote principal validation, approval channel and step-up | Self-management + Security |
| ADR-CTMM-006 | Registry store, secret provider, per-change human approval, audit durability, rollback | Registry + Security |
| ADR-CTMM-001 | Official pinned binary provenance, licensing, privilege and updater | Tunnel + Release |
| ADR-CTMM-002/005 | MCP 2025/2026 transport and auth boundary; one aggregated namespace | Proxy + Security |
| ADR-CTMM-007 | Unprivileged GUI/tray, Linux package and Windows installer controls | GUI + Release |

Open questions for review: which principal can approve before graphical login (default proposed answer: no new mutation before login, approved desired state may auto-start); whether running arbitrary third-party stdio servers is permitted without OS sandbox (default: deny or require explicit high-risk operator exception); audit storage and retention policy; local recovery credential/step-up; exact target isolation semantics and release artifact signing mechanism.

## 8. Acceptance, rollback and provenance

Task P00-01D documentation acceptance: identified principals/assets and boundaries; enumerated high-risk abuse cases; testable fail-closed invariants; mapped QA negative test IDs; marked unknown ADRs and safety preconditions. Positive result means DOCUMENTED/REVIEWED ONLY, not vulnerability mitigated or runtime verified. Validate links and paired logbook presence before status SYNC_VERIFIED.

No executable, service, tunnel or secret is created. Revisions to this proposal require a new append-only correction event in both Git and Obsidian, preserving the original audit trail. Runtime rollback is NOT APPLICABLE; isolated future tests must specify their own restore plan.

References inspected 2026-10-16:
- OpenAI Secure MCP Tunnel: https://developers.openai.com/api/docs/guides/secure-mcp-tunnels
- OpenAI MCP server tool use: https://developers.openai.com/api/docs/guides/tools-connectors-mcp
- OpenAI platform RBAC: https://developers.openai.com/api/docs/guides/rbac
- Internal: docs/requirements/V1-Q03-Q08-CONFIRMED.md; docs/adr/PROPOSED-ADR-CTMM-001-007.md; docs/architecture/SELF-MANAGEMENT-MCP-PROPOSAL.md; docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md; docs/plans/V1-MILESTONES.md; Obsidian Planning/Task-Dependencies.md.

Evidence so far: read authoritative Git and Obsidian project documents and public official specs; design review only. All security, Rust, system, GUI, updater, permission and E2E tests: NOT RUN.