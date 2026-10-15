# CTMM Proxy — normative MCP 2026/legacy negotiation contract (correction addendum)

Event: CTMM-20261016-090. Status: PROPOSED / documentation-only; protocol/runtime tests NOT RUN.
Workstream: CHAT-05; V1-R03, R04, R05, R06, R08, R09; P00-01E research contribution; P04 BLOCKED. Prior event: CTMM-20261016-012.

## 0. Correction / provenance

Earlier CTMM proxy research and Obsidian note called server/discover "optional" without specifying which party. This was imprecise. For the 2026-07-28 spec:
- Every **server** that implements the modern revision MUST implement server/discover.
- A **client** MAY call server/discover, or call a method inline and handle UnsupportedProtocolVersionError. Dual-era stdio clients SHOULD probe first.
- On 2026 requests, _meta keys are namespaced: io.modelcontextprotocol/protocolVersion and io.modelcontextprotocol/clientCapabilities REQUIRED; io.modelcontextprotocol/clientInfo OPTIONAL but SHOULD normally be provided. Missing required metadata -> -32602, and on HTTP 400.
- Modern results MUST carry resultType; a client receiving earlier legacy result without resultType treats absence as complete. Do not mutate tool errors or misrepresent tool-level isError as protocol success/failure.
This addendum supersedes the ambiguous phrasing in CTMM-20261016-012 without rewriting that append-only logbook event. The prior test matrix remains a test *plan*; no tests have run.
Cross-workstream discrepancy: docs/architecture/SELF-MANAGEMENT-MCP-PROPOSAL.md also says "optional server/discover". Its owner should disambiguate server MUST/client MAY in its next paired change-set; this workstream does not alter that document.

## 1. Modern vs legacy entry decision (proposed CTMM adapter)

Modern M = protocol 2026-07-28 or later, per-request metadata; Legacy L = 2025-11-25 or earlier, initialize handshake. A server/process may implement both. The effective wire adapter must derive from explicit evidence, not "best guess" or a ChatGPT tool description.

### stdio, dual-era CTMM client
1. Start only a named isolated/authorized backend subprocess via allowlisted program/arguments. Send server/discover using modern version and correctly namespaced params._meta. Do not send tools/call during era detection.
2. DiscoverResult with supportedVersions: classify MODERN and select the newest mutually supported version allowed by policy; verify capabilities and handle cached ttl as permitted.
3. Recognized modern error (e.g. -32022 UnsupportedProtocolVersion with supported list): classify MODERN, choose a mutually supported modern revision or return precise unsupported-version error. Do NOT initialize/fallback.
4. Any other error or probe timeout (legacy server may use -32601 or -32602): select LEGACY path, send initialize and wait for initialized; if it fails, report diagnostic and mark backend BLOCKED/DEGRADED. Never silently count the probe as a valid business operation.
5. Cache era only for stable process lifetime, re-probe after process restart/config generation change. No additional auto-retry for side-effecting tool invocation. Version pinning policy remains an ADR question.

### Streamable HTTP, dual-era CTMM client
1. On an allowlisted HTTPS endpoint, send one compliant modern POST with MCP-Protocol-Version and Mcp-Method headers; when applicable Mcp-Name and Mcp-Param-* derived from actual body/schema. Supply required _meta and Accept/Content-Type per binding.
2. Success -> MODERN. HTTP 400 with recognized modern JSON-RPC error -> still MODERN; distinguish -32022 (reselect supported version), -32020 (header mismatch) and -32021 (client capability missing). Do not downgrade because a 400 occurred.
3. HTTP error with no recognized modern error body -> legacy candidate; only then attempt allowed legacy initialize transport. Deprecated two-endpoint HTTP+SSE is an independently gated OPTIONAL adapter, not automatic or default.
4. HTTP 404 with valid -32601 is a MODERN unknown-method response. Do not infer old SSE from code alone. Reject untrusted redirects, cross-origin credential propagation, unsupported backend origin and inconsistent HTTP routing headers.

## 2. Canonical modern request specimen (synthetic)

A legal example request for a modern server, not an on-host packet capture:

```json
{
  "jsonrpc": "2.0",
  "id": "proxy-discover-1",
  "method": "server/discover",
  "params": {
    "_meta": {
      "io.modelcontextprotocol/protocolVersion": "2026-07-28",
      "io.modelcontextprotocol/clientInfo": {
        "name": "ctmm-proxy-fixture",
        "version": "0.0.0-test"
      },
      "io.modelcontextprotocol/clientCapabilities": {}
    }
  }
}
```

HTTP equivalent request headers (synthetic only): MCP-Protocol-Version: 2026-07-28, Mcp-Method: server/discover. For tools/call add Mcp-Name from params.name. For non-header-safe values use the specification's exact =?base64?...?= sentinel; ambiguous literal sentinel must also be encoded. x-mcp-header is a mandatory-to-support client feature when a backend tool schema advertises it, and invalid definitions MUST be excluded from tools/list. Do not simply ignore this feature and claim full modern HTTP compliance.

