# CTMM Rust module & file rules (proposed engineering standard)
Status: PROPOSED FOR REVIEW; not accepted ADR. Event: CTMM-20261016-004.

## Partitioning
1. One module = one coherent responsibility / invariants. One file = a cohesive unit that can be understood and unit tested; **no rule forcing one function/algorithm per file**.
2. A nontrivial algorithm merits a separate `.rs` module when it has independent invariants, a distinct failure model, multiple implementations, meaningful tests, or predictable independent change. Helpers private to one algorithm should stay beside it.
3. Split by *reason to change*, not by arbitrary line count or technology layer alone. Prefer domain-specific names; avoid `utils.rs`, `misc.rs`, catch-all `common.rs` without explicit ownership.
4. Visibility minimal: private by default; `pub(crate)` for intentional internal contracts; `pub` only for a documented external API. No hidden import cycles or dependence on GUI types in core.
5. Stable domain types, explicit Result/error enums, typed lifecycle transitions, cancellation/retry policy, and configuration validation at boundaries. Do not hide network I/O inside pure algorithms.
6. A new crate only for genuine dependency/OS/security boundary, independent consumption or build/test requirement. Avoid speculative crate count and unnecessary abstraction.
7. Keep protocol schemas and public tool contracts versioned and backward-compatible; match MCP negotiated capabilities, not string-based latest guessing.
8. A module that can change externally visible behavior must link to requirement/task/ADR when relevant, have positive/negative/boundary tests, sanitized diagnostics and failure/rollback guidance.
9. Use `cargo fmt`, `cargo clippy`, `cargo test`, `cargo doc`, static secret scan and integration fixture gates after Cargo files actually exist. Do not mark these PASS for docs-only work.
10. Service != GUI: no GTK/Tauri/Windows GUI dependency in headless path; no panic/fatal log due to unavailable display. Inbound/outbound tool calls pass authz gate and journal independently of front end.

## Example of small algorithm boundaries (illustrative)
`retry_policy.rs` contains timing/backoff/jitter with deterministic clock/test RNG, but tiny bound-clamping helpers stay with the algorithm. `validation.rs` groups cohesive config invariants; separate a single validator into its own file only when complexity or independent change warrants it.

## Definition of Done for code tasks
Requirement ID and scope, dependencies, acceptance positive/negative/edge, permission boundary, unit/contract tests, visible status snapshot and sanitized evidence, operational rollback, code review, recorded commit and paired Git/Obsidian event, readback/sync. Any untested case clearly NOT TESTED.
