# Quy ước làm việc và logbook hai chiều — BẮT BUỘC

## Không gian chính thức
- Repository: `congnvp/Chatgpt-tunnel-mcp-manager`, nhánh mặc định `main`; tạo feature branches theo quy trình sau khi chốt.
- Obsidian trên Ubun: vault `main`, `20_Projects/Active/Chatgpt-tunnel-mcp-manager/`.
- Git logbook: `LOGBOOK.md`.
- Obsidian logbook: `20_Projects/Active/Chatgpt-tunnel-mcp-manager/Logbook.md`.

## Quy tắc bất biến
**MỖI lần thêm/sửa/xóa bất kỳ mã nguồn, tài liệu, ghi chú, trạng thái hoặc ADR thuộc dự án, phải tạo sự kiện tương ứng tại CẢ HAI logbook**, dù thay đổi bắt đầu từ Git hay Obsidian. Không được chỉ cập nhật một phía rồi tuyên bố hoàn tất. Mỗi phiên có thể gom những thay đổi liên quan thành một change-set có cùng event ID; kê rõ TỪNG đường dẫn tác động.

Schema bắt buộc mỗi sự kiện:
`Event-ID | ngày/giờ Asia/Bangkok | actor | action | files_Git | notes_Obsidian | requirement/ADR/task | status | evidence/tests | git_commit | pending/next`.
- ID duy nhất, ổn định: `CTMM-YYYYMMDD-NNN`. Kết quả `untested` phải ghi rõ; không gắn PASS khi chưa chạy test.
- Phần chứng cứ: sha/PR/issue, kết quả lệnh test, đường dẫn log đã lọc secret; nếu là docs-only, ghi `documentation review; no runtime tests`.
- Trong Git logbook khi event thuộc ngay commit hiện tại, ghi `git_commit: this commit`; logbook Obsidian bổ sung SHA thực tế sau commit.
- Logbook theo kiểu **append-only**: sửa sai thêm sự kiện `correction` dẫn lại event cũ, không xóa dấu vết.
- Các thay đổi đối với chính hai logbook như một phần của change-set **không đệ quy sinh vô hạn sự kiện mới**.
- Nội dung nguồn gốc không được sửa đồng thời độc lập ở hai nơi: Obsidian là nguồn gốc quyết định/trạng thái, Git là nguồn mã/tests và bản snapshot tài liệu phát hành; đồng bộ bằng ID và provenance.

## Trình tự một thay đổi
1. Đọc BOOT, ROUTER, entrypoint dự án, Project-State và logbook gần nhất; xác minh trạng thái repo, nhánh và môi trường.
2. Gán Event-ID và phạm vi; ghi thay đổi theo scope, không ghi đè người khác.
3. Chạy kiểm tra tương xứng (nếu chưa chạy ghi `NOT RUN`); tuyệt đối không khẳng định đã kiểm thử.
4. Commit code/docs + mục Git `LOGBOOK.md` trong cùng change-set/commit khi có thể.
5. Append sự kiện **cùng Event-ID** và kết quả/commit SHA thực tế vào Obsidian `Logbook.md`, cập nhật Project-State; đọc lại kiểm chứng cả hai.
6. Nếu một phía thất bại: đánh dấu **SYNC_PENDING**, ghi nguồn sự thật nào đã lưu, retry có kiểm soát trước khi mở task khác. Không báo DONE nếu logbook đôi chưa đồng bộ.
7. Kết phiên: thay đổi, decisions, evidence, blockers và next task; không suy từ lịch sử chat.

## Bảo vệ vận hành và bí mật
Không lưu API key/token/password/cookie/SSH private key/`.env` vào repo hoặc vault. GitHub hiện là **PUBLIC**, kiểm tra nội dung trước mọi push. Mọi thao tác service/tunnel/proxy đang chạy là ngoài scope nếu chưa được cho phép riêng. Không import lại tài liệu/mã/ADR từ dự án cũ.
