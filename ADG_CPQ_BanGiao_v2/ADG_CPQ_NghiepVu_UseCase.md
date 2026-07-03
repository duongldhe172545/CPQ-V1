# ADG CPQ — Diễn giải nghiệp vụ & triết lý thiết kế (v3)

> **PHIÊN BẢN 3 (03/07/2026)** — giống hệt v2, **chỉ đổi cách tính BOM [D14]**: không import logic Excel vào hệ thống, BOM tính bằng chính file Excel của công ty qua hợp đồng INPUT/OUTPUT (xem A0 và A5).
> **PHIÊN BẢN 2 (02/07/2026)** — cập nhật sau review adversarial (23 findings) + phỏng vấn nghiệp vụ anh DUong.
> Bản gốc của BA: `docs/_ban-goc-BA/`. Truy vết quyết định: `_bmad-output/planning-artifacts/ADG-CPQ-decision-log.md` (mã D-x / F-x / O-x).

Tài liệu này đi kèm ba file thiết kế:

- `ADG_CPQ_ERD.dbml` (v2) — sơ đồ dữ liệu (import vào dbdiagram.io).
- `ADG_CPQ_Swimlane_v2.drawio` — luồng nghiệp vụ 4 bước (mở bằng draw.io / diagrams.net; bản PNG cũ lưu ở `docs/_ban-goc-BA/`).
- `ADG_CPQ_UseCase_ChiTiet.md` (v2) — đặc tả use case chi tiết (8 UC cấu hình + 12 UC pipeline + 7 UC phụ trợ).

---

## Phần A — Triết lý thiết kế & diễn giải nghiệp vụ

### A0. Nguyên tắc xuyên suốt: cấu hình bằng dữ liệu, không đụng schema

Yêu cầu cốt lõi: một đại lý bất kỳ có thể được cấp thêm sản phẩm mới, với bộ thông số và công thức hoàn toàn khác, mà đội phát triển không phải sửa cấu trúc cơ sở dữ liệu hay viết thêm code. ERD tổ chức quanh nguyên tắc này qua ba cơ chế:

**Tách định nghĩa khỏi giá trị.** Sản phẩm không có cột "chiều dài", "chiều cao" cứng; nó khai báo động bộ thông số trong `project_param_def`, giá trị đại lý nhập nằm ở `quote_item_param`. Thêm thông số mới = thêm một dòng cấu hình. Cùng mô hình cho thông số linh kiện.

**Công thức nhỏ là dữ liệu; logic BOM là file Excel [D14].** Hai tầng logic được tách bạch. (1) **Auto-fill thông số linh kiện + điều kiện tương thích SKU**: lưu dạng biểu thức chuỗi do admin quản lý trong hệ thống, tham chiếu biến qua `code` ổn định (`proj.<code>`, `comp.<code>`), hỗ trợ rẽ nhánh `if(...)` — đặc tả trong Project Note của file DBML. (2) **Toàn bộ logic tính vật tư (BOM)**: nằm **nguyên trong file Excel của công ty** — hệ thống không import, không tái hiện, chỉ làm việc qua **hợp đồng INPUT/OUTPUT**: bơm giá trị thông số vào sheet INPUT, đọc kết quả (mã vật tư + số lượng) từ sheet OUTPUT. Đội làm giá tiếp tục làm việc trên file quen thuộc; đổi lại hệ thống kiểm soát file bằng bộ ba *phiên bản hoá + checksum + nghiệm thu bộ ca mẫu* (UC-A6/A8).

**Snapshot khi chốt [D4].** Mọi giá trị của một báo giá đã phát hành (thông số, linh kiện, phụ kiện, dòng BOM, con số giá, **thông tin khách hàng [F2] và sản phẩm [F3]**) được sao lưu trong nhóm bảng `quote_*`. Admin thoải mái đổi cấu hình/công thức/đơn giá về sau — báo giá đã phát hành giữ nguyên số liệu vĩnh viễn. Ngược lại, báo giá **nháp không cam kết gì**: khi phát hành, hệ thống luôn tính lại theo cấu hình mới nhất và yêu cầu đại lý xác nhận nếu giá đổi — không bao giờ phát hành giá lỗi thời.

### A1. Cấu trúc báo giá: nhiều hạng mục [D1]

Khách của Austdoor (xây nhà, công trình) thường đặt **nhiều cửa trong một lần báo giá**. Vì vậy một `quote` = một khách hàng + **nhiều `quote_item`** (hạng mục); mỗi hạng mục là một sản phẩm với bộ thông số, linh kiện, phụ kiện, ảnh hiện trường, số lượng và giá riêng. Hai cửa giống hệt nhau = một hạng mục với số lượng 2. Giá tính theo từng hạng mục; tổng tiền và VAT tính ở cấp báo giá; khách nhận **một PDF** liệt kê mọi hạng mục.

Pipeline vì thế gồm 4 bước lớn thay cho 8 phase tuyến tính cũ: **(1) Khách hàng → (2) Hạng mục** (vòng lặp cấu hình từng hạng mục: sản phẩm → ảnh hiện trường → thông số → linh kiện → phụ kiện) **→ (3) Giá → (4) Hoàn thiện** (mock-up, PDF, phát hành).

