# Rust tunnel compatibility desk audit — otunnel 0.2.0 vs OpenAI tunnel-client 0.0.16

- Event: **CTMM-20261016-022** (2026-10-16 04:26:42 Asia/Bangkok)
- Workstream: Tunnel / CTMM-TASK-P00-01B; V1-R01/R02/R03/R08/R09; Q03A/Q04A/Q08A; candidate ADR-CTMM-001
- Status: **SOURCE-AUDITED, COMPATIBILITY UNVERIFIED, ADR PROPOSED ONLY**.
- Non-goal: NO Rust product code, no third-party binary execution, no tunnel or existing production service activity, no keys.
- Baselines (immutable tags): [Rust otunnel v0.2.0](https://github.com/zetaloop/otunnel/releases/tag/v0.2.0) 2026-09-25 (tag commit `3e90fe1952a4f2cbc41808a5614d843a9a24f8bc`); [official Go tunnel-client v0.0.16](https://github.com/openai/tunnel-client/releases/tag/v0.0.16) 2026-10-06 UTC. otunnel v0.2.0 release notes explicitly claim alignment with **official v0.0.15**, not v0.0.16.
- Definitive upstream change list: [official v0.0.15...v0.0.16 compare](https://github.com/openai/tunnel-client/compare/v0.0.15...v0.0.16), [v0.0.16 wire protocol](https://github.com/openai/tunnel-client/blob/v0.0.16/docs/protocol.md#optional-response-timing-and-trace-context), [configuration](https://github.com/openai/tunnel-client/blob/v0.0.16/docs/configuration.md).
- Exact source audit is LIMITED to files cited in this document and release metadata. Absence in reviewed code does not establish complete absence across every possible runtime behavior. Live compatibility requires separately authorized tests.

## Compatibility and gap register

| Gap | Source-backed comparison | Impact | Classification |
|---|---|---|---|
| TUN-R01 Response timing | Official v0.0.16 permits optional `resp_timing` on terminal JSON-RPC result/error, with duration from downstream transport+server+SDK/backpressure. Rust [protocol.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/protocol.rs) `Reply` and [control.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/control.rs#L605-L625) response POST payload have no `resp_timing`; timing fields not found in these reviewed paths. | Observability parity GAP; no evidence baseline forwarding fails, because metadata is optional. | MISSING_IN_REVIEWED_RESPONSE_PATH; runtime NOT TESTED |
| TUN-R02 W3C trace forwarding | Official v0.0.16 `mcp.forward_trace_context` / flag / env (default FALSE) optionally forwards valid `traceparent`+`tracestate` in MCP request `params._meta`. Rust [config.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/config.rs#L509-L522) `Mcp` settings lack field; reviewed [http.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/transport/http.rs) and [pipe.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/transport/pipe.rs) contain no forwarding logic. | Distributed-tracing feature GAP. Do not assume traffic equivalence for opted-in profiles. | MISSING_IN_REVIEWED_CONFIG/TRANSPORT; runtime NOT TESTED |
| TUN-R03 Versioned config compatibility | Rust [config.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/config.rs#L14-L37) deserializes fields with `deny_unknown_fields`; v0.0.16 profile YAML containing `mcp.forward_trace_context` is expected to fail validation rather than silently accepted. Exact error wording not tested. | Needs profile-version/feature negotiation and a negative config test; fail closed preferable to dropping policy silently. | SOURCE-DERIVED EXPECTATION; NOT RUN |
| TUN-R04 Rust embedded health | Rust [lib.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/lib.rs#L7-L29) conditionally compiles `health` behind `cli` feature; README embedding with `default-features=false` therefore does not provide the HTTP health module. `runtime::Snapshot` remains exported. Full CLI [health.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/health.rs) implements /healthz,/readyz,/health/* and metrics. | CTMM embed adapter must export a separately authenticated/local read-only status contract, or enable an appropriate feature after security review. | CONFIRMED SOURCE FEATURE BOUNDARY; NOT RUN |
| TUN-R05 Rust MCP/wire baseline | Rust [protocol.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/protocol.rs#L13-L22) declares MCP `2026-07-28`, wire `2026-08-25`, five protocol revisions; [runtime.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/runtime.rs#L421-L545) implements channel dispatch/expiry; [control.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/control.rs#L605-L686) has HTTPS response delivery/retry and shard token. | There is a plausible Rust transport implementation, not proof of conformance with v0.0.16 or the target ChatGPT workspace. | SOURCE PRESENT; BEHAVIOR UNVERIFIED |
| TUN-R06 Package provenance / dependency chain | Official v0.0.16 publishes SHA256SUMS, Sigstore provenance, SPDX and license evidence. Third-party otunnel v0.2.0 release contains six Linux/macOS/Windows archives with GitHub API SHA256 digests; no corresponding Sigstore/SPDX release assets in enumerated release. Its [CI workflow](https://github.com/zetaloop/otunnel/blob/v0.2.0/.github/workflows/release.yml) checks fmt/clippy/cargo check and packages; this is not equivalent to signature, SBOM or interoperability verification. | Hash verify binary/source, audit cargo deps, provenance/attestation/SBOM policy; third-party supply-chain risk OPEN. | RELEASE METADATA DOCUMENTED; BYTES/BUILD NOT VERIFIED |
| TUN-R07 README dependency pin mismatch | Rust [v0.2.0 README](https://github.com/zetaloop/otunnel/blob/v0.2.0/README.md#library) still illustrates Cargo dependency `version = "0.1"`; v0.2.0 [Cargo.toml](https://github.com/zetaloop/otunnel/blob/v0.2.0/Cargo.toml#L5-L14) advertises `version = "0.2.0"`. Because of pre-1.0 semver boundaries, `version="0.1"` does **not** select v0.2.0. | Must pin explicit exact version `=0.2.0`/commit after ADR and verification; do not copy example uncritically. | CONFIRMED SOURCE DOC MISMATCH |
| TUN-R08 Windows & boot | Rust crate has cfg(windows) dependencies and release assets for Linux/Windows. Neither release asset metadata nor Cargo check proves system service boot-before-login or a Windows Service implementation. | Separate isolated Ubuntu boot/no-DISPLAY and later Windows parity tests remain mandatory. | BUILD TARGETS ADVERTISED; SERVICE NOT VERIFIED |

## Compatibility verdict

1. **Usable as a Rust candidate? YES, for controlled evaluation.** Wire/API foundations and multi-platform distribution exist; Rust `Tunnel` embeds in Tokio, avoiding a Go sidecar as an architectural possibility.
2. **Verified substitute for official v0.0.16? NO.** Upstream parity statement ends at v0.0.15; v0.0.16 response timing/trace features are missing in reviewed paths; no isolated fake-control-plane tests or authorized ChatGPT E2E executed.
3. **Blocking for baseline MCP echo? NOT DEMONSTRATED.** Upstream states `resp_timing` is optional and trace forwarding is opt-in off by default. Basic forwarding may still work, but never mark PASS from documentation alone.
4. **Selection status:** retain ADR-CTMM-001 as PROPOSED. Evaluate A = official pinned Go sidecar, B = third-party Rust `otunnel` pinned and audited, C = clean-room Rust wire implementation. Do not convert user preference for a Rust core into an implicit approval to adopt third-party code.

## Test gates (design only; none executed)

| ID | Dedicated isolated fixture and acceptance (positive + negative) |
|---|---|
| RUST-TUN-01 | Provenance: lock exact otunnel tag/commit and official binary, audit SHA256, Rust crates and release metadata; reject altered bytes or mismatched tag/digest. |
| RUST-TUN-02 | Compile: cargo check/test/clippy for Linux and later Windows in **sandboxed clean external-source workspace**, not CTMM product tree; record versions and lockfile; fail if source requires unexpected privileges/network. |
| RUST-TUN-03 | Fake control-plane: deterministic HTTP poll 204/200, shard/correlation, response POST 200/404, 401/403, 429, wrong-cluster, close, cancel; compare Rust vs official normalized traces. |
| RUST-TUN-04 | MCP transport: fake stdio and Streamable HTTP version negotiation, initialize/notifications, tools/list/tools/call, timeout, streaming and session affinity. |
| RUST-TUN-05 | New v0.0.16 delta: optional resp_timing presence/omission; opt-in valid/malformed traceparent, tracestate, default-off, no baggage; reject v0.0.16 config field on v0.2.0 Rust with explicit error. |
| RUST-TUN-06 | Offline / reconnect / backpressure: no network, 401/403, MCP crash, queue pressure, expired request; truthful local status without GUI, bounded retry, no replay of ambiguous unsafe mutation. |
| RUST-TUN-07 | Only after operator separate approval: dedicated tunnel_id + short-lived host-secret-store key + target ChatGPT workspace; same unique fake echo nonce and tools/list/tools/call observed from ChatGPT and sanitized host logs. |
| RUST-TUN-08 | Verify existing tunnel/proxy/service before and after and safe rollback: stop/revoke only dedicated test assets, no changes to production. |

## Recommendation / ADR-CTMM-001 addendum (PROPOSED only)

- P00: maintain official Go v0.0.16 as **reference/oracle**, third-party Rust v0.2.0 as **candidate**; avoid selecting either as implementation before comparable proof.
- P01: regardless of tunnel choice, CTMM Rust core owns typed status/security policy; loopback read-only status and boot credential access are independent ADR gates.
- P02 gate: for Rust-only deployment require the vendor-source delta tests above, provenance/security review, explicit operator approval of third-party dependency and isolated E2E. If gap remains high risk, official client sidecar is fallback without abandoning a Rust core.
- Upgrade/rollback: lock exact official/otunnel versions; config compatibility matrix; atomic test-profile revert; never auto-switch running tunnel/client or reuse original tunnel_id.
- No developer or end-user approvals are inferred. No runtime, compiler, credentials, HTTP request to a tunnel endpoint, or ChatGPT E2E testing performed.

## Evidence & limitations

Directly inspected (read-only) immutable GitHub tagged source and release metadata on 2026-10-16 04:26:42 Asia/Bangkok; source tree, config, protocol, control, runtime, transport, health, CI/workflows. No local clone/download/build, private secrets, current host inspection, network authorization or binary runtime evidence. Source-based expectations need negative tests before VERIFIED designation.
