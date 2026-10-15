# CTMM operations and replayable logs — proposed procedure
Event: CTMM-20261016-004. Applies immediately to research/docs work; runtime journal is NOT IMPLEMENTED.

## Durable checkpoint workflow
1. Before any substantive action, verify latest Project-State + last event in both logbooks and Git HEAD; assign stable task ID and unique CTMM event ID, scope and operator. Record pre-state and test sandbox vs production boundary.
2. Write intent/checkpoint record **before potentially state-changing execution**: expected inputs as sanitized references, command class, targets, permission, acceptance tests, negative tests, rollback. For docs-only sessions keep a persistent session note.
3. Execute one bounded task; record real timestamps/exit codes/evidence, not assumptions. Label PENDING/RUNNING/BLOCKED/FAILED/VERIFIED distinctly.
4. Commit Git changes with `LOGBOOK.md` same event; append Obsidian `Logbook.md` with actual SHA, update `Project-State.md`; re-read and verify both. A failure on either side is `SYNC_PENDING`, and blocks moving to another task until repaired.
5. End every substantive chat with a concise decision+action summary linked to current task and source notes; link unresolved questions and next exact checkpoint. Not every token/message needs verbatim duplication; record outcomes, evidence, approvals, and operator-relevant conversations.
6. When interrupted/stuck: do not blindly repeat a command. Re-read current states, snapshots, git diff, operation status and retry idempotency. Retry/rollback only with reviewed safety preconditions.

## Later runtime journal specification, not yet coded
Append-only local `operations.jsonl` (or evaluated transactional DB) outside public repository and Obsidian; fields: op_id, task_id, event_id, at, actor, target, action, scope, requested_approval, before_hash, after_hash, status, result_code, sanitized_evidence_ref, rollback_ref. Restrict access; do not store raw credentials, secret args, HTTP authorization headers or whole tool payloads. Rotate/retain based on separate policy. `RUNNING` operations must reconcile on startup rather than be treated as completed. Sensitive materials use host secret store; use aliases in records.
Any service lifecycle mutation requires approved operator command, audit and rollback. AI can **propose**, but a deterministic server-side authorization and confirmation gate must approve any sensitive mutation.

## Current source-of-truth map
- Obsidian: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Project-State.md`, `Logbook.md`, and per-workstream notes.
- GitHub: `main`, `LOGBOOK.md`, approved `docs/adr/` and source when created.
- ChatGPT Project discussions are the coordination medium, not persistence of canonical state.
