# CTMM Registry V1 — structural schema + validation gates
Status: PROPOSED / DOCUMENTATION CONTRACT / NOT AN ACCEPTED ADR
Event: CTMM-20261016-030 · Task: RS-D01 · Date: 2026-10-16 (Asia/Bangkok)
Related: V1-R04/R06/R07/R08/R09; ADR-CTMM-002/004/005/006/007 PROPOSED; D-ARCH-02/04/06 OPEN.

## Source files (schema, not application code)
- `docs/architecture/schemas/registry-v1.proposed.schema.json` — strict Draft 2020-12 desired-state structural validator candidate.
- `docs/architecture/examples/registry-v1.inert-example.json` — inert disabled fake stdio server + disabled remote HTTPS placeholder (`example.invalid`). Placeholder secret reference is NOT a credential, and is never resolved in these documentation tests.
- Previous workstream: `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md` (CTMM-20261016-010).
- Later confirmed user decisions: `docs/requirements/V1-Q03-Q08-CONFIRMED.md` (CTMM-20261016-020) override earlier conflicting GUI-optional *release* wording.

## Confirmed constraints applied, without choosing implementation
1. Q03A: system-boot-before-login headless service. The only boot-time activations are **previously approved persisted** configurations with valid local policy, authorized profile scope, available secret provision and compatible tunnel; no unattended mutation or assumption that desktop keyring is unlocked.
2. Q04A: pinned verified compatible tunnel; no registry action may implicitly replace or auto-update tunnel binaries.
3. Q05A: each AI-proposed mutation is independently human-approved. Approval is bound to exact proposal digest, actor, target profile, expected revision, expiry and nonce, and rechecked server-side at apply. No UI click/model call/annotations can themselves create elevated authority.
4. Q06A: Ubuntu GUI/tray required for **V1 release**, but ChatGPT MCP Apps UI remains optional and GUI remains absent from boot/service execution path.
5. Q07A: one outward ChatGPT MCP endpoint aggregates eligible servers. Profile existence alone does not confer exposure; access-controlled aggregated catalog and reserved built-in management namespace require Proxy+Security ADR.
6. Q08A: **no live isolated E2E** until discovery and separate operator execution approval. This document performs only isolated local JSON Schema smoke validation.

## Syntactic schema invariants (enforced by Draft 2020-12)
- Root object strictly allows `schema_version=1`, service-owned nonnegative `registry_revision`, bounded arrays of `profiles` (1..32) and `servers` (0..256); unknown object properties rejected throughout.
- Profile has ASCII lower-case ID, human label and candidate activation enum `manual|boot_approved`. The latter is a **desired label**, not sufficient authorization.
- Server requires stable ID, existing profile identifier (existence checked separately), ASCII lower-case alias, display name, explicit boolean `enabled`, discriminated transport, bounded execution controls and explicit tools/resources/prompts booleans.
- Transport union exactly one of `stdio` (structured executable+args+working directory) or `streamable_http` (HTTPS URL). Legacy SSE intentionally not in this candidate structural schema; protocol compatibility ADR is still OPEN.
- Secret binding allows only `env|http_header`, a binding name and a restricted-shape opaque `secret://ctmm/...` reference. It does not allow a credential-value property; no actual credential is present.
- Candidate limits (timeouts 500..300000 ms, concurrency 1..128, cardinalities, path/string lengths) are **REVIEW REQUIRED**, not accepted performance budgets. Do not rely on regex/pattern alone to secure an endpoint.
- Format choice is **JSON SCHEMA CANDIDATE ONLY**: backend persistence format is open (JSON/TOML/SQLite). The proposed API structure must not select its physical database.

## Trusted non-schema invariants — MUST BE IMPLEMENTED AND TESTED BEFORE P05
| ID | Check | Required failure |
|---|---|---|
| V-01 | Strict UTF-8 JSON parser detects duplicate object members *before* normal mapping; size/depth/input caps | REJECT input, no state change |
| V-02 | Stable server/profile identifiers, cross-object references and duplicate profile IDs/server IDs | REJECT on missing/duplicate IDs |
| V-03 | Alias collision within profile and globally published namespace, reserved built-in manager names, Unicode/confusable ambiguity | REJECT/explicit conflict resolution, no silent remap |
| V-04 | Binding target compatible with transport (`env` only stdio; `http_header` only HTTP), duplicate binding names normalized | REJECT secret misuse |
| V-05 | `secret_ref` resolves for the correct authorized principal/profile **only on protected host service**, never from chat or import | DENY/fail closed with redacted code |
| V-06 | `stdio` path absolute and permitted after canonicalization/symlink checks; args are argv, never a shell; sandbox/limits | REJECT unsafe executable and no launch |
| V-07 | HTTPS endpoint authority, TLS, DNS rebind, redirects, IP/metadata ranges, SSRF; explicit separately approved isolated-lab exception only | REJECT unsafe network target |
| V-08 | Caller identity/scope and per-change operator approval; expiry/nonce/replay and atomic revision/CAS | DENY/CONFLICT, record sanitized audit |
| V-09 | Import staging always disabled, profile remap explicit, no implicit prod scope, secret rebinding required | PREVIEW only; no auto enable/start |
| V-10 | Portable export omits secret values AND internal secret refs, runtime observation, auth headers, privileged paths as specified; generate local rebinding checklist | No credential leakage; unsupported export fails closed |
| V-11 | Atomic desired snapshot update + immutable routing generation; validated activation, bounded draining/in-flight behavior and safe restore | DEGRADED on failed restore; never fake READY |
| V-12 | Disconnect/upstream tunnel outage/GUI closed/DISPLAY absent | Local diagnostics and stored desired config remain usable |
| V-13 | Audit separate from immutable project Git+Obsidian dual logbooks; actor, safe canonical diff digest, prior/new revisions, result, recovery linkage | DENY if required audit sink unavailable (policy TBD), no secret logs |
| V-14 | Idempotency key replay: same payload may replay result; different payload with same key rejected | CONFLICT with no duplicate side effect |

