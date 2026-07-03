# ADG CPQ — Bản ghi Quyết định & Hạng mục Sửa đổi

> **Nguồn:** Review adversarial 4 tài liệu BA (23 findings) + phỏng vấn nghiệp vụ anh DUong, 02/07/2026.
> **Trạng thái:** Sống — cập nhật khi các Open Decision được chốt.
> **Cách dùng:** Mọi thay đổi lên 4 tài liệu gốc (`ADG_CPQ_ERD.dbml`, `ADG_CPQ_NghiepVu_UseCase.md`, `ADG_CPQ_UseCase_ChiTiet.md`, swimlane) phải truy vết về một mã D-x / F-x / O-x trong file này.

---

## A. Quyết định nghiệp vụ đã chốt (D)

| Mã | Chủ đề | Quyết định | Tác động tài liệu |
|----|--------|-----------|-------------------|
| D1 | Cấu trúc báo giá | Một báo giá chứa **nhiều hạng mục sản phẩm** (khách đặt nhiều cửa trong một lần báo giá). Mỗi hạng mục có bộ thông số, linh kiện, phụ kiện, số lượng riêng. | ERD: thêm tầng `quote_item`; toàn bộ nhóm `quote_*` treo theo item thay vì quote. UC-03→UC-08 chạy lặp theo từng hạng mục. Swimlane thêm vòng lặp hạng mục. |
| D2 | Lọc SKU tương thích | Làm **ngay MVP**: admin cấu hình điều kiện tương thích (VD bộ tời X dùng cho diện tích ≤ 25m²); Phase 4 chỉ hiển thị SKU hợp lệ theo thông số dự án, đại lý chọn 1 trong số đó. **Đảo ngược** rule của BA ("hệ thống không gợi ý SKU"). | ERD: thêm bảng quy tắc tương thích. UC-A4, UC-05 viết lại. |
| D3 | Range giá bán | Chỉ dùng **một cơ chế min–max** do công ty quy định. **Bỏ `band_percent`** (±%). Giá trị cụ thể: xem O1. | ERD: xoá cột `band_percent`. UC-A7, UC-08 cập nhật. |
| D4 | Vòng đời giá | Nháp = không cam kết, hệ thống **tính lại theo cấu hình mới nhất tại thời điểm phát hành** (báo đại lý xác nhận nếu giá đổi so với nháp). **Đã gửi khách = giá đóng băng vĩnh viễn** (snapshot). Điều chỉnh giá trong thi công nằm ngoài hệ thống báo giá. | UC-07, UC-10, UC-11 bổ sung quy tắc. |
| D5 | Chiết khấu MVP | Cơ chế "đại lý chọn giá trong min–max" **chính là** công cụ điều chỉnh giá/chiết khấu ở MVP. Các chính sách khuyến mãi từ công ty: sau MVP, thiết kế phải **chừa hook** (không hard-code giá). | Ghi rõ scope trong tài liệu nghiệp vụ. |
| D6 | Ảnh mock-up AI | Quy tắc 3 tầng, không deadlock: (1) **bắt buộc theo nguyên tắc** (cờ `mockup_required`, mặc định bật); (2) chặn từ gốc: hướng dẫn chụp + kiểm tra tối thiểu (định dạng, dung lượng, độ phân giải) khi upload; (3) lối thoát: hết 3 lượt gen không đạt → cho **thay ảnh hiện trường & reset lượt gen**, hoặc "phát hành không kèm ảnh" **kèm lý do bắt buộc, có log** để công ty giám sát. | UC-02, UC-09, UC-10 viết lại; sửa mâu thuẫn tiền điều kiện UC-10 và deadlock UC-09 E-1. |
| D7 | VAT | Giữ **một mức chung** toàn hệ thống. Không ưu tiên phân hoá thuế suất lúc này. | Giữ nguyên thiết kế; ghi chú xem lại sau MVP. |
| D8 | Thiết bị | Đại lý thao tác **chủ yếu trên điện thoại** — yêu cầu mobile-first cho phần đại lý (độc lập với quyết định nền tảng O5). | Thêm mục yêu cầu phi chức năng. |
| D9 | Vai trò hệ thống | **HUỶ ý tưởng vai thứ 3 (02/07, anh DUong đính chính):** khách hàng cuối **không truy cập hệ thống**. Chỉ có đúng **2 vai: admin + đại lý** như thiết kế gốc của BA; khách là người nhận báo giá thụ động. | Giữ nguyên enum `user_role`; không thêm nhóm UC khách. |
| D11 | Nền tảng: web-first | MVP = **web app mobile-first** cho đại lý + **web admin**; Zalo Mini App chỉ là cổng/vỏ bọc ở Phase 2, dùng chung API/UI web. Kênh gửi khách MVP: tải PDF thủ công (bắt buộc) + Zalo OA tự động (should-have). Đóng O5. | PRD NFR-04; kiến trúc frontend phải nhúng được vào webview ZMA. |
| D12 | Ảnh mock-up AI trong MVP | Chốt 03/07: mock-up AI **nằm trong MVP** (gắn với mục tiêu số 1 — tăng tỷ lệ chốt đơn). | PRD nhóm F6. |
| D13 | Quy mô & mục tiêu | Năm đầu: **toàn hệ thống >100 đại lý**. Ưu tiên mục tiêu: tăng chốt đơn > giảm sai BOM > rút ngắn thời gian báo giá > chuẩn hoá giá. | PRD mục 2, 9; NFR quy mô. |
| D10 | Engine BOM: import toàn bộ vào hệ thống | Chốt phương án **(B) — toàn bộ logic BOM nằm trong hệ thống**, không dùng Excel làm engine. Lý do: phiếu BOM sản xuất gửi nhà máy và nhiều logic phức tạp vượt khả năng Excel. File Excel hiện hành (khi công thức được chốt) là **nguồn để AI-assisted import một lần**, nghiệm thu bằng golden tests (chạy song song Excel vs hệ thống trên N bộ thông số thật, khớp 100% mới go-live); sau import, hệ thống là nguồn chân lý duy nhất — khoá sửa Excel. | UC-A6, UC-07 giữ mô hình công thức trong hệ thống như BA đề xuất, bổ sung yêu cầu ngôn ngữ công thức có rẽ nhánh (if/else, bảng tra theo lựa chọn) + UC công cụ import & golden tests. F4 giữ hiệu lực. |

