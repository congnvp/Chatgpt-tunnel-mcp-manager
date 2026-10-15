# CTMM V1 — Self-management MCP contract and security gates
Status: **PROPOSED ONLY** · Event: CTMM-20261016-030 · Workstream: Self-management MCP
Scope: V1-R04/R06/R08/R09, Q03A/Q05A/Q07A; P03 (read) and P05 (mutations). Dependencies: ADR-CTMM-002/004/005/006 all PROPOSED. No implementation authorization.

## Ownership and compatibility
- Rust headless core owns authoritative desired registry and observed status. GUI, CLI and ChatGPT are clients with distinct trust boundaries; tunnel is transport, not an administrator.
- One outward ChatGPT MCP catalog (Q07A) routes to the manager namespace and independently managed downstream servers. Never assume the tunnel itself grants registry mutation authorization.
- Reuse Registry/Settings identity (`profile_id`, `server_id`, `alias`), service-owned monotonic `registry_revision`, sanitized runtime snapshot, and audit journal; do not implement a second registry writer.
- Compatible wire adapters MUST be tested per negotiated protocol revision: MCP 2026-07-28 is stateless and uses optional `server/discover`; 2025-11-25-era MCP uses handshake/session conventions. Domain tools stay transport-agnostic. Object-root structured content is intentional for cross-era compatibility.
- Client/tool annotations or UI confirmation alone are not authorization. Tool visibility may be filtered, but all invocations must independently pass server-side authorization.

## P03 read-only tools
| Tool | Input | Authoritative output | Security / failure behavior |
|---|---|---|---|
| `manager_status` | empty | service/tunnel/proxy typed state + registry revision, source, observation time, freshness | sanitized; no successful remote call when tunnel itself is offline |
| `manager_diagnostics` | component, bounded limit/cursor | sanitized code, severity and safe message | never raw stderr, access token, headers or private env |
| `manager_list_servers` | scoped profile, bounded limit/cursor | desired enabled vs observed state; stable server ID | no downstream secret refs or profile cross-scope leak |
| `manager_selftest` | bounded suite from allowlist | per-check PASS/FAIL/INCONCLUSIVE; side_effects=0 | no network probing or subprocess execution via these suites |
All input and output schema definitions: `docs/contracts/self-management-mcp-v1.schema.json` (`$defs/Manager*Input` and `$defs/Manager*Output`). `source=live|snapshot|unavailable`; `freshness=fresh|stale|unknown`. Never turn stale snapshot into READY proof.

## Local offline fallback
- Tunnel OFFLINE but core alive: trusted local IPC/CLI can read core snapshot; remote ChatGPT MCP calls through that tunnel cannot succeed. No alternative public ingress is added by this design.
- Downstream MCP failed but manager and tunnel healthy: manager read tools report downstream DEGRADED/FAILED truthfully.
- Core offline: a separately authorized local recovery reader may show last durable snapshot as `snapshot/stale` only; it is not live health.
- No snapshot: `UNKNOWN/unavailable`, never `READY`. No GUI/DISPLAY requirement for any local observation.

## P05 write workflow (deliberately not exposed during P03)
1. `manager_propose_change`: authenticate operator principal and profile permission, canonicalize and validate local draft; create immutable proposed diff and binding to principal, profile, base revision and digest. No runtime/config mutation.
2. Local `approve`: separate trusted GUI/CLI operator path, with OS peer identity and re-authentication/step-up as ADR determines; never a model-controllable MCP approve tool. Store one-time, expiring approval bound to exact digest + operator + target/scope + revision.
3. `manager_apply_change`: authenticate and authorize again; enforce rate limit, proposal ownership, digest, approval, revision CAS and policy; durable audit intent; idempotent commit of desired config; reconciliation and verified observed state as a separate step.
4. `manager_rollback_change`: protected operator-only operation; restore previous desired generation if safe, then reconcile and verify; cannot claim runtime rollback complete unless observed proof exists. If uncertain, `RECOVERY_REQUIRED` and halt further mutations.
- `add` is a reference to a **locally staged, validated registry draft** (`draft_id`) with profile scope. The model cannot submit arbitrary executable, shell command, URL or plaintext secret directly in `apply`.
- `enable` and `disable` address a server within a profile, never the built-in manager, audit, tunnel supervisor or other protected component. Startup of previously approved configuration at boot does not imply approval for new changes before login.
- A repeated identical idempotency key returns the recorded outcome without re-executing. Same key with different payload is a conflict. Every denied call is audited without leaking input. Audit failure => mutation fail-closed.
- Desired commit success may initially return `COMMITTED_PENDING_RECONCILE`; `READY` only after independent health proof. Failed reconcile must surface `RECONCILE_FAILED` and optional supervised rollback.