### A2. Catalog & thông số dự án

`category` là cây danh mục nhiều cấp. `product` giữ **tối giản** (tên, SKU, thumbnail, mô tả): màu sắc, kích thước... là thông số dự án khai ở `project_param_def` — đây chính là "thiết kế phiếu nhập liệu" mà đại lý điền cho từng hạng mục. Mỗi thông số khai kiểu dữ liệu, cách nhập (freeform/dropdown), đơn vị, bắt buộc; option của dropdown nằm ở `project_param_option` với `value` dùng được trong công thức. *Lưu ý ghi nhận (O7 đã đóng):* nếu màu là thông số chọn lúc báo giá, ảnh dùng cho mock-up AI cần theo màu đã chọn — xử lý khi có catalog thật.

### A3. Linh kiện: auto-fill + lọc tương thích [D2]

Linh kiện mô hình hoá ba tầng dùng chung: `component_type` (loại), `component_param_def` (bộ thông số của loại), `component_sku` (mã cụ thể với đơn giá). Giá trị thông số mặc định theo SKU (`component_sku_param_value`) là nguồn biến `comp.*`.

Hai cơ chế đều đặt tại **bảng gán `product_component_type`** (vì loại linh kiện dùng chung nhưng logic phải tham chiếu thông số dự án vốn riêng theo sản phẩm):

1. **Công thức auto-fill** (`component_param_formula`): cùng một "bộ tời" gắn vào hai sản phẩm khác nhau có thể có công thức tính công suất khác nhau. Đại lý được chỉnh giá trị auto-fill (cờ `is_overridden`).
2. **Điều kiện tương thích** (`component_sku_applicability`) — *thay đổi lớn so với bản BA [D2]:* hệ thống **lọc** danh sách SKU theo thông số dự án đã nhập (VD kích thước lớn → chỉ còn 2 bộ tời hợp lệ), rồi **đại lý chọn** trong danh sách đã lọc. Nguyên tắc phân vai: *hệ thống lọc, đại lý chọn* — hệ thống không tự quyết. Bản BA ghi "không gợi ý SKU, để sau MVP" nhưng phỏng vấn nghiệp vụ xác nhận việc chọn sai SKU dẫn tới BOM sai gửi nhà máy — nên bộ lọc vào MVP.

### A4. Phụ kiện

`accessory` là danh mục dùng chung, gán vào sản phẩm qua `product_accessory`. Ở báo giá, phụ kiện chỉ tick chọn, **số lượng luôn = 1 mỗi hạng mục**, đơn giá cộng thẳng một lần vào giá vốn của hạng mục.

### A5. Vật tư & workbook BOM [D14]

`material` là danh mục vật tư do admin quản lý, một đơn giá hiện hành (không lịch sử — MVP); `code` vật tư phải khớp mã vật tư trong OUTPUT của workbook. `product_bom_workbook` gắn theo sản phẩm: admin upload file Excel chuẩn hoá (sheet INPUT ô đặt tên theo thông số, sheet OUTPUT bảng chuẩn [mã vật tư, số lượng]), hệ thống kiểm tra hợp đồng + lưu **phiên bản có checksum**; **đúng một version active mỗi sản phẩm [F4]** và chỉ version vượt **nghiệm thu bộ ca mẫu** (UC-A8) mới được active. Linh/phụ kiện không tác động BOM. **[OPEN]** OUTPUT khuyến nghị chỉ gồm mã + số lượng (đơn giá lấy từ hệ thống để giữ audit/bảo mật) — chốt khi xem file thật.

### A6. Định giá & che giấu dữ liệu nhạy cảm

Tính **theo từng hạng mục**: giá vốn (`cost_material`) = tiền vật tư BOM + tiền linh kiện + tiền phụ kiện. Giá nhập = `cost_material × (1 + gross%/100)`, trong đó `gross% = global_gross%` (theo sản phẩm) cộng gross đại lý — mức theo-sản-phẩm (`dealer_product_gross`) nếu có sẽ **thay thế** mức chung (`dealer.default_gross_percent`), không cộng dồn.

Từ giá nhập, đại lý chọn giá bán trong **range min–max theo sản phẩm [D3** — đã bỏ cơ chế ±% của bản BA; giá trị cụ thể **OPEN O1]**. Việc chọn giá trong range chính là công cụ điều chỉnh giá/chiết khấu của đại lý ở MVP [D5]; chính sách khuyến mãi từ công ty là hạng mục sau MVP, thiết kế không hard-code giá để chừa hook. Mặc định **không cho bán dưới giá nhập** (cờ `allow_below_import` để nới — **OPEN O2**). VAT một mức chung toàn hệ thống [D7], tính trên tổng giá bán; tổng gửi khách = subtotal + VAT. Tiền làm tròn về đồng tại từng dòng (đề xuất — kế toán xác nhận).

