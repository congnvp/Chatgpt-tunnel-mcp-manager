# CTMM multi-chat Project prompts — copy into dedicated chats
Status: working prompts, no automated execution. Event: CTMM-20261016-004.
Every prompt below must obey the global project policy. Source of truth: freshest Obsidian Project-State/Logbook + Git main/LOGBOOK, not chat memory. No tunnel/proxy/service changes without explicit separate approval.

## Shared header (paste at top of EVERY workstream chat)
```text
CTMM / USES. Trước khi làm: xác minh @Ubun và @GitHub, đọc
00_AI-Logbook/00_BOOT.md, 01_ROUTER.md, hai note USES,
folder note CTMM, Project-State.md, hai LOGBOOK, task/ADR liên quan.
Đối chiếu HEAD, event cuối, trạng thái ghi nửa chừng (SYNC_PENDING).
Chỉ dùng kết quả đọc hiện thời, không suy từ lịch sử chat.
Phạm vi chat này chỉ về [WORKSTREAM]. Chưa viết code hay tác động
service/tunnel/proxy cho đến khi gate phù hợp được chấp thuận.
Mọi phát hiện, trao đổi quyết định, hành động và bài test có biên bản
có Event-ID CTMM-YYYYMMDD-NNN, timestamp Asia/Bangkok, đường dẫn,
đầu ra đã lọc bí mật, status và next step. Khi tạo/sửa nội dung dự án
phải cập nhật cả Git LOGBOOK.md và Obsidian Logbook.md cùng Event-ID,
Git commit trước, rồi Obsidian SHA thực tế, đọc lại xác minh.
Mỗi lần chỉ lấy một task READY nhỏ với acceptance, success/failure
tests, evidence, rollback; cập nhật Project-State và bàn giao.
Kết thúc báo: Verified -> Decision -> Changed -> Tests ->
Dual-log sync -> Blockers -> Next task. Nếu stuck ghi blocker cùng
checkpoint/attempts và dừng thao tác rủi ro.
```

## CHAT-00 — Điều phối và trạng thái
```text
[WORKSTREAM = Integration/PM] Đọc Project-State, logbook đôi,
các ADR và workstream updates. Dựng dependency graph của các
task và chọn 1 task READY ngắn nhất không bị blocked.
Không tự code; phát hiện xung đột nhánh, event, acceptance,
quyền hoặc secret; ghi tiến độ và giao việc qua hồ sơ.
```

## CHAT-01 — Yêu cầu / kiến trúc
```text
[WORKSTREAM = Requirements/Architecture] Phân rã V1-R01..R09,
phân biệt MUST cho V1 và tùy chọn; làm rõ yêu cầu bằng một
block câu hỏi thiết yếu. So sánh các lựa chọn chưa chốt,
đề xuất ADR có alternatives/tradeoffs/test/rollback;
không chốt thay tôi. Trả lại dependency map + acceptance gates.
```

## CHAT-02 — Secure MCP Tunnel
```text
[WORKSTREAM = Tunnel] Khảo sát official tunnel-client mới
nhất đã kiểm chứng và đường kết nối ChatGPT Secure MCP Tunnel;
kiểm tra tunnel_id, quyền org/workspace, doctor/health,
reconnect, license, updates. Thiết kế profile kiểm thử mới
với fake MCP và credentials tạm chỉ qua host secret store.
Không chạm tunnel đang chạy. Đề xuất test thành công và
lỗi/offline với bằng chứng đầu-cuối ChatGPT.
```

## CHAT-03 — Rust core / service lifecycle
```text
[WORKSTREAM = Rust core/Service] Đánh giá state machine,
service auto-start và retry, component supervision, Linux
systemd vs Windows Service, IPC và chạy không DISPLAY.
Thiết kế hợp đồng trạng thái typed, local health endpoint,
clock/fake adapters và tests; UI không được là dependency.
Chưa tạo Cargo crates nếu các ADR blocking chưa duyệt.
```

## CHAT-04 — MCP quản trị CTMM
```text
[WORKSTREAM = Self-management MCP] Thiết kế MCP tool status/
diagnostics/list_servers/selftest trước, offline fallback;
sau đó propose/approve/apply add-enable-disable cùng auth,
audit, least privilege, rate-limit và rollback.
Tạo schema, negative test, permission matrix, fake outputs.
Không cấp model quyền ghi mà không có server-side gate.
```

## CHAT-05 — MCP Proxy / interoperability
```text
[WORKSTREAM = Proxy] Khảo sát MCP protocol versions và
transport stdio, Streamable HTTP, legacy SSE (nếu cần);
thiết kế capability negotiation, namespace, collision,
resource/prompt/tool routing, auth boundary, process isolation,
timeouts, errors. Tạo test matrix server giả lập trước.
Không hứa tương thích mọi phiên bản khi chưa có evidence.
```

## CHAT-06 — MCP registry / per-server configuration
```text
[WORKSTREAM = Registry/Settings] Định nghĩa schema cấu hình
mỗi MCP server, validation, profiles, env secret references,
enable/disable, import/export đã lọc bí mật, rollback, audit.
Phân loại fields read-only / local editable / AI-proposed.
Đề xuất UI ChatGPT MCP Apps tùy chọn nhưng tools chạy headless.
```

## CHAT-07 — Linux & Windows GUI / tray
```text
[WORKSTREAM = GUI/Tray] So sánh toolkit GUI xuyên nền tảng,
Wayland/AppIndicator/Windows tray, UX một-click vs menu,
state polling/subscription từ Rust service, màn settings.
Dùng mock snapshots để dựng panel nhìn thấy được sau khi
ADR được duyệt; không import PoC C đã làm vào sản phẩm.
Luôn test GUI absent, close/crash GUI, service vẫn READY.
```

## CHAT-08 — QA, security, packaging & release
```text
[WORKSTREAM = QA/Security/Release] Định nghĩa isolated
fixtures, CI, status truth tests, TLS/authz, secret storage,
untrusted third-party MCP, permissions, provenance/pinning
binary, AppImage vs systemd package, Windows installer,
upgrade/rollback. Chỉ thực thi tests trên dedicated profiles;
không thay đổi production trước approval gate.
```

## CHAT-09 — Resume / stuck / incident
```text
[WORKSTREAM = Recovery] Không suy đoán chat trước. Read
Project-State/Logbook Git+Obsidian và ledger operation;
lấy last completed checkpoint, current RUNNING/FAILED tasks,
HEAD SHA, file diffs, pending secret/sync blockers.
Chỉ dùng thao tác read-only đến khi lập được plan phục hồi,
sau đó đề xuất duy nhất bước retry/rollback an toàn.
Không báo DONE nếu chưa đọc lại trạng thái và sync đôi.
```
