# ChatGPT Tunnel MCP Manager

**Trạng thái:** Khởi tạo hồ sơ / Discovery — chưa triển khai phần mềm.

Dự án mới hoàn toàn. Không kế thừa kiến trúc, mã, yêu cầu hoặc quyết định của dự án MCP Panel Manager cũ. Mọi đề xuất cần được khảo sát và phê duyệt riêng.

## Mục tiêu đã xác nhận
- Xây **lõi phần mềm bằng Rust**, ưu tiên thiết kế phần lõi độc lập nền tảng.
- Sau khi đặc tả và kiểm chứng lõi, phát triển ứng dụng cho **Linux** và **Windows**.
- Nghiên cứu và xác định các tính năng quản lý kết nối ChatGPT tunnel / MCP trong ChatGPT Project; **chưa mặc định chốt kiến trúc, giao diện, tích hợp, quy trình cập nhật, API hay IPC**.
- Giữ nguyên tunnel, MCP Proxy và các dịch vụ đang hoạt động; không áp dụng bất cứ thay đổi vận hành nào trong giai đoạn khởi tạo.

## Nguồn hồ sơ
- Obsidian vault `main`: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/`
- GitHub: https://github.com/congnvp/Chatgpt-tunnel-mcp-manager
- Nguồn code local trên Ubuntu (dự kiến khi bắt đầu development): `/home/congnvp-su/Projects/Chatgpt-tunnel-mcp-manager/` — **chưa tạo hoặc clone**.
- Giao thức phát triển: USES trong Obsidian `20_Projects/Active/Universal-Software-Engineering-Skills/`.

## Tài liệu khởi động
- [Project charter](docs/PROJECT-CHARTER.md)
- [Nguyên tắc kiến trúc](docs/ARCHITECTURE-PRINCIPLES.md)
- [Quy tắc Git ↔ Obsidian và logbook](docs/WORKING-AGREEMENT.md)
- [Câu hỏi còn mở](docs/OPEN-QUESTIONS.md)
- [Project Instructions](docs/CHATGPT-PROJECT-INSTRUCTIONS.md)
- [Logbook tại Git](LOGBOOK.md)

**Quy định:** Mọi thay đổi tài liệu, mã nguồn hoặc trạng thái dự án phải được ghi sự kiện cùng ID ở **hai logbook** (Git và Obsidian), xem `docs/WORKING-AGREEMENT.md`. Repo là public: tuyệt đối không commit token, secret hoặc cấu hình nhạy cảm.
