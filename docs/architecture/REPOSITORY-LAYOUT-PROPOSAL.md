# CTMM repository layout — proposal, NOT an accepted ADR
Status: PROPOSED ONLY. Event: CTMM-20261016-004.
Cargo workspace/packages must NOT be created until protocol, process and dependency boundaries are accepted.

## Directory map: documentation skeleton materialized in Git
```text
Chatgpt-tunnel-mcp-manager/
├── README.md
├── LOGBOOK.md
├── docs/
│   ├── requirements/V1-SCOPE.md
│   ├── architecture/REPOSITORY-LAYOUT-PROPOSAL.md
│   ├── engineering/MODULE-RULES.md
│   ├── plans/V1-MILESTONES.md
│   ├── workflows/CHAT-PROMPTS.md
│   ├── workflows/RECOVERY-AND-LOGGING.md
│   └── adr/README.md
├── crates/README.md               # Reserved, no Rust implementation
├── apps/README.md                 # Reserved, no app/service created
├── tests/README.md                # Reserved, no tests yet
├── fixtures/README.md             # Reserved, no real credentials
└── packaging/README.md            # Reserved, no installer/service
```

## Candidate layout AFTER architecture gate (not created)
```text
Cargo.toml                       # workspace only if multiple packages justified
Cargo.lock
crates/
  ctmm-core/src/                 # pure domain, state machines, registry, policy, contracts
    lib.rs
    lifecycle/{mod.rs,state.rs,transition.rs,retry_policy.rs}
    registry/{mod.rs,identity.rs,validation.rs}
    ports/{tunnel.rs,mcp_transport.rs,secret_store.rs,clock.rs}
  ctmm-mcp/src/                  # self-management MCP endpoints / tool contracts
    lib.rs
    management/{status.rs,diagnostics.rs,propose_change.rs}
  ctmm-proxy/src/                # protocol-aware multiplexer / adapters
    lib.rs
    transport/{stdio.rs,streamable_http.rs,legacy_sse.rs}
    routing/{registry.rs,namespace.rs,capability.rs}
  ctmm-adapters/src/             # controlled official tunnel client, process, OS storage
    lib.rs
    tunnel/{supervisor.rs,doctor.rs,version.rs}
    platform/{linux.rs,windows.rs}
apps/
  ctmm-service/src/main.rs       # no GUI dependency; service host
  ctmm-desktop/src/main.rs       # optional UI / tray client; candidate only
  ctmm-cli/src/main.rs           # optional local admin/debug client
tests/{contracts,integration,e2e}/
fixtures/{fake_mcp,protocol_matrix}/
packaging/{ubuntu,windows}/
```
This is **a responsibility map, not a promise of 4+ separate crates**. Start with the smallest cohesive Rust library after ADR; split only when dependency/platform boundaries demand it. Dependencies point inward to domain contracts, never domain -> tray/OS; external adapters implement core ports. An MCP server may be stdio and remote HTTP; transport selection is a capability/compatibility decision.

## Observability-first contract (draft)
All service functions emit typed status snapshots for local CLI/read-only interface and a GUI consuming the same data source. Candidate fields: `component_id`, `kind`, `version`, `state`, `observed_at`, `last_success_at`, `last_error_code`, `test_evidence_id`. Allowed status vocabulary to be refined via ADR: NOT_CONFIGURED, CONNECTING, READY, DEGRADED, FAILED, STOPPED, UNKNOWN. Status is not asserted healthy merely because a process exists.
ChatGPT management tools use the same snapshot semantics through access-controlled MCP. If tunnel is down, localhost status still works; no self-diagnosis dependency loop.

## Dependency decision gates
- D-ARCH-01: one crate vs workspace boundaries, Rust async/runtime and SDK.
- D-ARCH-02: process lifecycle/service account Linux/Windows, daemon permissions, IPC.
- D-ARCH-03: official tunnel-client process supervision vs embedding/reimplementation, license/provenance, version updates.
- D-ARCH-04: MCP protocol versions/transports, proxy routing, namespace conflicts and auth.
- D-ARCH-05: GUI toolkit shared across Linux/Windows, tray/Wayland limitations, headless.
- D-ARCH-06: secret storage, auth scopes, audit, action confirmation and rollback.
- D-ARCH-07: packaging, updates, distribution, test matrix.
All gates OPEN; create ADR in `docs/adr/` only with explicit accepted decision, alternatives and consequences.
