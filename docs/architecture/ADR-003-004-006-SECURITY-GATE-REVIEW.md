# CTMM — ADR-003 / ADR-004 / ADR-006 security gate review
Status: **PROPOSED / REVIEW ONLY; NO ADR ACCEPTED**  
Event: CTMM-20261016-060 · 2026-10-16 04:35 Asia/Bangkok · Workstream: Requirements/Architecture  
Scope: CTMM-TASK-P00-01D (threat, identity and approval analysis) and P00 architecture readiness. All P01+ execution remains blocked.

## A. Verified baseline and conflict-recovery checkpoint
- Confirmed user requirements Q03A (system boot before user login), Q04A (verified pinned tunnel and reviewed upgrades), Q05A (AI proposes, human approves each change), Q06A (Ubuntu GUI/tray is a V1 *release* requirement but not a boot dependency), Q07A (one aggregated ChatGPT MCP endpoint), Q08A (separately authorized isolated E2E).
- Official Git `CTMM-20261016-030` belongs to Self-management MCP: `docs/architecture/SELF-MANAGEMENT-MCP-PROPOSAL.md`, `docs/contracts/self-management-mcp-v1.schema.json`, commit `05f6c5b4e959a82c47f31273a503ed7fec157713`. Its Obsidian architecture note and **late mirror** Logbook event 030 have been read back.
- Registry originally placed event 030 in three immutable commits. Event `CTMM-20261016-040` supplies a separate corrective attribution, documentation and logbook records (Git log commit `7b0ac6f6c05b6751e9fccda47f7b43b17d7afcd6`); Obsidian 040 logbook and proposal note were read back. Do not rewrite original Git history.
- Earlier Git log entry 030 has `SYNC_PENDING` as a **historical state at write-time**; the paired Obsidian 030 late mirror now exists. This event 060 records the later readback checkpoint. Subsequent concurrent changes must be re-verified independently.
- Registry structural schema smoke evidence recorded by event 040: `SCHEMA_CHECK_VALID; CASES=17; MATCH=17; MISMATCH=0`. This is **structural JSON Schema only**, not security, semantic validation, Rust runtime or E2E.
- Systemd, Linux socket and project records were reviewed; the host's actual boot credentials, service permissions, MCP behavior, Windows parity and current runtime have NOT been tested by this workstream.

## B. Trust boundaries (design requirements, not mechanism choices)
```text
[OpenAI ChatGPT model / MCP client] --untrusted caller--> [remote MCP admission + authz]
       -> [read-only manager tools OR vetted propose command]
       -> [authoritative Rust policy/service] <-- [local GUI / CLI as separately authenticated principals]
       -> [desired registry / audit / per-profile secret reference resolver]
       -> [supervisor] --> [untrusted child MCP server or remote MCP URL]
                     --> [version-pinned tunnel transport]
```
The service is independent from GUI and tunnel. Tunnel connectivity MUST NOT confer privileged local control. MCP tool metadata, server descriptions and model text are untrusted input. Approval state is never derived from a ChatGPT confirmation, request JSON boolean, or tool annotations. A boot-time auto-connect may use **previously approved persisted desired state** only.

## C. ADR-CTMM-003 — Boot service identity and boot-time secret delivery
**Context:** Rust service must run before login with no DISPLAY. Secret availability and privilege isolation are blocking; a locked desktop keyring may not be accessible.
- A — `systemd` system unit with a dedicated least-privileged **static** service user; unit managed by system service manager. Pros: predictable pre-login startup, scoped filesystem ownership and stable UID for peer/secret ACL. Cons: packaging/admin setup, multi-user ownership model and privilege separation need design.
- B — `systemd --user` with linger enabled only by authorized operator. Pros: per-user ownership, possibly simpler user MCP paths. Cons: linger enablement and headless user manager/secret access must be proven; desktop keyring may be locked; may not meet multi-user/service-account intent without tradeoffs.
- C — root/system privileged daemon. Pros: provisioning simplicity; cons: enlarged blast radius for arbitrary third-party MCP child execution, unacceptable without separately justified privilege isolation. Do NOT treat as default.
- Candidate credential delivery under A: `LoadCredential=` for appropriately protected source or `LoadCredentialEncrypted=` for protected encrypted blob; app consumes `$CREDENTIALS_DIRECTORY` and NEVER logs contents or copies to Git/Obsidian. Official systemd documentation describes per-unit immutable credentials readable by unit identity. Encryption/key-provider and first-boot/unseal availability still require host-specific verification. A service compromise can still read credentials granted to it; encryption at rest does not prevent runtime misuse.
- Design alternatives for sensitive credentials: service-scoped protected OS provider, TPM2-bound encrypted credential if proven available at boot, or dedicated local broker process with separately defined principal and ACL. No backend selected.
- Windows later: Windows Service identity, service SID and OS-protected storage are separate ADR portability tests; do not assume `systemd` unit metadata ports to Windows.
- Positive acceptance: cold boot without human login, service process responds locally with fresh truthful snapshot; previously approved config reconnects only if valid/provisioned; least-privilege ownership demonstrated.
- Negative acceptance: missing/unsealable credential -> blocked connection with explicit `CREDENTIAL_UNAVAILABLE`, no false READY; offline boot -> bounded retry; no DISPLAY; unexpected user cannot read credential; child compromise cannot read unrelated profile's secret; restart storm bounded.
- Rollback: disable/remove **only dedicated new test unit**, erase approved test credential reference/material, restore isolated config snapshot, compare old production stack unchanged.
- **OPEN decision:** A vs B; static vs dynamic service user; secret backend and boot unlocking, profile owner semantics, service installation rights.

