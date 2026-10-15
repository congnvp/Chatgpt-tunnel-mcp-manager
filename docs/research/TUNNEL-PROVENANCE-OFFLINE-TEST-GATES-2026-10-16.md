# Secure MCP Tunnel V1: provenance, licensing and deterministic offline verification plan
Status: **PROPOSED / SOURCE-VERIFIED ONLY / RUNTIME NOT TESTED**
Event: **CTMM-20261016-061**; 2026-10-16 04:38:01 Asia/Bangkok
Project: CTMM / Tunnel workstream; task `CTMM-TASK-P00-01B` follow-on to `CTMM-20261016-022`.
Related: V1-R01/R02/R03/R08/R09, Q03A/Q04A/Q08A; ADR-CTMM-001 PROPOSED, ADR-CTMM-003/004/006 blocking.
No code, binary install/build/execution, credential use, live tunnel, proxy, GUI, production system or service change.

## Evidence pinned as of review

| Subject | Source and observed evidence | Verification status |
|---|---|---|
| Official `openai/tunnel-client` | [v0.0.16](https://github.com/openai/tunnel-client/releases/tag/v0.0.16), published 2026-10-06T22:04:53Z. Release 67 assets, including `SHA256SUMS.txt`, `tunnel-client-v0.0.16-provenance.sigstore.json`, platform SPDX 2.3 and license reports, enterprise evidence, full/runtime/runtime-cloudflared Linux/Windows binaries. | Release metadata and upstream README INSPECTED; archive bytes, checksum/signature/SBOM matching **NOT RUN**. |
| Official provenance process | [Official release validation runbook](https://github.com/openai/tunnel-client/blob/v0.0.16/README.md#validate-a-downloaded-release) documents tag and SHA256SUMS verification plus `gh attestation verify` with signer workflow `.github/workflows/release.yml`, source digest, SLSA provenance v1 and deny-self-hosted-runner policy. | PROCEDURE DOCUMENTED; local verification **NOT RUN**. |
| Official license | [LICENSE](https://github.com/openai/tunnel-client/blob/v0.0.16/LICENSE) Apache-2.0. Each runtime flavor has bundled dependency license reports and SPDX subject to binary-specific review. | TOP-LEVEL LICENSE INSPECTED; redistribution obligations and dependencies require downstream legal/license audit. |
| Third-party `zetaloop/otunnel` | [v0.2.0](https://github.com/zetaloop/otunnel/releases/tag/v0.2.0), published 2026-09-25T16:13:40Z, declares sync to official **v0.0.15**. Six Linux/Windows/macOS x86_64 and aarch64 archives have GitHub-provided asset SHA256 digests. | RELEASE METADATA INSPECTED; actual downloaded bytes **NOT VERIFIED**. |
| Rust license / dependency lock | [MIT license](https://github.com/zetaloop/otunnel/blob/v0.2.0/LICENSE), [Cargo.toml](https://github.com/zetaloop/otunnel/blob/v0.2.0/Cargo.toml), [Cargo.lock](https://github.com/zetaloop/otunnel/blob/v0.2.0/Cargo.lock) present with dependency versions/checksums. `aws-lc-sys`, `reqwest`, `rustls`, `tokio` are examples requiring transitive review. | TOP-LEVEL/LOCK CONTENT INSPECTED; third-party license, advisory audit and reproducible build **NOT RUN**. |
| Rust distribution workflow | [otunnel release.yml](https://github.com/zetaloop/otunnel/blob/v0.2.0/.github/workflows/release.yml) includes fmt, clippy and cargo check, `cargo dist`; that release workflow does not show a `cargo test` invocation. Six published release assets lack matching Sigstore provenance and SPDX/license sidecar assets in the inspected release asset list. | RELEASE WORKFLOW/ASSET LIST INSPECTED; not proof that no tests exist elsewhere or signatures are impossible. |
| Integration boundary | [otunnel lib.rs](https://github.com/zetaloop/otunnel/blob/v0.2.0/src/lib.rs) exports Rust `runtime::Tunnel` and `Snapshot`, but `health` is gated behind `cli` feature. [Official runtime flavors](https://github.com/openai/tunnel-client/blob/v0.0.16/README.md#narrow-runtime-artifacts) limit the operational API in narrow flavors. | SOURCE-BASED DESIGN INPUT ONLY; no ABI/build or health behavioral test. |

## Release-file manifest for future offline check

All hashes below are *GitHub release API claims*, not measured hashes of downloaded files.

| Artifact | Claimed SHA256 |
|---|---|
| `tunnel-client-runtime-v0.0.16-linux-amd64.zip` | `f8f48017e752810c2c6f02065cdc4ca1ad045b91a6d045cbab144b957fdcbdb1` |
| `tunnel-client-v0.0.16-linux-amd64.zip` | `d60cdba019bce451bcc3a15478cc5b9cb11270b049f5b56ea39a80b517f8b117` |
| `otunnel-x86_64-unknown-linux-gnu-v0.2.0.tar.gz` | `69f789fd8df156609ce26d2e004c5af80d6d9677df137220b25d48483174c415` |
| `otunnel-x86_64-pc-windows-msvc-v0.2.0.zip` | `8acae849794c4b0c5fb71304d27cf2b2f34b922beb94d6b190b649b8c8f3fcff` |

## Proposed verification workflow — execution NOT AUTHORIZED

**Environment:** separate unprivileged sandbox/VM/container, clean disposable checkout and package caches; outbound package download allowed only after operator approval; tunnel control-plane address must point only at a loopback fake endpoint. No production `tunnel_id`, runtime key, existing MCP process/service, real ChatGPT connector, or production host secret store. Prefer synthetic dummy credentials and dedicated fake MCP. No sudo, service installer, unit creation or system-wide reconfiguration.

1. **P01 Artifact admission (read-only input gate):** pin release tags, commit SHA, manifest and checksums; obtain archives, source, SBOM, Sigstore bundle only inside approved sandbox; verify hashes and upstream attestation. For otunnel, missing signed attestation is an explicit provenance policy exception requiring review or a reproducible trusted rebuild—not an automatic PASS.
2. **P02 Supply-chain gate:** obtain SBOM/license findings and advisory scan from reproducible exact lockfile; compare license reports of each official runtime flavor and Rust dependency licenses; record binary/source provenance separately. Do not equate MIT top-level license with dependency clearance.
3. **P03 Build & API gate:** build/check `otunnel` with exact `=0.2.0` (not README's stale `version="0.1"`), `--locked`, `default-features=false` and CLI variant; inspect `Tunnel` embedding, status `Snapshot`, host IPC adapter boundary. Record rustc version, host target, no secret/log leakage. Windows compile in later isolated cross-target environment.
4. **P04 Deterministic fake control plane:** static fake tunnel metadata + predictable `GET /v1/tunnels/{id}/poll` 204/200 and `POST .../response`; fake MCP supports modern and legacy initialize, tools/list, tools/call, controlled failure. Capture normalized request method/path, selected headers, request ID/channel, result/error class and local elapsed time. Never capture bearer/shard-token raw values in public evidence.
5. **P05 Differential comparison:** run official v0.0.16 and Rust v0.2.0 sequentially against same fake scenarios with separate isolated instance IDs and test tunnel IDs. Compare normalized semantics, not volatile timestamps/order of concurrent requests. Distinguish `supports`, `rejects`, `missing`, `not_observed`. Do not assume runtime-`health` parity from CLI-only endpoints.
6. **P06 Explicit decision:** if baseline tests and supply-chain gate pass, present evidence to user for ADR-CTMM-001 choice. Only thereafter plan authorized separate ChatGPT E2E with temporary restricted host credentials; no cutover or automatic upgrade.

### Acceptance/negative-case matrix

| Test ID | Trigger | Expected evidence; failure/negative assertions |
|---|---|---|
| OFF-01 | Unmodified artifact + matching exact digest | SHA256 and provenance report recorded; tampered byte/tag refuses admission |
| OFF-02 | `Cargo.lock` pinned build / no-default-features | Reproducible version and embedded API compile; unpinned / typo version blocked |
| OFF-03 | Fake poll 204 followed by 200 | 204 treated as empty success, subsequent work delivered; no busy-loop |
| OFF-04 | Correlation and shard token | Return request ID/channel and confidential shard token only over authorized header; no token in logs |
| OFF-05 | Fake MCP tool list/call | Deterministic tool catalog and nonce echo; malformed schema/error never presented as success |
| OFF-06 | Request expiration / slow MCP / cancellation | Deadline respected; do not execute expired work or upload late response |
| OFF-07 | 401/403 auth and 429 with Retry-After | Correct AUTH_DENIED vs RATE_LIMITED state and bounded backoff; 403 never reported READY |
| OFF-08 | Wrong-cluster correction and 404 delivery | Proper route correction, correctly classified terminal response 404 (not automatically 200); no duplicate execution |
| OFF-09 | Disconnect mid-poll and mid-response POST | Bounded reconnect and ambiguous completion semantics; **no replay** of unsafe mutations after uncertain delivery |
| OFF-10 | 2026-07-28 vs legacy initialize/initialized | Negotiated expected tool calls; mismatches rejected cleanly without protocol contamination |
| OFF-11 | Official optional `resp_timing` | Official publishes timing only for supported terminal results; Rust 0.2.0 may omit it; omission tagged optional GAP, never fabricated |
| OFF-12 | Official W3C `traceparent/tracestate` | Default off; opt-in valid trace forwarded only as spec; invalid field/baggage ignored; Rust unsupported profile gets explicit validation error |
| OFF-13 | Headless and health status | No DISPLAY; local health distinguishes live, ready and observed MCP/response delivery; Rust embedded mode supplies own adapter |
| OFF-14 | Multi-MCP aggregated endpoint (separate future test) | One ChatGPT-facing catalog exposes two independent fake MCPs with stable namespace; no cross-profile secrets |
| OFF-15 | Disconnect/stop fake server + rollback | Both clients cease new work, show accurate degraded status; only sandbox resources torn down |
| OFF-16 | Cleanup & legacy isolation | Diff sandbox inventory, restore test profile, revoke any test-only secret if provisioned; original tunnel/proxy/service unchanged |

**Acceptance truth:** OFF-01..OFF-16 are **NOT RUN**. No `PASS`, `DONE`, `COMPATIBLE`, `PROVENANCE VERIFIED` or production-safe conclusion is inferred from static code reading or release metadata.

## Decision matrix for ADR-CTMM-001 (ALL OPEN)

- **Alternative A — official pinned Go process supervised by Rust:** strongest upstream release verification and reference compatibility; cost is second runtime/process supervision, IPC boundaries and packaging complexity.
- **Alternative B — pinned third-party Rust otunnel in-process:** Rust-native ownership, fewer runtime processes; cost is external maintainer trust, source-feature gaps, uncertain binary attestation and strict Rust dependency audit.
- **Alternative C — own minimal Rust transport:** maximum control and responsibility, large protocol/auth/retry/conformance burden and upgrade risk. Not justified without evidence A/B cannot satisfy requirements.
- **Recommendation (PROPOSED):** maintain A as compatibility oracle and B as preferred Rust research candidate. Do not accept B before OFF-01..OFF-13 evidence and operator acceptance; C deferred. Rollback is reverting only isolated test release/profile/adapters, never modifying active production tunnel.

## Ownership, blockers and next checkpoint

- **Tunnel workstream:** OFF-01..13 fixture specification and source/version diff; **QA/Security:** sandbox authorization, software provenance, third-party dependencies and secret isolation; **Core/Architecture:** embedding status contract, boot identity/IPC; **PM:** update DAG and gate, decide ADR with user; **Operator:** separate explicit approval for any sandbox binary execution and later live ChatGPT E2E.
- Q08A grants **future intent**, not current permission. P00 documentation is allowed; P01/P02 runtime/code are blocked until accepted ADRs and appropriately scoped authorization.
- If any test or artifact fails: mark `BLOCKED/FAILED` with sanitized evidence, classify ownership and stop without retesting against production; rollback only sandbox assets after approval.
- Earlier dual-log issue CTMM-20261016-030 was observed with Git `SYNC_PENDING`; later Obsidian 030 entry was present on 2026-10-16 readback. This is documentation visibility, **not** evidence of runtime correctness.

Source references: the exact immutable URLs in table above, [official license](https://github.com/openai/tunnel-client/blob/v0.0.16/LICENSE), [Rust license](https://github.com/zetaloop/otunnel/blob/v0.2.0/LICENSE), [official wire protocol](https://github.com/openai/tunnel-client/blob/v0.0.16/docs/protocol.md), [prior CTMM-022 research](https://github.com/congnvp/Chatgpt-tunnel-mcp-manager/blob/main/docs/research/OTUNNEL-V020-VS-OFFICIAL-V0016-2026-10-16.md).