**Ranh giới hiển thị:** `cost_material`, đơn giá vật tư, dòng BOM và mọi % gross là thông tin mật, **ẩn hoàn toàn với đại lý**. Đại lý chỉ thấy giá nhập, range, giá bán mình chọn, VAT. Yêu cầu mới so với bản BA: việc ẩn phải **thực thi ở tầng API** (field-level filtering theo role), không chỉ ẩn trên giao diện. Kèm theo: mọi thay đổi dữ liệu giá/công thức ghi vào `config_audit_log` — trả lời được "ai đổi gì lúc nào" khi có khiếu nại.

### A7. Báo giá & vòng đời

Một `quote` gắn một khách hàng (chọn lại từ kho khách của đại lý theo SĐT — `unique(dealer_id, phone)`) và nhiều hạng mục. Ảnh hiện trường thuộc **từng hạng mục** (mỗi cửa một vị trí lắp), cho phép trống khi nháp và bắt buộc trước khi phát hành [F1]. Báo giá hỗ trợ lưu nháp và quay lại sửa (`current_phase` cấp quote + `step` từng hạng mục). Phát hành xong → `submitted`, giá đóng băng vĩnh viễn, quy trình kết thúc — không theo dõi phản hồi khách (phạm vi MVP đã xác nhận). Báo giá có thể có **hạn hiệu lực** in trên PDF (`valid_until` — cơ chế đã sẵn, số ngày **OPEN O3**).

### A8. Ảnh AI, gửi khách, gửi nhà máy

Ở bước hoàn thiện, dịch vụ AI ghép ảnh mock-up **cho từng hạng mục** từ ảnh hiện trường của hạng mục + thumbnail sản phẩm — minh hoạ "nhà khách sau khi lắp cửa". Quy tắc 3 tầng [D6]: về nguyên tắc **bắt buộc** (cờ `mockup_required`); chặn từ gốc bằng hướng dẫn chụp + kiểm tra kỹ thuật ảnh đầu vào; lối thoát khi hết `ai_max_regenerate` lượt mà không ảnh nào đạt — thay ảnh hiện trường (reset lượt) hoặc phát hành không kèm ảnh **với lý do bắt buộc, có log giám sát**. Lần gen lỗi kỹ thuật không tính lượt.

Sau phát hành: PDF gửi khách (`customer_delivery` — kênh cụ thể **OPEN O5**, chốt cùng quyết định nền tảng; đại lý luôn tải được PDF gửi thủ công) và phiếu BOM sản xuất gộp mọi hạng mục gửi về nhà máy (`factory_bom_dispatch` — quy trình/kênh/định tuyến **OPEN O4**). Cả hai kênh có retry (UC-P4).

### A9. Vai trò & thiết bị

Đúng **hai vai** đăng nhập: **admin** (tổng công ty — cấu hình catalog, công thức, giá) và **dealer** (đại lý — pipeline báo giá) [D9]. Khách cuối **không truy cập hệ thống** — chỉ nhận báo giá. Một đại lý có nhiều tài khoản, thấy chung dữ liệu của đại lý đó. Đại lý thao tác **chủ yếu trên điện thoại** (chụp ảnh tại công trình) → phần đại lý phải mobile-first [D8]; quyết định nền tảng cụ thể (Zalo Mini App / web) là **OPEN O5**.

---

## Phần B — Các quyết định còn treo (Open Decisions)

Chi tiết đầy đủ trong `ADG-CPQ-decision-log.md`; tóm tắt các điểm còn hiệu lực:

| Mã | Vấn đề | Trạng thái thiết kế | Chờ ai |
|---|---|---|---|
| O1 | Giá trị min–max cho đại lý | Cơ chế sẵn, nhập số qua admin | Công ty |
| O2 | Bán dưới giá nhập? | Mặc định chặn, có cờ nới | Công ty / sếp |
| O3 | Hạn hiệu lực báo giá | `validity_days` cấu hình, trống = không in | Công ty / sếp |
| O4 | Quy trình BOM nhà máy | MVP: xuất phiếu chuẩn; kênh chốt sau | Bộ phận sản xuất |
| O5 | Nền tảng & kênh gửi khách | Tài liệu viết trung lập nền tảng | Anh DUong / công ty |
| — | File Excel BOM thật, chuẩn hoá thành workbook INPUT/OUTPUT | Đầu vào cho UC-A6/A8 (upload + nghiệm thu) [D14] | Công ty |
| — | OUTPUT workbook trả gì: chỉ [mã + số lượng] (khuyến nghị) hay cả tiền | Tài liệu viết theo khuyến nghị | Xem file thật rồi chốt |
| — | Quy tắc làm tròn tiền | Đề xuất: về đồng từng dòng | Kế toán |

## Đặc tả use case

Bộ use case đầy đủ ở **`ADG_CPQ_UseCase_ChiTiet.md`** (v3): 8 UC cấu hình (UC-A1..A8, trong đó UC-A6/A8 theo mô hình workbook BOM [D14]), 12 UC pipeline theo mô hình nhiều hạng mục (UC-01..12), 7 UC phụ trợ (UC-P1..P7). Mỗi UC gồm bảng metadata, luồng chính đánh số, luồng phụ/ngoại lệ (a/b/c, E-) và business rules; các điểm chưa chốt đánh dấu `[OPEN — Ox]` tại chỗ.