## Dataflow (contract proposal, not executable)
```text
GUI (V1 release) / local CLI / ChatGPT tools (optional MCP Apps UI)
    -> authenticated service boundary
    -> propose(candidate, expected_revision, idempotency_key)
    -> strict parser + JSON schema + semantic/security/policy checks
    -> immutable proposal(diff digest, risk, expiry; NO side effect)
    -> independent operator approval for EACH diff (Q05A)
    -> apply: reauthorize+revalidate+CAS + durable desired-state generation
    -> router reconcile (new aggregate catalog generation)
    -> observed runtime snapshot/status + audit result
    -> on error: safe rollback attempt; otherwise DEGRADED + evidence
```
Pre-login boot is not a privileged mutation: it reads only approved persisted state and explicit policy, never auto-approves or migrates dangerous changes. The sample `boot_approved` enum is not proof that an operator approved its activation.

## Import/export boundary
- Portable export should use its **own versioned sanitized envelope** and `secret_requirements` (without `secret_ref` value), not pretend to round-trip the live Registry JSON. Its detailed schema belongs to RS-D03.
- Imported objects get new local IDs where needed; base `registry_revision` cannot be imported or overwritten by client, and `enabled` must become false when staged irrespective of the source file. A source identifier can be retained only as inert provenance if permitted.
- Local privileged sealed backup is a distinct feature requiring threat model, key lifecycle, disaster recovery and separate ADR; not a portable export.
- Reject unknown critical version, duplicate keys, unsafe URLs/paths, redaction failure, unauthorized reference creation and imported attempts to alter built-in management service.

## Offline structural smoke test actually executed
- Host: authorized Ubuntu `python3` one-shot, standard `jsonschema` package, no build/install/credential, no product services or network calls. Test evaluated immutable copies of the actual Git source schema and inert example.
- `Draft202012Validator.check_schema` accepted the schema; `Draft202012Validator(...).iter_errors` exercised the 17 exact synthetic cases listed below.
- Baseline example: ACCEPT. Negative cases all correctly REJECT: schema-version, negative-revision, uppercase-profile-id, missing-activation-policy, unknown-server-field, unknown-transport, http-extra-stdio-field, missing-stdio-executable, http-cleartext-url, secret-inline-value-field, malformed-env-name, timeout-zero, enabled-string, empty-name, unknown-root-field, invalid-exposure-value.
- Observed result: `SCHEMA_CHECK_VALID; CASES=17; MATCH=17; MISMATCH=0`. This is **structural-document validation only**; tests RS-01..RS-18 and V-01..V-14 are NOT RUN.
- Test limitations: `jsonschema` alone cannot reject duplicate raw JSON members after normal parsing, compare cross-array IDs, verify secret provenance, authenticate callers, enforce process/network policy or prove crash durability. No live endpoint was called.
- Repeatable offline method: feed both Git JSON documents to any conforming Draft 2020-12 validator; apply one mutation per negative scenario; ensure all 17 expected outcomes; use separate trusted parser for duplicate members and semantic harness for V-01..V-14.

## Potential ADR alternatives and acceptance gates (all OPEN)
- ADR-CTMM-006 storage: A atomic JSON/TOML with CAS+fsync+snapshot vs B SQLite transactions+schema migrations. Reject plaintext secret store. Benchmark/recovery design necessary before accept.
- Service identity and host secrets: machine/service account secure unlock at boot vs per-user secret store and mediated access. Must resolve Q03A/Q05A conflict explicitly.
- Local approval: privileged, OS-identified CLI vs authenticated GUI session vs external identity service, each with replay protections; always deny model self-approval.
- Profile identity and exposure: per-service global vs per-user mapped principals, to satisfy one aggregated Q07A endpoint without cross-profile leak.
- HTTP cleartext loopback testing exception: allow only documented isolated fixture after explicit authorization; do not globally weaken HTTPS candidate schema.
- P05 implementation gate: ADR-CTMM-001/002/004/005/006 choices accepted, stable Core+Proxy transport contracts, permission matrix reviewed, isolated profile and rollback evidence. Q08A live E2E requires its own authorization.
- RS-D01 can be marked **STRUCTURAL CANDIDATE REVIEWABLE**, not `DONE`/accepted: user still needs to decide the storage, host secret, profile and approval ADRs. RS-D02 permission matrix and RS-D03 export envelope remain follow-ups.

## References
- Draft 2020-12: https://json-schema.org/draft/2020-12/schema and https://json-schema.org/draft/2020-12/release-notes
- JSON member interoperability: RFC 8259 section 4 https://www.rfc-editor.org/rfc/rfc8259#section-4
- Project: `docs/requirements/V1-Q03-Q08-CONFIRMED.md`, `docs/adr/PROPOSED-ADR-CTMM-001-007.md`, `docs/architecture/REGISTRY-SETTINGS-PROPOSAL.md`.