## D. ADR-CTMM-004 — Local IPC, principal binding, remote boundary
**Context:** Local GUI, CLI and remote management MCP must read same authoritative state, but they MUST NOT share implicit privileges.
- A — Linux Unix Domain Socket with filesystem permissions plus kernel `SO_PEERCRED` for authenticated connected peer; Windows Named Pipe with explicit ACL/SID enforcement as platform adapter. Pros: default local-only, OS identity. Cons: OS-specific adapters, group membership may authorize too widely, cannot infer human approval solely from peer UID.
- B — loopback HTTP with authenticated request identities, issuer/audience checks, origin/CSRF and firewall/path controls. Pros: uniform client transport and tooling. Cons: increased attack surface including webview/browser loopback/SSRF, TLS/token operational complexity.
- C — unauthenticated local TCP or world-writable socket: excluded as mutation endpoint; violates least privilege.
- Proposed typed operations: `GetSnapshot`, `SubscribeStatus`, `ValidateDraft`, `SubmitProposal`, and independently `ApproveExactDiff` for suitably authenticated local approver. Remote MCP must never expose `ApproveExactDiff`. Runtime endpoint names/types remain candidates until ADR acceptance.
- Identity requirements: map kernel peer principal (local) or validated remote identity (MCP) to explicit `viewer/operator/approver/admin` role **and** scoped profile/target; never trust caller-declared role or unverified `_meta`. GUI is a client, not service parent.
- Zero-trust checks: per-method authz, max message size/rate, revision/CAS, connection timeout, event sequence and stale detection, diagnostics redaction, no broad environment inheritance. Approval step-up and session binding need human decision.
- Positive: authorized local viewer obtains sanitized authoritative snapshot offline; GUI crash/close never terminates service; remote scoped viewer cannot access other profiles.
- Negative: forged PID/actor, unauthorized socket group, cross-user IPC, malicious remote auth context, stale snapshot, request flood, wrong protocol version and loopback browser request -> rejected with safe codes; config unmodified.
- Rollback: disable **test IPC endpoint** or restore test socket ACL, independently retain headless service status, no production modification.
- **OPEN decision:** A vs B, IPC schema/version lifecycle, local roles/authentication, Windows ACL plan, fallback when IPC unavailable.