## Permission matrix (draft)
| Operation | Viewer | Operator with target scope | Local approver |
|---|---|---|---|
| status/list | allow scoped | allow scoped | allow scoped |
| diagnostics | allow sanitized | allow sanitized | allow sanitized |
| selftest | deny | allow safe suites | not implicit |
| propose | deny | allow scoped | not implicit |
| approve | deny | deny over remote MCP | allow via local trusted channel |
| apply | deny | allow ONLY with valid independent approval | no implicit remote apply |
| rollback | deny | exceptional gate only | exceptional local authorization |
- Default-deny anonymous/unknown principals; verify issuer, audience, expiry, scopes, profile/target ownership; identity in `_meta` is untrusted metadata. Never pass incoming token to downstream servers.
- Proposed starting budgets per principal (NOT approved): status 60/min, diagnostics 10/min, list 30/min, selftest 2/min, propose 3/min, apply 1/min; one registry mutation at a time. Fault tests must cover rate-limit storage failure.
- Runtime operation journal is independent of Git/Obsidian project logbooks, tamper-evident design pending ADR-CTMM-006. Store only redacted identifiers/digests, authorization decision, revision, outcome, timestamps and rollback linkage.

## Fake examples (not observed results)
Status offline: `source=live`, `service=READY`, `tunnel=OFFLINE`, `proxy=READY`, `overall_state=DEGRADED`, `registry_revision=12`; remote ChatGPT through offline tunnel unavailable.
Selftest example: `suite=registry_integrity`, `result=PASS`, `checks=[{id:unique_server_ids,result:PASS}]`, `side_effects=0` (SIMULATED, NOT TESTED).
Unauthorized apply example: `outcome=DENIED`, `reason_code=APPROVAL_REQUIRED`, `registry_revision=12`; no side effects (SIMULATED).

## Negative and acceptance test plan — ALL NOT RUN
| ID | Injection / scenario | Expected invariant |
|---|---|---|
| SM-01 | missing/expired token | deny, no data or mutation |
| SM-02 | wrong audience/issuer | deny, no action |
| SM-03 | viewer calls propose/apply | denied, unchanged revision |
| SM-04 | forged actor or capability in `_meta` | ignored for authorization |
| SM-05 | propose call | no desired/runtime side effect |
| SM-06 | apply without independent approval | DENIED + audit |
| SM-07 | approval expired or revoked | DENIED |
| SM-08 | digest or scope differs from approved | DENIED |
| SM-09 | registry revision stale | CAS conflict, no overwrite |
| SM-10 | proposal owned by another principal | deny IDOR |
| SM-11 | replay same key/same payload | return same recorded outcome, once-only mutation |
| SM-12 | same key/different payload | conflict, no new mutation |
| SM-13 | rate limit exceeded | deny before side effect |
| SM-14 | audit store failure | fail-closed mutation |
| SM-15 | unknown draft or untrusted executable/SSRF path | reject with safe code |
| SM-16 | protected manager disable | reject |
| SM-17 | crash before/after desired commit | consistent revision or recovery-required; no false READY |
| SM-18 | reconcile failed / rollback failed | observable state truth, block follow-on writes |
| SM-19 | no tunnel / no DISPLAY | local read still works, no remote-success claim |
| SM-20 | injected prompt in MCP description/errors | inert data, no bypass of policy |
| SM-21 | raw error includes fake secret | redacted output and journal |
| SM-22 | 2025 legacy initialize vs 2026 stateless discover | compatible adapters or explicit unsupported error |
| SM-23 | two concurrent applies same revision | at most one commit |
| SM-24 | incomplete snapshot or clock skew | freshness UNKNOWN/STALE, not green READY |
Fixtures: fake identity/approval/audit/clock/registry/tunnel/proxy; deterministic concurrency and crash checkpoints; isolated no-network profiles. Never use or restart production tunnel/proxy in a test.

## Architecture decisions and handoff
- Map to existing ADR-CTMM-004 (IPC/auth) and ADR-CTMM-006 (registry/OS secrets/approval). Proposed subquestions SM-D01 identity+scopes; SM-D02 trusted local approval; SM-D03 audit failure semantics; SM-D04 reconciliation/rollback; all **OPEN** and must not supersede accepted ADR without an event.
- Dependent on Registry/Settings RS-D01/02/03 field contract, Proxy PX namespace/auth, and Core typed health API. First eligible future implementation slice is P03 read-only after P01/P02 gate; P05 mutation remains blocked by explicit ADR and execution authorization.
- Rollback of this document: append a correction event and revise proposal; no source code/runtime or service rollback needed.

## References inspected 2026-10-16
- https://blog.modelcontextprotocol.io/posts/2026-07-28/ (MCP stateless revision, header routing, auth hardening)
- https://developers.openai.com/api/docs/guides/secure-mcp-tunnels (tunnel transport, separation of platform/workspace permissions)
- https://ts.sdk.modelcontextprotocol.io/v2/migration/support-2026-07-28 (revision-specific transport behavior)
- Internal: `docs/requirements/V1-Q03-Q08-CONFIRMED.md`, `docs/adr/PROPOSED-ADR-CTMM-001-007.md`, `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`, `docs/research/PROXY-INTEROP-AND-FAKE-TEST-MATRIX.md`.
