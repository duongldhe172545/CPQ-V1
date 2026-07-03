# Addendum — PRD ADG CPQ

Nội dung có giá trị cho các chặng sau (kiến trúc, UX, epics) nhưng không thuộc thân PRD.

## Cho chặng Kiến trúc

- **Quyết định nền tảng [D11] — nguyên văn chủ dự án (03/07):** *"ZMA chỉ là cổng thôi, cái CPQ này như 1 web đính vào, nên cứ WEB trước đã."* Hàm ý kiến trúc: frontend đại lý là web app (SPA/PWA) nhúng được vào webview của Zalo Mini App ở Phase 2; tránh phụ thuộc API riêng của ZMP ở tầng lõi; auth phải mở đường cho đăng nhập qua Zalo ID sau này mà không đổi mô hình phiên.
- **Engine BOM — lịch sử quyết định [D14 đảo ngược D10, 03/07]:** phương án ban đầu (import toàn bộ logic Excel vào hệ thống, golden tests, hệ thống là nguồn chân lý) bị công ty đảo ngược: **chỉ quan tâm INPUT/OUTPUT của file Excel** — logic tính nằm nguyên trong file, hệ thống là người bơm input và đọc output. Đánh đổi đã ghi nhận từ phân tích 02/07 và được chấp nhận: file là hộp đen trong đường găng nghiệp vụ → kiểm soát bằng bộ ba *phiên bản hoá + checksum + nghiệm thu bộ ca mẫu* (FR-090/091); macro/VBA không hỗ trợ.
- **Engine tính file Excel server-side (AD-17 — cần spike):** ứng viên: LibreOffice headless (tính toán đầy đủ, nặng process), HyperFormula (JS, nhanh, cần kiểm phủ hàm), SheetJS + thư viện formula, hoặc sidecar Python (`formulas`). Tiêu chí chọn: phủ đủ hàm trong file thật của công ty, chạy được trong worker, thời gian tính ≤ NFR-06. Chốt bằng story spike chạy trên chính file thật.
- **Ngôn ngữ công thức (chỉ còn auto-fill + điều kiện tương thích):** đặc tả trong `docs/ADG_CPQ_ERD.dbml` Project Note. Engine đánh giá biểu thức cần: sandbox (không eval tuỳ tiện), giới hạn hàm cho phép, thông báo lỗi trỏ đúng biến/công thức, kiểm tra biến tồn tại lúc lưu cấu hình (fail-fast ở admin, không đợi đại lý gặp).
- **Ẩn dữ liệu mật tầng API:** gợi ý mô hình serializer/DTO theo vai — vai dealer không bao giờ nhận field nhạy cảm trong payload, kể cả null. Test bảo mật bắt buộc có ca này (NFR-01).
- **Mock-up AI:** chưa chọn provider. Yêu cầu năng lực: nhận 2 ảnh (hiện trường + sản phẩm) → ảnh ghép; thời gian ≤ 60s; chi phí/lần cần ước tính khi chọn (ảnh hưởng trực tiếp cấu hình số lượt gen). Cân nhắc: API thương mại (nhanh, phí/lần) vs self-host (đầu tư trước). Thiết kế hàng đợi + webhook/polling vì thời gian sinh không tức thời.
- **ERD:** `docs/ADG_CPQ_ERD.dbml` là thiết kế dữ liệu mức logical đã review kỹ — dùng làm điểm xuất phát, kiến trúc sư được quyền điều chỉnh physical (index, partition) nhưng thay đổi logical phải đối chiếu decision log.

## Cho chặng UX

- Pipeline đại lý: tối ưu thao tác một tay trên điện thoại ngoài công trình (nắng, vội). Bước chụp ảnh có overlay hướng dẫn khung hình.
- Sao chép hạng mục (FR-041) là thao tác tần suất cao — đặt nổi.
- Màn hình chọn giá bán: hiển thị giá nhập + range dạng slider/ô nhập có chặn; **tuyệt đối không render field mật rồi giấu bằng CSS**.
- Admin: màn hình công thức cần bộ kiểm thử nhúng (FR-013, UC-A6 bước 4) — admin không phải dev, feedback lỗi phải bằng ngôn ngữ nghiệp vụ.

## Cho epics/stories

- Thứ tự build gợi ý theo phụ thuộc: nền quản trị (F8) → catalog (F1–F3) → giá (F4) → pipeline (F5) → phát hành (F7) → mock-up AI (F6) → nghiệm thu workbook & seed (F9). Mock-up để sau phát hành vì có đường phát hành-không-ảnh [D6].
- Khung nghiệm thu workbook (FR-090/091) nên build sớm cùng story spike chọn engine — cả hai chạy được với file Excel giả lập, không đợi file thật của công ty.

## Ghi chú quy mô (nguồn: phỏng vấn 03/07)

- 4 mục tiêu kinh doanh đều được chọn; thứ tự ưu tiên: chốt đơn > BOM đúng > tốc độ > kiểm soát giá.
- Quy mô năm đầu: toàn hệ thống >100 đại lý. Con số NFR-05 (300 tài khoản, 50 đồng thời, 5.000 quote/tháng) là ước tính của người soạn từ quy mô đại lý — cần chủ dự án xác nhận lại khi có số thật.