## E. ADR-CTMM-006 — Registry, audit, secrets and operator per-change approval
**Context:** Every AI-origin mutation must be proposed without side effects, approved by a human at a trusted boundary and applied once under the same exact policy and base revision.
- Storage A — service-owned validated atomic JSON/TOML desired-state document + separately protected host secret provider. Pros: simple inspectable state/backup. Cons: crash consistency across audit+config and concurrent writers needs explicit journal/atomicity mechanism; no direct external edits bypassing validation.
- Storage B — SQLite durable desired-state, revisions and proposal/audit metadata + separately protected host secret provider. Pros: stronger transactional revision/CAS, restart recovery. Cons: migrations/backup/locking and protection of database file. SQLite is not a secret vault.
- Disallowed — direct plaintext secrets in config, raw shell/URL from model handed directly to execution, GUI-only confirmation logic, ChatGPT model issuing approval tokens, unvalidated state edits.
- Approval channel A — trusted local GUI with re-auth/step-up and explicit immutable diff display (convenient for V1 GUI, requires desktop login). B — trusted local privileged CLI with isolated operator authentication (headless admin use, harder UX). C — remote enterprise approval/identity provider (more complex, not V1 prerequisite absent need). For both A/B, approve while user is present; if nobody is logged in, new proposed mutations remain pending.
- Four-phase invariant: `propose(change, expected_revision, idempotency_key)` -> `human approve(proposal_id, exact digest, principal, profile, expiry, nonce)` -> `apply (re-authenticate + revalidate policy + compare-and-swap)` -> `reconcile desired/observed`. Proposal cannot start/stop a server. One-time approval is bound to canonical delta and target revision; apply is not itself proof of READY.
- Audit: write tamper-evident or append-only operation intent and outcome without secrets; abort privileged mutation if mandatory audit sink fails; idempotency and crash recovery MUST be durable. Project Git/Obsidian logbooks are for engineering changes, not live runtime operation logs.
- Protect built-in management namespace and supervisors from generic add/enable/disable. For third-party executables reject shell expansion, enforce allowlisted paths/resources, and apply transport-specific network and secret scope checks; default imported profiles disabled.
- Positive: approved exact proposal commits desired state once, record reason/actor/old-new revision, reconciliation verifies observable status. Previously approved saved config can auto-reconcile after reboot without prompting human again.
- Negative: unauthorized/expired/replayed/mismatched approvals, stale revision, inconsistent policy, audit failure, double apply, forged local identity, malicious MCP metadata, cross-profile leakage, corrupt store and failed rollback -> deny or `RECOVERY_REQUIRED` without false READY. Recovery actions themselves require privileged local gate.
- Rollback: restore previous isolated desired generation, audit compensation, revalidate before starting MCP; when real state is uncertain, stop further changes and surface `RECOVERY_REQUIRED`. No automatic rollback of active production stack.
- **OPEN decision:** storage A vs B; approval A vs B/C; secret provider; role/tenant model; durable audit mechanism; policy for in-flight MCP calls.

## F. Cross-ADR dependency and approval gates
1. Prerequisites for P01 Rust status-only slice: state contract, no-DISPLAY invariant, core/IPC trust contract and bounded synthetic identity tests; no real secret or network required. No Cargo project until blocking ADRs accepted.
2. Prerequisites for P02 isolated tunnel/mock MCP: ADR-001 tunnel choice/provenance; ADR-003 boot secret plan (test profile); ADR-004 remote/local auth; ADR-006 secret+operator policy; platform/workspace entitlements and separate test approval. No approved run yet.
3. Prerequisites for P03/P05 self-management writes: ADR-004 principal mapping and ADR-006 policy/approval+registry durability, fake auth/audit/clock/registry; all SM-01..SM-24 negative cases planned, NOT RUN.
4. Ubuntu V1 release: P06 GUI/tray and P07 boot operations each MUST pass, but GUI remains optional for service runtime. Windows follows separately.
5. All ADR-003/004/006 **remain PROPOSED**. User must choose explicit alternatives; no design can be marked ACCEPTED from Q03A..Q08A alone.

## G. Evidence, test and rollback status
- Readback from Git/Obsidian logs 030 and 040 and source documents. No request sent to running tunnel, proxy or service. Host settings/ACL/secret store were NOT modified or tested.
- Registry `CASE=17` source-smoke is independently recorded by 040; it is **not** evidence of IPC/secret/auth security.
- Proposed test groups: BOOT-01..04, IPC-01..06, AUTH-01..10, RECOVERY-01..03. ALL `NOT RUN` in this workstream; no PASS claims.
- No accepted ADR, no implementation, no live E2E or Windows validation.

## References reviewed
- CTMM: `docs/requirements/V1-Q03-Q08-CONFIRMED.md`; `docs/adr/PROPOSED-ADR-CTMM-001-007.md`; `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`; `docs/architecture/SELF-MANAGEMENT-MCP-PROPOSAL.md`; `docs/architecture/REGISTRY-V1-SCHEMA-VALIDATION.md`.
- systemd service credentials (security and boot caveats): https://systemd.io/CREDENTIALS/ and https://www.man7.org/linux/man-pages/man5/systemd.exec.5.html
- Linux AF_UNIX peer identity: https://man7.org/linux/man-pages/man7/unix.7.html
- Windows named pipes security: https://learn.microsoft.com/en-us/windows/win32/ipc/named-pipe-security-and-access-rights
- Git event 030 source commit: https://github.com/congnvp/Chatgpt-tunnel-mcp-manager/commit/05f6c5b4e959a82c47f31273a503ed7fec157713
- Registry 040 correction: https://github.com/congnvp/Chatgpt-tunnel-mcp-manager/commit/7b0ac6f6c05b6751e9fccda47f7b43b17d7afcd6
