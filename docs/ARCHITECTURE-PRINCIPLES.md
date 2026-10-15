# Nguyên tắc kiến trúc sơ bộ

**Không phải thiết kế đã duyệt.** Các nguyên tắc dưới đây chỉ định hướng nghiên cứu cho dự án mới.

1. **Rust-first core:** đặt nghiệp vụ và hợp đồng dữ liệu của phần lõi trong Rust; không trộn logic lõi với widget hay API hệ điều hành.
2. **Tách giao diện nền tảng:** thiết kế adapter cho Linux và Windows sau khi khảo sát trường hợp sử dụng; không mặc định framework GUI hoặc IPC.
3. **Trạng thái rõ ràng:** khai báo vòng đời, lỗi và khôi phục bằng các trạng thái kiểm thử được; không dùng hành vi ngầm.
4. **An toàn theo mặc định:** không ghi khóa API vào code, log, Git/Obsidian; không tự nâng quyền, cài service, thay config hay restart khi chưa có phê duyệt.
5. **Khả năng kiểm thử:** mock/fake adapters, case thành công và lỗi; thí nghiệm trên tunnel và cấu hình tách biệt với hệ đang chạy.
6. **Khả năng phân phối:** research packaging trước khi chọn, không tự mặc định AppImage/MSI/EXE hoặc cơ chế self-update.
7. **Thực chứng:** mỗi kiến trúc/claim cần requirement/ADR/test link; `proposed`, `accepted`, `implemented`, `verified` là các trạng thái khác nhau.

**OPEN:** core crates, runtime, async strategy, privileges, IPC, transport, process model, GUI, state storage, updater, audit log, compatible SDK.
