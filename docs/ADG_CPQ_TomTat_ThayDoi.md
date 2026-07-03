# ADG CPQ — Tóm tắt thay đổi tài liệu (v1 → v2)

## 1. Năm thay đổi lớn nhất (đọc kỹ nếu chỉ đọc một mục)

### ① Một báo giá chứa NHIỀU sản phẩm — thay đổi cấu trúc lớn nhất [D1]
- **v1:** mỗi báo giá chỉ có đúng 1 sản phẩm, số lượng luôn là 1. Khách mua 3 cửa = nhận 3 file báo giá riêng.
- **v2:** một báo giá = một khách hàng + nhiều **hạng mục** (mỗi hạng mục là một sản phẩm với kích thước, linh kiện, phụ kiện, ảnh hiện trường, số lượng và giá riêng). Khách nhận **một PDF duy nhất** liệt kê mọi hạng mục kèm tổng tiền.
- **Lý do:** khách của Austdoor (xây nhà, công trình) hầu như luôn đặt nhiều cửa một lần.
- **Kéo theo:** quy trình 8 phase tuyến tính của v1 được tổ chức lại thành **4 bước**: Khách hàng → Hạng mục (lặp) → Giá → Hoàn thiện.

### ② Hệ thống LỌC linh kiện tương thích ngay từ đầu [D2]
- **v1:** đại lý thấy toàn bộ danh sách SKU linh kiện và tự chọn, "hệ thống không gợi ý" — tính năng kiểm tra tương thích bị đẩy sang sau MVP.
- **v2:** admin cấu hình điều kiện tương thích (VD: bộ tời X chỉ dùng cho diện tích ≤ 25m²); hệ thống **chỉ hiển thị các SKU hợp lệ** theo thông số đã nhập, đại lý chọn trong số đó.
- **Lý do:** chọn sai linh kiện → BOM sai gửi thẳng về nhà máy. Rủi ro này không thể để sau MVP.

### ③ Toàn bộ logic tính BOM nằm TRONG hệ thống [D10]
- Câu hỏi lớn: logic tính vật tư đang nằm trong file Excel phức tạp của công ty — đẩy vào hệ thống hay để hệ thống gọi Excel?
- **Chốt:** đẩy toàn bộ vào hệ thống. Khi công ty chốt bảng tính chính thức, logic sẽ được **import (có AI hỗ trợ) và nghiệm thu bằng "golden tests"** — chạy song song Excel với hệ thống trên hàng loạt bộ thông số thật, khớp 100% mới đưa vào sử dụng. Sau đó **hệ thống là nguồn chân lý duy nhất**, sửa công thức = sửa trên hệ thống (có ghi vết).
- Ngôn ngữ công thức được đặc tả mới, hỗ trợ **rẽ nhánh điều kiện** (VD: "cao trên 3m thì dùng định mức khác") — bản v1 hoàn toàn không đặc tả điều này.

### ④ Giá cả có kỷ luật vòng đời rõ ràng [D3][D4][D5]
- Khoảng giá bán cho đại lý chỉ còn **một cơ chế min–max** do công ty quy định (bỏ cơ chế ±% chồng chéo của v1).
- **Nháp** không cam kết gì — khi bấm phát hành, hệ thống **tính lại theo cấu hình mới nhất** và bắt xác nhận nếu giá đổi. **Đã gửi khách = mọi con số đóng băng vĩnh viễn.** (v1 có lỗ hổng: nháp để lâu có thể phát hành với giá đã lỗi thời.)
- Việc đại lý chọn giá trong khoảng min–max chính là công cụ điều chỉnh giá/chiết khấu ở giai đoạn đầu; chính sách khuyến mãi từ công ty làm sau, thiết kế đã chừa chỗ.

### ⑤ Ảnh mock-up AI: bắt buộc nhưng không bao giờ gây tắc [D6]
- **v1 tự mâu thuẫn:** chỗ nói ảnh bắt buộc để phát hành, chỗ nói "theo chính sách"; và nếu cả 3 lần tạo ảnh đều xấu, đại lý bị ép chọn một ảnh xấu — bế tắc.
- **v2 — quy tắc 3 tầng:** (1) về nguyên tắc bắt buộc; (2) chặn từ gốc bằng hướng dẫn chụp + kiểm tra kỹ thuật ảnh đầu vào; (3) lối thoát: thay ảnh hiện trường khác (được tạo lại từ đầu) hoặc phát hành không kèm ảnh **kèm lý do bắt buộc, có ghi vết** để công ty giám sát.

## 2. Các sửa lỗi kỹ thuật đáng chú ý (nhóm F)

| Lỗi ở v1 | Đã sửa ở v2 |
|---|---|
| Bắt buộc có ảnh ngay khi tạo báo giá — nhưng quy trình lại upload ảnh sau → tạo báo giá là lỗi ngay | Ảnh cho phép trống khi nháp, bắt buộc trước khi phát hành [F1] |
| Sửa hồ sơ khách làm thay đổi cả báo giá đã gửi (không có bản chụp thông tin khách) | Snapshot tên/SĐT/địa chỉ khách vào báo giá lúc phát hành [F2] |
| Đổi tên sản phẩm làm sai lệch báo giá cũ | Snapshot thông tin sản phẩm vào từng hạng mục [F3] |
| Một sản phẩm có thể có nhiều BOM cùng hoạt động — không biết dùng cái nào | Đúng một BOM hoạt động cho mỗi sản phẩm [F4] |
| Không có ghi vết ai đổi giá/công thức lúc nào | Thêm nhật ký thay đổi cấu hình (`config_audit_log`) — trả lời được khiếu nại giá |

Ngoài ra v2 bổ sung: quy tắc làm tròn tiền VND, hạn hiệu lực báo giá in trên PDF (cơ chế sẵn, số ngày chờ chốt), yêu cầu ẩn dữ liệu mật (giá vốn, gross) phải thực thi ở tầng API chứ không chỉ giấu trên giao diện, và 7 use case phụ trợ được viết đầy đủ (quản trị người dùng, hồ sơ đại lý, gửi lại khi lỗi, tìm kiếm & sao chép báo giá...) — bản v1 dồn tất cả vào một câu.