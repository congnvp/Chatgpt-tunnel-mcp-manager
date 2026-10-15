# Project Charter — ChatGPT Tunnel MCP Manager

**Project ID:** CTMM | **Date:** 2026-10-16 | **Phase:** Intake / Discovery | **Status:** Draft, pending discovery.

## Bài toán
Tạo ứng dụng quản lý tunnel và MCP dùng với ChatGPT. Sản phẩm phải được thiết kế lại từ đầu, KHÔNG dựa vào thiết kế/mã nguồn/ADR/quyết định của MCP Panel Manager trước đây.

## Các yêu cầu đã chốt
- Toàn bộ phần lõi viết bằng **Rust**; các thành phần giao diện/tích hợp OS triển khai **sau** khi phần lõi được xác định rõ và kiểm chứng.
- Hai nền tảng đích: **Linux** và **Windows**.
- Vận hành hồ sơ qua **ChatGPT Project + Obsidian Ubuntu + GitHub**.
- Mọi thay đổi và ghi chép phải có logbook đối ứng ở **cả Obsidian và Git**.
- Trong giai đoạn khởi tạo **không sửa/restart/uninstall/replace** tunnel, MCP Proxy hoặc phần mềm đang hoạt động.

## Chưa chốt
- Danh sách tính năng, mô hình tunnel/proxy, giao thức kết nối, quản lý vòng đời process, mode GUI/CLI/daemon/headless.
- GUI toolkit, packaging, update, quyền hệ điều hành, cấu trúc bảo mật và cơ chế lưu cấu hình.
- Kiến trúc module/crate, giao diện IPC/API, mức tương thích, acceptance criteria, CI/CD, định nghĩa MVP.

## Hướng đi theo phase (không phải cam kết tiến độ)
1. Khảo sát và ghi yêu cầu mới, nghiên cứu phương án; review từng quyết định.
2. Chốt ADR, security boundaries, architecture, data contracts và tiêu chí kiểm thử.
3. Viết lõi Rust, kiểm thử tự động, kiểm chứng độc lập với production.
4. Xây bản Linux; test giao diện và tích hợp thực tế trên tunnel **riêng**.
5. Xây bản Windows, tái sử dụng lõi, test; release/rollback.

**Gate:** Không bắt đầu code sản phẩm cho tới khi yêu cầu blocking và các quyết định kiến trúc liên quan được chốt.

## Kho lưu
Obsidian: `main/20_Projects/Active/Chatgpt-tunnel-mcp-manager/`.
GitHub: https://github.com/congnvp/Chatgpt-tunnel-mcp-manager (public).
Local source on Ubun **proposed, not created**: `/home/congnvp-su/Projects/Chatgpt-tunnel-mcp-manager/`.