## 3. Error and state oracle

| Condition | Classification | Expected CTMM observable response |
|---|---|---|
| no _meta required key, valid modern endpoint | INVALID_MODERN_REQUEST | JSON-RPC -32602, HTTP 400; no routed call |
| modern unsupported revision with recognized -32022 | MODERN_VERSION_MISMATCH | try only policy-allowed intersection; zero legacy initialize |
| modern HTTP header differs from body | HEADER_MISMATCH | HTTP 400 / -32020; no backend side effect |
| missing declared client capability | MISSING_CLIENT_CAP | HTTP 400 / -32021; do not alter auth |
| stdio discover returns legacy unknown-method error | LEGACY_CANDIDATE | initialize exactly once after probe, record both outcomes |
| stdio discover hangs | LEGACY_CANDIDATE after bounded timeout | initialize after abort, bounded total startup, no false READY |
| HTTP 400 with unrecognized body | LEGACY_CANDIDATE | authorized legacy initialize, no open network redirect |
| HTTP 404 with recognized modern -32601 | MODERN_UNKNOWN_METHOD | no fallback to old SSE |
| HTTP 404/405 with nonmodern body | LEGACY_TRANSPORT_CANDIDATE | only explicitly enabled legacy transport |
| unsupported modern resultType/extension | SEMANTIC_UNSUPPORTED | reject/capability suppress; don't fabricate result |
| backend offline while tunnel online | BACKEND_DEGRADED | management/status remains truthful; other backend routes work |

State reporting must distinguish "backend process alive", "protocol negotiated", "authorized route available" and "ChatGPT tunnel upstream connected": no single READY scalar can prove all four.

## 4. Revised deterministic acceptance probes (all NOT RUN)

| ID | Priority | Input/fault | Oracle |
|---|---|---|---|
| PN-01 | MUST | modern stdio server implements discover | DiscoverResult, modern classified, no initialize |
| PN-02 | MUST | modern stdio returns -32022 + supported | modern reselect only; legacy fallback count zero |
| PN-03 | MUST | legacy stdio server returns -32601 to discover | one initialize handshake; correctly gated READY |
| PN-04 | MUST | legacy stdio probe timeout | bounded initialize fallback, no indefinite process |
| PN-05 | MUST | modern HTTP returns 400/-32022 | never legacy fallback, retry common modern revision |
| PN-06 | MUST | modern HTTP missing _meta key | 400/-32602; no forwarding |
| PN-07 | MUST | header/body mismatch incl Mcp-Name | 400/-32020; zero side effect |
| PN-08 | MUST | HTTP 404 with valid -32601 body | distinguish modern method error from legacy HTTP+SSE 404 |
| PN-09 | MUST | modern tool x-mcp-header with valid string/number constraints | correct header routing and validation; no header injection |
| PN-10 | MUST | invalid x-mcp-header schema | omit invalid tool from catalog, log sanitized reason |
| PN-11 | MUST | resultType absent only from legacy | legacy complete semantics, no invented modern MRTR |
| PN-12 | MUST | unauthorized cross-origin redirect | no auth/token passthrough or metadata leak |
| PN-13 | MUST | two principals, two backend revisions | no version/capability cache bleed across tenant policy |
| PN-14 | MUST | backend process restart/config generation | re-probe and invalidate stale catalog, no automatic replay of side-effecting tool |

Dependencies: PN-01..04 can be fixture-tested only after protocol/runtime ADR and isolated harness permission; PN-05..14 add HTTP and security. Promote evidence from NOT RUN to PASS only with actual executed fixtures, exact pinned versions, sanitized traces and cleanup. The official ChatGPT tunnel E2E remains a separate test; do not claim full compatibility from offline mock alone.

## 5. Decision record inputs / handoff

Candidate A: pin official rmcp SDK plus explicit modern/legacy adapter + spec-derived header/metadata validator. Candidate B: modern-only V1 (smaller scope, fails legacy servers). Candidate C: custom transport/protocol implementation (audit burden). Candidate D: optional deprecated HTTP+SSE sidecar only with real inventory and explicit authorization.

No ADR accepted. The architectural owner must resolve: support matrix by actual upstream Secure MCP Tunnel revision, schema/header validation ownership, negative fallback security policy, per-origin/principal caching and proxy namespace mapping, SDK version/provenance, latency/resource bounds, service identity and security permissions. User confirmed Q07A one outward ChatGPT endpoint and Q08A forbids live E2E before separate authorization; document-only work here.

Test rollback later: stop only isolated fixture children; retain previous registry generation, no production stop. Document rollback now: paired correction event (no destructive log rewrite).

## Official primary references (verified 2026-10-16)

- https://modelcontextprotocol.io/specification/2026-07-28/basic/index
- https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning
- https://modelcontextprotocol.io/specification/2026-07-28/server/discover
- https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio
- https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http
- https://github.com/modelcontextprotocol/rust-sdk