## B. Lỗi tài liệu phải sửa (F) — mâu thuẫn nội bộ, không cần chốt nghiệp vụ

| Mã | Lỗi | Hướng sửa |
|----|-----|-----------|
| F1 | `quote.field_photo_url` khai `not null` nhưng quote được tạo ở Phase 1 trước khi upload ảnh (UC-02) → insert fail ngay bước đầu. | Cho phép NULL khi `status = draft`; validate bắt buộc khi chuyển phase / phát hành. |
| F2 | UC-01 tuyên bố "sửa khách cũ không hồi tố báo giá đã gửi" nhưng `quote` chỉ giữ `customer_id`, không snapshot khách → sửa khách là báo giá cũ đổi theo. | Snapshot tên/SĐT/địa chỉ khách vào quote tại thời điểm phát hành. |
| F3 | Sản phẩm không được snapshot trong quote (chỉ `product_id`) → admin đổi tên/thumbnail là PDF cũ sai lệch. Trái nguyên tắc snapshot A0. | Snapshot tên/SKU/thumbnail sản phẩm vào hạng mục báo giá. |
| F4 | `bom_template` cho phép nhiều template active cùng lúc cho một sản phẩm; UC-07 mặc nhiên số ít. | Ràng buộc: đúng **một** BOM active mỗi sản phẩm (unique partial index hoặc validate tầng app). *(Có thể vô hiệu nếu O6 chọn phương án Excel-engine.)* |
| F5 | Tiền điều kiện UC-10 ("phải có ảnh mock-up") mâu thuẫn với ngoại lệ E-3 ("theo chính sách"). | Thay bằng quy tắc D6. |

## C. Quyết định còn treo (O) — thiết kế chừa sẵn chỗ, chờ chốt

| Mã | Vấn đề | Cách xử lý tạm trong tài liệu | Ai chốt |
|----|--------|------------------------------|---------|
| O1 | Giá trị min–max cụ thể cho đại lý | Cơ chế làm trước; con số nhập qua màn hình admin khi công ty ban hành. | Công ty / sếp |
| O2 | Đại lý có được bán dưới giá nhập không | Mặc định **chặn** (giá bán ≥ giá nhập), kèm cờ cấu hình để nới. | Công ty / sếp |
| O3 | Thời hạn hiệu lực báo giá | Trường `validity_days` cấu hình được, nullable; để trống = không in hạn lên PDF. | Công ty / sếp |
| O4 | Quy trình đặt hàng nhà máy (kênh, format, định tuyến nhà máy nào) | MVP dừng ở "xuất phiếu BOM chuẩn"; kênh gửi (API/email) chốt sau. | Anh DUong hỏi bộ phận sản xuất |
| ~~O5~~ | **ĐÃ ĐÓNG (03/07) → D11:** web-first. Lời chủ dự án: "ZMA chỉ là cổng, CPQ như một web đính vào — cứ WEB trước." MVP = web app mobile-first (đại lý) + web admin; Phase 2 bọc vào Zalo Mini App dùng chung API/UI. Kênh gửi khách MVP: tải PDF thủ công (bắt buộc) + Zalo OA tự động (should-have). | — | — |
| ~~O6~~ | **ĐÃ ĐÓNG (02/07) → D10:** chốt import toàn bộ logic BOM vào hệ thống. Còn phụ thuộc: công thức cụ thể chưa chốt — chờ bảng tính chính thức từ công ty để thực hiện import + golden tests. | — | — |
| ~~O7~~ | **ĐÃ ĐÓNG (02/07):** giữ `product` tối giản theo triết lý config-driven; màu sắc & tương tự là thông số dự án (UC-A2). Rủi ro ghi nhận để xử lý khi có catalog thật: nếu màu chọn lúc báo giá thì ảnh dùng cho mock-up AI phải theo màu đã chọn — một `thumbnail_url` không đủ. | — | — |
| ~~O8~~ | **ĐÃ ĐÓNG (02/07) → D9:** khách hàng không truy cập hệ thống; câu hỏi về luồng khách tự cấu hình không còn hiệu lực. | — | — |

## D. Ghi chú phạm vi đã xác nhận

- Sau phát hành **không theo dõi phản hồi khách** (chấp nhận cho MVP).
- Không lưu **lịch sử giá vật tư** ở MVP *(sẽ vô hiệu nếu O6 chọn Excel-engine — giá nằm trong hệ thống hay Excel sẽ quyết định lại)*.
- Các UC phụ trợ (quản trị người dùng, hồ sơ đại lý, đơn vị tính, retry gửi, tìm kiếm/sao chép báo giá) phải được viết đầy đủ trong bản cập nhật — không để dồn một câu phụ lục.
- Cần bổ sung: quy tắc làm tròn tiền VND, audit trail cho dữ liệu giá (ai đổi gì lúc nào), cơ chế thực thi "ẩn dữ liệu mật với đại lý" ở tầng API.
