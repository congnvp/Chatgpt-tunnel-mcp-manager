# CTMM Proxy workstream — MCP protocol survey and fake-server test matrix

Event: CTMM-20261016-012. Status: PROPOSED research, NOT AN ACCEPTED ADR; verified online documentation 2026-10-16, runtime tests NOT RUN.
Scope: V1-R03/R04/R05/R06/R08/R09; P00-01E research, future P04 protocol proxy. No source code, credentials, tunnel/client/service changes.

## Findings from official specifications

Current stable MCP revision: 2026-07-28 (official releases). Major incompatibility boundary: revisions <=2025-11-25 use initialize + notifications/initialized, session-scoped capabilities and versions. Revision 2026-07-28 instead requires per-request params._meta keys io.modelcontextprotocol/protocolVersion and io.modelcontextprotocol/clientCapabilities; modern SERVERS MUST implement server/discover, while CLIENTS MAY call it (dual-era stdio clients SHOULD probe). Modern transport has no protocol-level session and uses request-scoped streams/subscriptions/listen and multi-round-trip result shapes. See corrective addendum PROXY-NEGOTIATION-CORRECTION-2026-10-16.md. A JSON-RPC forwarding layer alone is not a compatibility layer.

Transport contract:
- stdio: one UTF-8 JSON-RPC frame per line on stdin/stdout; child stderr is logging. Never pass non-MCP text on stdout. Isolate subprocess and bound stdout, stderr, restart, resource usage.
- Streamable HTTP: single POST endpoint; response is JSON or request-scoped SSE; validate Origin, protocol version, standard routing headers. In 2026-07-28 Mcp-Method and Mcp-Name mirror message semantics, no session ID or GET notification stream; closing SSE response cancels its request.
- Legacy HTTP+SSE (two-endpoint): feature deprecated and only conditionally proposed for V1; official Rust rmcp README excludes the old transport. Do not mark supported absent fixture and evidence.
- Rust rmcp repository advertises stable 2026-07-28 and legacy protocol support. This is an SDK vendor declaration; no CTMM compilation, pinned dependency/security review or runtime conformance has occurred.

## Architecture contract proposal (not approved)

Client-facing transport adapter -> independent server-side authn/authz -> per-request protocol/capability adapter -> filtered catalog + stable reversible namespace -> router with request correlation -> per-backend transport adapter (stdio, Streamable HTTP, optional old SSE) -> managed fake or third-party MCP servers.
Local diagnostics/supervision must work with no GUI/DISPLAY and when upstream tunnel is offline.

1. Negotiate modern-vs-legacy by documented probe and recognized error. For stdio, server/discover first; fall back to initialize only when signal indicates legacy (not a modern UnsupportedProtocolVersionError); for HTTP inspect modern error semantics. Track backend era and actual common version; do not claim support from version strings alone. For unknown revision, fail closed with diagnosable error.
2. Effective exposed capabilities = client-compatible AND locally implemented AND backend-advertised AND principal-permitted. Only expose tools/list, tools/call, prompts/list/get, resources/list/read/templates, subscriptions and optional extensions if their complete semantics are implemented. Do not use unauthenticated serverInfo, annotations or tool descriptions as authority.
3. Candidate namespace: server_id + '__' + original_name for tools and prompts; stable mapping, allowlisted exported characters, collision detection and deterministic rejection when map is not reversible. IDs do not silently change on reconnect. For resources, use a unique, canonical public URI mapped to (backend_id, original_uri), validate URI/path/traversal and enforce per-resource permissions. Preserve provenance without exposing restricted identifiers.
4. Preserve JSON-RPC id mappings scoped per requesting client + request + backend; sanitize status/errors without merging tool-level isError and JSON-RPC error. Preserve supported outputSchema/structuredContent types, pagination cursors and metadata. Translation of 2026 MRTR to 2025 server-initiated flows and subscription semantics must be explicitly implemented/tested or rejected with an incompatibility error; never simulate success.
5. Authentication boundary: upstream credentials are for CTMM only, not for arbitrary downstream servers. HTTP backend gets independently issued, resource-/audience-bound credential from host secret store; issuer, audience, scope, expiration, principal and tenant checked before any routing. stdio gets separate, allowlisted environment references. No token passthrough, no secret in logs/config/export. Enforce policy server-side even when model requests writes, reject unsafe HTTP origins/redirects/SSRF. API authorization does not follow descriptions in untrusted tool metadata.
6. Process/timeouts: one isolated child supervision unit per stdio backend, no shell interpretation, bounded CPU/RAM/FD/process count, maximum concurrent requests, body/frame limits; separate connect/discover/call/idle/cleanup deadlines, circuit breaker and capped restart. Never replay potentially side-effecting tool call without idempotency evidence; cancellation must clean resources. Offline state, failed backend and healthy backend state remain distinct.
7. Registry update: validate a new immutable routing generation before atomic swap; in-flight requests retain generation and authorizations; rollback to previous generation on error. Catalog caches obey TTL/private scope and authentication partition; no stale READY.
8. Deterministic observation: fake clock, trace/request IDs, redacted evidence, transport/revision/backend/permission snapshot, expected vs actual and exit codes for each test. Feature gate remains BLOCKED until architecture/user approval and isolated P02 fixture E2E.

## Fake server fixture descriptions (not implemented)

- fake-modern: spec 2026-07-28, both stdio and Streamable HTTP; server/discover; echo, search; one resource and prompt; ttl/cacheScope and subscriptions/listen; injected header/method errors, MRTR and version rejection.
- fake-legacy: 2025-11-25 initialize/initialized, both stdio and Streamable HTTP; deterministic tools, resources and prompts, stale session and invalid negotiation.
- fake-collision: same tool search, prompt greet and original resource URI exposed by two backends; intentionally invalid names and namespace overlap.
- fake-untrusted: malformed JSON/JSON Schema, stdout garbage, tool instruction injection, unauthorized URI/path, oversized result and deliberately misleading serverInfo.
- fake-failure: controlled latency, hangs, SIGTERM/crash, EOF, partial SSE, HTTP 429/503, restart storm; synthetic clock, no real tunnel touched.
- fake-auth: two principals, separate tenants and scope/audience; expired token, redirect and denial fixtures; all credentials synthetic.

## Deterministic test matrix — expected behavior only; every case NOT RUN

| ID | Setup / fault | Expected result | Gate |
|---|---|---|---|
| PX-01 | modern server/discover | modern version/capability recognized, no legacy initialize | MUST |
| PX-02 | legacy initialize + initialized | session-era READY only after correct handshake | MUST |
| PX-03 | no common version | deterministic unsupported-version failure; backend not called | MUST |
| PX-04 | modern upstream to legacy echo | only semantically equivalent operation works; incompatible capability suppressed | MUST |
| PX-05 | two servers tool search | unique stable namespace, route to correct backend | MUST |
| PX-06 | same resource URI across servers | canonical mapping returns correct backend data, respects ACL | MUST |
| PX-07 | prompt greet collision | original args validated and routed correctly | MUST |
| PX-08 | invalid tool JSON Schema args | reject before side effect | MUST |
| PX-09 | paginated tool listing | complete, stable order, no duplicated/skipped entries | MUST |
| PX-10 | missing advertised prompt capability | do not advertise or forward prompts | MUST |
| PX-11 | tool returns isError | preserve tool-error semantics, not fabricated transport fault | MUST |
| PX-12 | JSON-RPC error codes | preserve class and correct correlation | MUST |
| PX-13 | backend request timeout | bounded duration/cleanup, others unaffected | MUST |
| PX-14 | child crash mid-flight | only its backend DEGRADED; siblings respond | MUST |
| PX-15 | stdout non-protocol contamination | framing validation rejects bad output, captures stderr separately | MUST |
| PX-16 | close HTTP SSE response | cancellation, zero writes after cancel, cleanup | MUST |
| PX-17 | wrong Origin/header/body mismatch | deny without forwarding | MUST |
| PX-18 | expired/wrong audience/scope/tenant | 401/403 as applicable; no token replay or leak | MUST |
| PX-19 | tool metadata prompt injection | policy and permissions unchanged | MUST |
| PX-20 | disabled backend | disappear from catalog, new call denied | MUST |
| PX-21 | cache ttl/private scope and principal change | no cross-principal cache bleed or stale READY | MUST |
| PX-22 | same RPC id from concurrent clients | no cross-client reply mixup | MUST |
| PX-23 | legacy two-endpoint HTTP+SSE | SKIP until compatibility ADR plus fixture | OPTIONAL |
| PX-24 | upstream/tunnel offline | local read-only diagnostics still available | MUST |
| PX-25 | subscriptions/listen | only subscribed notifications correlated to correct request/principal | MUST if exposed |
| PX-26 | MRTR input_required and legacy bridge | supported only if semantic adapter proven; otherwise explicit deny | MUST if exposed |
| PX-27 | oversized frame and restart storm | bounded resource use, circuit breaker | MUST |
| PX-28 | SSRF/redirection/token exfil attempt | deny and redact | MUST |
| PX-29 | array structuredContent and outputSchema | 2026 data shape preserved, no silent coercion | MUST |
| PX-30 | invalid registry reload | atomic rejection, old generation still routes | MUST |

Evidence format for future execution: case_id, fixture_id, revision, transport, UTC+07 timestamp, client principal alias, policy revision, seeded request and expected status, sanitized trace/exit code, actual result, verdict PASS/FAIL/BLOCKED/SKIP, cleanup proof. At this time all PX cases are NOT RUN. No conformance, security, performance, or ChatGPT E2E claim.

## ADR options and workstream handoff

A (candidate): official rmcp pinned + explicit modern/legacy compatibility adapters, stdio and Streamable HTTP as V1 targets; legacy two-endpoint HTTP+SSE deferred. B: pin legacy-only (smaller short-term scope, excludes current stateless protocol). C: custom protocol implementation (larger audit burden). D: isolated legacy HTTP+SSE sidecar only after requirement evidence. No option accepted.

Blocking questions: prove actual upstream tunnel MCP revision on authorized P02 fixture; approve supported history, legacy SSE inclusion, namespace/URI contract, per-principal authentication and secret store, limits/cancellation/rollback, selected SDK release and provenance. Owner PM/Architecture must review; no ADR authored as ACCEPTED here.
Rollback of this workstream now is documentation correction event only. Later tests use isolated ports/profiles/processes and never alter existing tunnel/proxy.

Official references (inspected 2026-10-16):
- https://github.com/modelcontextprotocol/modelcontextprotocol/releases
- https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning
- https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio
- https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http
- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- https://modelcontextprotocol.io/specification/2026-07-28/server/tools
- https://modelcontextprotocol.io/specification/2026-07-28/server/resources
- https://modelcontextprotocol.io/specification/2026-07-28/server/prompts
- https://github.com/modelcontextprotocol/rust-sdk
- https://developers.openai.com/api/docs/guides/secure-mcp-tunnels
