# ADG CPQ — Đặc tả Use Case chi tiết

Tài liệu đặc tả use case cho hệ thống báo giá tự động ADG CPQ. Đọc kèm:

- `ADG_CPQ_ERD.dbml` — mô hình dữ liệu (cột "Bảng liên quan" trỏ tới các bảng trong file này).
- `ADG_CPQ_Swimlane.drawio` — luồng nghiệp vụ 8 phase.
- `ADG_CPQ_NghiepVu_UseCase.md` — diễn giải triết lý thiết kế.

## Quy ước

- **Actor:** Đại lý, Admin (tổng công ty), Hệ thống CPQ, Dịch vụ AI, Zalo OA, Cổng nhà máy.
- **Ký hiệu luồng phụ:** đánh theo bước của luồng chính, ví dụ *3a* là nhánh rẽ tại bước 3; *E-* là ngoại lệ (lỗi/không thoả điều kiện).
- **Phân nhóm:** nhóm **UC-A*** là cấu hình (Admin, tiền đề để pipeline chạy); nhóm **UC-0*** là pipeline báo giá (Đại lý).

## Bản đồ use case

| Nhóm | Mã | Tên use case | Actor chính | Phase |
|---|---|---|---|---|
| Cấu hình | UC-A1 | Quản lý danh mục & sản phẩm | Admin | — |
| Cấu hình | UC-A2 | Định nghĩa thông số dự án cho sản phẩm | Admin | — |
| Cấu hình | UC-A3 | Quản lý loại linh kiện, thông số & SKU linh kiện | Admin | — |
| Cấu hình | UC-A4 | Gán loại linh kiện vào sản phẩm & cấu hình công thức auto-fill | Admin | — |
| Cấu hình | UC-A5 | Quản lý phụ kiện & gán vào sản phẩm | Admin | — |
| Cấu hình | UC-A6 | Quản lý vật tư & BOM template | Admin | — |
| Cấu hình | UC-A7 | Cấu hình giá (gross, range, VAT) | Admin | — |
| Pipeline | UC-01 | Khởi tạo báo giá & chọn/tạo khách hàng | Đại lý | 1 |
| Pipeline | UC-02 | Upload ảnh hiện trường | Đại lý | 1 |
| Pipeline | UC-03 | Chọn category & sản phẩm | Đại lý | 2 |
| Pipeline | UC-04 | Nhập thông số dự án | Đại lý | 3 |
| Pipeline | UC-05 | Chọn SKU linh kiện & auto-fill thông số | Đại lý | 4 |
| Pipeline | UC-06 | Chọn phụ kiện | Đại lý | 5 |
| Pipeline | UC-07 | Tính BOM & giá nhập | Hệ thống | 6 |
| Pipeline | UC-08 | Chọn giá bán trong range | Đại lý | 7 |
| Pipeline | UC-09 | Tạo & duyệt ảnh mock-up AI | Đại lý | 8 |
| Pipeline | UC-10 | Phát hành báo giá (PDF, Zalo OA, BOM nhà máy) | Hệ thống | 8 |
| Pipeline | UC-11 | Lưu nháp & tiếp tục / sửa phase trước | Đại lý | 1–8 |

---

# Nhóm cấu hình (Admin)

## UC-A1 — Quản lý danh mục & sản phẩm

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A1 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Tạo/sửa cây danh mục và hồ sơ sản phẩm nhận diện |
| **Tiền điều kiện** | Admin đã đăng nhập, có quyền quản trị catalog |
| **Trigger** | Admin mở màn hình quản lý danh mục/sản phẩm |
| **Hậu điều kiện (thành công)** | Sản phẩm được lưu, sẵn sàng để gắn thông số/linh kiện/phụ kiện/BOM/giá |
| **Hậu điều kiện (thất bại)** | Không thay đổi dữ liệu; hiển thị lỗi |
| **Bảng liên quan** | `category`, `product` |

**Luồng chính**

1. Admin chọn tạo hoặc chỉnh danh mục; nhập tên, danh mục cha (nếu có), thứ tự.
2. Admin chọn tạo sản phẩm; nhập tên, SKU, thumbnail, danh mục, mô tả, trạng thái.
3. Hệ thống kiểm tra tính hợp lệ (SKU không trùng) và lưu.
4. Hệ thống hiển thị sản phẩm trong danh sách.

**Luồng phụ & ngoại lệ**

- **3a (SKU trùng):** hệ thống báo lỗi trùng SKU, giữ nguyên dữ liệu đang nhập để admin sửa.
- **4a (ẩn sản phẩm):** admin đặt `is_active = false`; sản phẩm không xuất hiện cho đại lý ở Phase 2 nhưng báo giá cũ vẫn giữ snapshot.
- **E-1 (danh mục cha bị xoá/vô hiệu):** hệ thống chặn xoá danh mục còn sản phẩm/con; đề nghị chuyển hoặc vô hiệu hoá.

**Business rules**

- SKU sản phẩm là duy nhất toàn hệ thống.
- Vô hiệu hoá thay cho xoá cứng để bảo toàn dữ liệu báo giá lịch sử.

---

## UC-A2 — Định nghĩa thông số dự án cho sản phẩm

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A2 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Khai báo bộ thông số dự án riêng của từng sản phẩm |
| **Tiền điều kiện** | Sản phẩm đã tồn tại (UC-A1) |
| **Trigger** | Admin mở tab "Thông số dự án" của một sản phẩm |
| **Hậu điều kiện (thành công)** | Thông số được lưu, dùng ở Phase 3 và làm biến `proj.*` cho công thức |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi cụ thể |
| **Bảng liên quan** | `project_param_def`, `project_param_option`, `unit_of_measure` |

**Luồng chính**

1. Admin thêm một thông số: nhập tên, `code`, kiểu dữ liệu (number/text/boolean/date/select), kiểu nhập (freeform/dropdown), đơn vị tính, bắt buộc, thứ tự.
2. Nếu kiểu nhập là dropdown, admin nhập danh sách option (value, label).
3. Hệ thống kiểm tra `code` không trùng trong cùng sản phẩm và lưu.

**Luồng phụ & ngoại lệ**

- **1a (đơn vị chưa có):** admin mở UC phụ "Quản lý đơn vị tính" để thêm rồi quay lại.
- **2a (freeform):** bỏ qua bước option.
- **E-1 (`code` trùng trong sản phẩm):** báo lỗi, không lưu.
- **E-2 (`code` đang được công thức tham chiếu mà admin đổi/xoá):** cảnh báo các công thức bị ảnh hưởng (UC-A4/A6), yêu cầu xác nhận.

**Business rules**

- `code` là định danh ổn định dùng trong công thức; hạn chế đổi sau khi đã có công thức tham chiếu.
- Thông số là riêng theo sản phẩm, không dùng chung giữa các sản phẩm.

---

## UC-A3 — Quản lý loại linh kiện, thông số & SKU linh kiện

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A3 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Khai báo loại linh kiện dùng chung, bộ thông số và các SKU linh kiện |
| **Tiền điều kiện** | Admin đã đăng nhập |
| **Trigger** | Admin mở màn hình "Linh kiện" |
| **Hậu điều kiện (thành công)** | SKU linh kiện sẵn sàng để đại lý chọn (Phase 4); thông số mặc định làm biến `comp.*` |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `component_type`, `component_param_def`, `component_param_option`, `component_sku`, `component_sku_param_value` |

**Luồng chính**

1. Admin tạo loại linh kiện (`component_type`): tên, code.
2. Admin khai bộ thông số của loại (`component_param_def`): tên, code, kiểu dữ liệu, kiểu nhập, đơn vị, bắt buộc.
3. Admin tạo các SKU (`component_sku`): SKU, tên, đơn giá, đơn vị, thuộc loại linh kiện.
4. Với mỗi SKU, admin nhập giá trị thông số mặc định (`component_sku_param_value`).
5. Hệ thống kiểm tra và lưu.

**Luồng phụ & ngoại lệ**

- **2a (thông số dropdown):** admin nhập option (`component_param_option`).
- **4a (SKU không cần đủ mọi thông số):** để trống thông số không áp dụng; công thức nào tham chiếu thông số trống phải xử lý mặc định.
- **E-1 (SKU trùng):** báo lỗi.
- **E-2 (xoá loại linh kiện đang gán vào sản phẩm):** chặn; yêu cầu gỡ gán ở UC-A4 trước.

**Business rules**

- Loại linh kiện và thông số của nó dùng chung toàn hệ thống.
- Hệ thống **không** tự gợi ý SKU; đại lý tự chọn.

---

## UC-A4 — Gán loại linh kiện vào sản phẩm & cấu hình công thức auto-fill

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A4 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Xác định sản phẩm cần loại linh kiện nào và công thức auto-fill thông số linh kiện |
| **Tiền điều kiện** | Đã có sản phẩm (UC-A1), thông số dự án (UC-A2), loại linh kiện (UC-A3) |
| **Trigger** | Admin mở tab "Linh kiện của sản phẩm" |
| **Hậu điều kiện (thành công)** | `product_component_type` và `component_param_formula` được lưu; Phase 4 auto-fill hoạt động |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi cú pháp/biến |
| **Bảng liên quan** | `product_component_type`, `component_param_formula`, `project_param_def`, `component_param_def` |

**Luồng chính**

1. Admin gán một hoặc nhiều loại linh kiện vào sản phẩm; đặt bắt buộc/không, thứ tự.
2. Với mỗi thông số linh kiện cần auto-fill, admin nhập biểu thức công thức, tham chiếu biến `proj.<code>` và `comp.<code>`.
3. Admin bấm kiểm thử công thức với bộ giá trị mẫu.
4. Hệ thống validate cú pháp và sự tồn tại của các biến, rồi lưu.

**Luồng phụ & ngoại lệ**

- **2a (thông số nhập tay hoàn toàn):** admin bỏ trống công thức; Phase 4 để đại lý tự nhập.
- **3a (kết quả kiểm thử sai kỳ vọng):** admin sửa biểu thức, lặp lại bước 3.
- **E-1 (biến không tồn tại):** hệ thống chỉ ra biến sai, chặn lưu.
- **E-2 (biểu thức lỗi cú pháp / chia cho 0 khi test):** báo lỗi tại dòng công thức.

**Business rules**

- Công thức auto-fill đặt tại cặp *sản phẩm ↔ loại linh kiện*, nên cùng loại linh kiện có thể có công thức khác nhau ở các sản phẩm khác nhau.
- Đầu ra một công thức là **một giá trị của một thông số linh kiện**.

---

## UC-A5 — Quản lý phụ kiện & gán vào sản phẩm

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A5 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Khai báo phụ kiện dùng chung và gán danh sách phụ kiện khả dụng cho sản phẩm |
| **Tiền điều kiện** | Sản phẩm đã tồn tại |
| **Trigger** | Admin mở màn hình "Phụ kiện" |
| **Hậu điều kiện (thành công)** | Phụ kiện hiển thị để tick chọn ở Phase 5 |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `accessory`, `product_accessory` |

**Luồng chính**

1. Admin tạo phụ kiện: tên, thumbnail, đơn giá.
2. Admin gán phụ kiện vào một hoặc nhiều sản phẩm (`product_accessory`).
3. Hệ thống lưu.

**Luồng phụ & ngoại lệ**

- **2a (gỡ gán):** admin bỏ gán phụ kiện khỏi sản phẩm; báo giá cũ vẫn giữ snapshot phụ kiện đã chọn.
- **E-1 (gán trùng cặp sản phẩm–phụ kiện):** hệ thống bỏ qua bản trùng.

**Business rules**

- Ở báo giá, phụ kiện chỉ tick chọn, số lượng luôn = 1.

---

## UC-A6 — Quản lý vật tư & BOM template

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A6 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Quản lý danh mục vật tư (đơn giá) và BOM template theo công thức tính số lượng |
| **Tiền điều kiện** | Sản phẩm và thông số dự án đã có |
| **Trigger** | Admin mở màn hình "Vật tư / BOM" |
| **Hậu điều kiện (thành công)** | BOM template active cho sản phẩm; Phase 6 tính được cost_material |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi công thức |
| **Bảng liên quan** | `material`, `bom_template`, `bom_template_line`, `project_param_def` |

**Luồng chính**

1. Admin tạo/sửa vật tư (`material`): code, tên, đơn vị, đơn giá hiện hành.
2. Admin tạo BOM template cho sản phẩm (`bom_template`).
3. Admin thêm các dòng BOM (`bom_template_line`): chọn vật tư, nhập `quantity_formula` (chỉ dùng biến `proj.*`).
4. Admin kiểm thử với bộ thông số mẫu để xem tổng lượng và giá vốn NVL.
5. Hệ thống validate và lưu; đặt template `is_active`.

**Luồng phụ & ngoại lệ**

- **3a (số lượng cố định):** công thức là hằng số (không tham chiếu biến).
- **4a (kết quả bất hợp lý):** admin sửa công thức/đơn giá, lặp lại bước 4.
- **E-1 (công thức tham chiếu thông số không thuộc sản phẩm):** chặn lưu.
- **E-2 (vật tư thiếu đơn giá):** cảnh báo, không cho active template.

**Business rules**

- Đơn giá vật tư chỉ lưu **một giá hiện hành** (MVP, không lịch sử).
- Linh kiện và phụ kiện **không** tác động BOM.
- Đơn giá vật tư và dòng BOM là dữ liệu mật, ẩn với đại lý.

---

## UC-A7 — Cấu hình giá (gross, range, VAT)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A7 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Đặt lợi nhuận gộp, khoảng giá bán và VAT |
| **Tiền điều kiện** | Sản phẩm đã tồn tại; đại lý đã tồn tại (cho gross riêng) |
| **Trigger** | Admin mở màn hình "Cấu hình giá" |
| **Hậu điều kiện (thành công)** | Có global gross + range theo sản phẩm, gross đại lý theo sản phẩm (nếu có), VAT chung |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `product_pricing_config`, `dealer_product_gross`, `dealer`, `system_setting` |

**Luồng chính**

1. Admin đặt `global_gross_percent` và range (band_percent và/hoặc min–max) cho sản phẩm.
2. Admin đặt mức gross riêng cho cặp đại lý–sản phẩm (`dealer_product_gross`) khi cần.
3. Admin đặt VAT chung toàn hệ thống (`system_setting.vat_percent`).
4. Hệ thống lưu.

**Luồng phụ & ngoại lệ**

- **1a (không giới hạn range):** để trống band; đại lý nhập giá bán tự do ≥ giá nhập (theo chính sách).
- **2a (không đặt gross riêng):** hệ thống dùng `dealer.default_gross_percent`.
- **E-1 (min > max):** báo lỗi khoảng giá không hợp lệ.

**Business rules**

- `gross% = global_gross% + gross đại lý`, trong đó gross đại lý theo-sản-phẩm **thay thế** mức chung của đại lý nếu tồn tại.
- Toàn bộ cấu hình giá ẩn với đại lý.

---

# Nhóm pipeline báo giá (Đại lý)

## UC-01 — Khởi tạo báo giá & chọn/tạo khách hàng (Phase 1)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-01 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Tạo báo giá mới và gắn khách hàng |
| **Tiền điều kiện** | Đại lý đã đăng nhập |
| **Trigger** | Đại lý bấm "Tạo báo giá" |
| **Hậu điều kiện (thành công)** | `quote` `draft` gắn `customer_id`, `current_phase = customer` |
| **Hậu điều kiện (thất bại)** | Không tạo được khách/báo giá; báo lỗi |
| **Bảng liên quan** | `quote`, `customer`, `dealer` |

**Luồng chính**

1. Đại lý bấm tạo báo giá; hệ thống tạo bản nháp `quote`.
2. Đại lý nhập SĐT khách.
3. Hệ thống tra kho khách của đại lý theo SĐT.
4. Nếu chưa có, đại lý nhập tên, địa chỉ (số nhà, đường, phường/xã, tỉnh/thành).
5. Hệ thống lưu khách và gắn vào báo giá.

**Luồng phụ & ngoại lệ**

- **3a (khách đã tồn tại):** hệ thống nạp lại thông tin khách cũ; đại lý xác nhận hoặc chỉnh sửa.
- **4a (chỉnh khách cũ):** cập nhật `customer`; áp dụng cho báo giá mới, không hồi tố báo giá đã gửi.
- **E-1 (thiếu trường bắt buộc):** chặn lưu, chỉ rõ trường thiếu.

**Business rules**

- Khách được định danh trong phạm vi đại lý bằng SĐT (`unique(dealer_id, phone)`).

---

## UC-02 — Upload ảnh hiện trường (Phase 1)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-02 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Đính đúng một ảnh hiện trường vào báo giá |
| **Tiền điều kiện** | Báo giá đã có khách hàng (UC-01) |
| **Trigger** | Đại lý mở bước upload ảnh |
| **Hậu điều kiện (thành công)** | `quote.field_photo_url` được set; `current_phase = product`; đã lưu nháp |
| **Hậu điều kiện (thất bại)** | Ảnh không được lưu; giữ ảnh cũ nếu có |
| **Bảng liên quan** | `quote` |

**Luồng chính**

1. Đại lý chọn một ảnh hiện trường.
2. Hệ thống kiểm tra định dạng và dung lượng ≤ 8MB.
3. Hệ thống lưu ảnh, cập nhật `field_photo_url`, lưu nháp.

**Luồng phụ & ngoại lệ**

- **1a (thay ảnh):** đại lý tải ảnh mới thay ảnh cũ (luôn giữ tối đa 1 ảnh).
- **E-1 (quá 8MB):** báo lỗi dung lượng, không lưu.
- **E-2 (sai định dạng):** báo lỗi định dạng.
- **E-3 (chọn nhiều hơn 1 ảnh):** chỉ nhận 1 ảnh, cảnh báo.

**Business rules**

- Ảnh hiện trường là **bắt buộc**, tối đa 1 ảnh, ≤ 8MB.

---

## UC-03 — Chọn category & sản phẩm (Phase 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-03 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn sản phẩm cần báo giá và nạp cấu hình của nó |
| **Tiền điều kiện** | Đã có ảnh hiện trường (UC-02) |
| **Trigger** | Đại lý mở bước chọn sản phẩm |
| **Hậu điều kiện (thành công)** | `quote.product_id` được gán; cấu hình sản phẩm đã nạp; `current_phase = project_params` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; cảnh báo |
| **Bảng liên quan** | `category`, `product`, `project_param_def`, `product_component_type`, `product_accessory`, `bom_template`, `product_pricing_config` |

**Luồng chính**

1. Đại lý chọn danh mục.
2. Đại lý chọn sản phẩm (tên, SKU, thumbnail).
3. Hệ thống nạp thông số dự án, loại linh kiện, phụ kiện, BOM và cấu hình giá của sản phẩm.

**Luồng phụ & ngoại lệ**

- **2a (đổi sản phẩm sau khi đã nhập dữ liệu phase sau):** hệ thống cảnh báo sẽ xoá dữ liệu thông số/linh kiện/phụ kiện đã nhập; yêu cầu xác nhận.
- **E-1 (sản phẩm chưa cấu hình đủ — thiếu BOM/giá):** cảnh báo và không cho đi tiếp.
- **E-2 (sản phẩm đã bị vô hiệu):** không hiển thị trong danh sách.

**Business rules**

- Một báo giá chỉ gắn đúng **một** sản phẩm.

---

## UC-04 — Nhập thông số dự án (Phase 3)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-04 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Nhập các thông số dự án của sản phẩm |
| **Tiền điều kiện** | Đã chọn sản phẩm (UC-03) |
| **Trigger** | Đại lý mở bước thông số dự án |
| **Hậu điều kiện (thành công)** | `quote_project_param` đầy đủ; `current_phase = components` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; báo lỗi validate |
| **Bảng liên quan** | `project_param_def`, `project_param_option`, `quote_project_param` |

**Luồng chính**

1. Hệ thống hiển thị các thông số theo `project_param_def` của sản phẩm.
2. Đại lý nhập freeform (đúng kiểu dữ liệu) hoặc chọn từ dropdown.
3. Hệ thống validate và lưu vào `quote_project_param` kèm snapshot tên/đơn vị.

**Luồng phụ & ngoại lệ**

- **2a (dropdown):** đại lý chọn từ `project_param_option`.
- **E-1 (sai kiểu dữ liệu):** báo lỗi tại trường.
- **E-2 (thiếu thông số bắt buộc):** chặn chuyển bước.

**Business rules**

- Giá trị thông số dự án là nguồn biến `proj.*` cho công thức auto-fill và BOM.

---

## UC-05 — Chọn SKU linh kiện & auto-fill thông số (Phase 4)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-05 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn SKU cho từng loại linh kiện và điền/chỉnh thông số linh kiện |
| **Tiền điều kiện** | Đã nhập đủ thông số dự án (UC-04) |
| **Trigger** | Đại lý mở bước linh kiện |
| **Hậu điều kiện (thành công)** | `quote_component` + `quote_component_param`; `current_phase = accessories` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước |
| **Bảng liên quan** | `product_component_type`, `component_sku`, `component_sku_param_value`, `component_param_formula`, `quote_component`, `quote_component_param` |

**Luồng chính**

1. Hệ thống liệt kê các loại linh kiện của sản phẩm.
2. Đại lý chọn một SKU cho từng loại và nhập số lượng.
3. Hệ thống chạy `component_param_formula` (biến `proj.*` + `comp.*`) để auto-fill thông số linh kiện vào `quote_component_param`.
4. Đại lý xem lại; chỉnh giá trị nếu cần (đặt `is_overridden = true`).
5. Hệ thống lưu.

**Luồng phụ & ngoại lệ**

- **3a (thông số nhập tay không có công thức):** đại lý tự nhập.
- **4a (đại lý sửa auto-fill):** lưu giá trị mới, đánh dấu overridden.
- **E-1 (loại linh kiện bắt buộc chưa chọn SKU):** chặn chuyển bước.
- **E-2 (công thức lỗi khi chạy):** ghi log, để trống giá trị và cho đại lý nhập tay, cảnh báo admin.
- *(Sau MVP)* **4b (kiểm tra tương thích):** cảnh báo khi SKU không phù hợp thông số dự án.

**Business rules**

- Hệ thống không gợi ý SKU; đại lý tự chọn.
- Số lượng linh kiện do đại lý nhập; đơn giá × số lượng cộng vào cost_material.

---

## UC-06 — Chọn phụ kiện (Phase 5)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-06 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Tick chọn các phụ kiện đi kèm |
| **Tiền điều kiện** | Đã hoàn tất linh kiện (UC-05) |
| **Trigger** | Đại lý mở bước phụ kiện |
| **Hậu điều kiện (thành công)** | `quote_accessory` cho các phụ kiện đã tick; `current_phase = bom` |
| **Hậu điều kiện (thất bại)** | — (bước không bắt buộc chọn) |
| **Bảng liên quan** | `product_accessory`, `accessory`, `quote_accessory` |

**Luồng chính**

1. Hệ thống liệt kê phụ kiện khả dụng của sản phẩm (tên, thumbnail, đơn giá).
2. Đại lý tick chọn các phụ kiện muốn thêm.
3. Hệ thống ghi mỗi phụ kiện được chọn thành một dòng `quote_accessory` (số lượng = 1).

**Luồng phụ & ngoại lệ**

- **2a (không chọn phụ kiện nào):** cho phép; không tạo dòng nào.
- **2b (bỏ tick phụ kiện đã chọn):** xoá dòng tương ứng.

**Business rules**

- Phụ kiện luôn số lượng = 1; đơn giá cộng một lần vào cost_material.

---

## UC-07 — Tính BOM & giá nhập (Phase 6)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-07 |
| **Actor chính** | Hệ thống CPQ |
| **Actor phụ** | Đại lý (kích hoạt) |
| **Mục tiêu** | Tính giá vốn NVL và giá nhập (ẩn phần mật) |
| **Tiền điều kiện** | Đã có thông số dự án, linh kiện, phụ kiện |
| **Trigger** | Đại lý chuyển sang Phase 6 |
| **Hậu điều kiện (thành công)** | `quote_bom_line`, `cost_material`, `applied_gross_percent`, `import_price` được ghi; `current_phase = pricing` |
| **Hậu điều kiện (thất bại)** | Dừng tính; báo lỗi cấu hình cho admin |
| **Bảng liên quan** | `bom_template`, `bom_template_line`, `material`, `quote_bom_line`, `dealer`, `dealer_product_gross`, `product_pricing_config`, `quote` |

**Luồng chính**

1. Hệ thống chạy `quantity_formula` từng dòng BOM (biến `proj.*`) để ra số lượng vật tư.
2. Hệ thống tính tiền NVL = Σ(số lượng × đơn giá vật tư), ghi `quote_bom_line` (ẩn).
3. Hệ thống cộng tiền linh kiện (Σ đơn giá × số lượng) và tiền phụ kiện (Σ đơn giá) → `cost_material`.
4. Hệ thống resolve gross: `global_gross%` + (gross đại lý theo sản phẩm nếu có, ngược lại `default_gross%`).
5. Hệ thống tính `import_price = cost_material × (1 + gross%/100)` và lưu (ẩn cost_material/gross).

**Luồng phụ & ngoại lệ**

- **1a (công thức hằng số):** trả về số lượng cố định.
- **E-1 (công thức lỗi / chia 0):** dừng, thông báo lỗi cấu hình, không cho đi tiếp.
- **E-2 (vật tư thiếu đơn giá):** dừng, cảnh báo admin.

**Business rules**

- `cost_material`, đơn giá vật tư, dòng BOM và % gross **ẩn hoàn toàn** với đại lý; đại lý chỉ xem `import_price`.

---

## UC-08 — Chọn giá bán trong range (Phase 7)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-08 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn giá bán hợp lệ và tính VAT/tổng |
| **Tiền điều kiện** | Đã có `import_price` (UC-07) |
| **Trigger** | Đại lý mở bước định giá |
| **Hậu điều kiện (thành công)** | `selling_price`, `vat_percent`, `vat_amount`, `total_amount` được lưu; `current_phase = finalize` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; báo lỗi range |
| **Bảng liên quan** | `product_pricing_config`, `system_setting`, `quote` |

**Luồng chính**

1. Hệ thống hiển thị giá nhập và range bình ổn (từ `product_pricing_config`).
2. Đại lý nhập/chọn giá bán trong range.
3. Hệ thống lấy VAT chung (`system_setting`) và tính `vat_amount`, `total_amount`.
4. Hệ thống lưu snapshot giá.

**Luồng phụ & ngoại lệ**

- **1a (range không giới hạn):** đại lý nhập giá bán tự do (≥ giá nhập theo chính sách).
- **E-1 (giá ngoài range/min–max):** chặn, hiển thị khoảng hợp lệ.

**Business rules**

- VAT là một mức chung toàn hệ thống; đơn vị tiền là VND.

---

## UC-09 — Tạo & duyệt ảnh mock-up AI (Phase 8)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-09 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ, Dịch vụ AI |
| **Mục tiêu** | Sinh và chọn ảnh demo sản phẩm sau hoàn thiện |
| **Tiền điều kiện** | Đã chốt giá bán (UC-08); có ảnh hiện trường và thumbnail |
| **Trigger** | Đại lý mở bước tạo ảnh mock-up |
| **Hậu điều kiện (thành công)** | Có đúng một `quote_ai_image.is_selected = true` |
| **Hậu điều kiện (thất bại)** | Không có ảnh được chọn; theo chính sách có thể phát hành không kèm ảnh |
| **Bảng liên quan** | `quote_ai_image`, `quote` |

**Luồng chính**

1. Đại lý yêu cầu tạo ảnh; hệ thống gọi Dịch vụ AI với ảnh hiện trường + thumbnail sản phẩm.
2. Dịch vụ AI trả ảnh; hệ thống lưu `quote_ai_image` (`generation_no` tăng), tăng `ai_generation_count`.
3. Đại lý xem ảnh.
4. Đại lý duyệt ảnh: đặt `is_selected = true`.

**Luồng phụ & ngoại lệ**

- **3a (chưa ưng, còn lượt < 3):** đại lý yêu cầu tạo lại; quay lại bước 1.
- **E-1 (đã đủ 3 lần):** không cho tạo thêm; buộc chọn một trong các ảnh đã có.
- **E-2 (Dịch vụ AI lỗi/timeout):** cho thử lại (không tính vào 3 lần nếu chưa sinh được ảnh) hoặc bỏ qua theo chính sách.

**Business rules**

- Tối đa 3 lần sinh ảnh cho một báo giá.
- Chỉ một ảnh được chọn để đính vào PDF.

---

## UC-10 — Phát hành báo giá: PDF, Zalo OA, BOM nhà máy (Phase 8)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-10 |
| **Actor chính** | Hệ thống CPQ |
| **Actor phụ** | Đại lý (kích hoạt), Zalo OA, Cổng nhà máy |
| **Mục tiêu** | Xuất PDF, gửi khách qua Zalo OA và gửi BOM về nhà máy |
| **Tiền điều kiện** | Đã chọn giá bán (UC-08) và có ảnh mock-up được chọn (UC-09) |
| **Trigger** | Đại lý bấm "Phát hành báo giá" |
| **Hậu điều kiện (thành công)** | `quote.status = submitted`; PDF đã gửi khách; BOM đã gửi nhà máy |
| **Hậu điều kiện (thất bại)** | PDF vẫn được tạo; bản ghi gửi lỗi ở trạng thái `failed`, cho gửi lại |
| **Bảng liên quan** | `quote`, `quote_ai_image`, `customer_delivery`, `factory_bom_dispatch` |

**Luồng chính**

1. Hệ thống sinh PDF báo giá (thông tin khách, ảnh mock-up đã duyệt, giá bán, VAT, tổng), lưu `quote_pdf_url`.
2. Hệ thống gửi PDF cho khách qua Zalo OA (`customer_delivery`).
3. Hệ thống gửi phiếu BOM về cổng nhà máy qua API/email (`factory_bom_dispatch`).
4. Hệ thống đặt `quote.status = submitted`, ghi `submitted_at`.

**Luồng phụ & ngoại lệ**

- **E-1 (gửi Zalo OA lỗi):** ghi `customer_delivery.status = failed`; cho đại lý gửi lại (UC phụ retry).
- **E-2 (gửi nhà máy lỗi):** ghi `factory_bom_dispatch.status = failed`; cho gửi lại; không chặn PDF/Zalo.
- **E-3 (thiếu ảnh mock-up):** theo chính sách chặn phát hành hoặc phát hành không kèm ảnh.

**Business rules**

- Sau phát hành, quy trình kết thúc; không theo dõi phản hồi khách.

---

## UC-11 — Lưu nháp & tiếp tục / sửa phase trước

| Trường | Nội dung |
|---|---|
| **Mã** | UC-11 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Lưu tiến độ và quay lại chỉnh sửa các bước đã qua |
| **Tiền điều kiện** | Báo giá đang ở trạng thái `draft` |
| **Trigger** | Đại lý lưu nháp hoặc mở lại báo giá dở, hoặc quay lại phase trước |
| **Hậu điều kiện (thành công)** | Nháp được giữ; khôi phục đúng `current_phase`; dữ liệu phụ thuộc được tính lại |
| **Hậu điều kiện (thất bại)** | Giữ nguyên trạng thái trước đó |
| **Bảng liên quan** | `quote` và toàn bộ nhóm `quote_*` |

**Luồng chính**

1. Đại lý lưu nháp ở phase bất kỳ; hệ thống lưu dữ liệu và `current_phase`.
2. Lần sau, đại lý mở lại báo giá dở; hệ thống khôi phục về `current_phase`.
3. Đại lý quay lại một phase trước để chỉnh sửa.

**Luồng phụ & ngoại lệ**

- **3a (đổi thông số dự án ở Phase 3):** hệ thống đánh dấu cần tính lại auto-fill (Phase 4), BOM và giá (Phase 6–7).
- **3b (đổi linh kiện/phụ kiện):** cập nhật cost_material và giá ở Phase 6–7.
- **E-1 (báo giá đã `submitted`):** không cho sửa; đề nghị tạo báo giá mới (hoặc sao chép làm mẫu — UC phụ).

**Business rules**

- Thay đổi ở phase trước làm mất hiệu lực kết quả tính ở phase sau; hệ thống buộc tính lại trước khi phát hành.

---

# Phụ lục — Use case phụ trợ cần bổ sung

Trợ lý hoàn thiện các UC sau theo cùng khuôn mẫu: quản trị người dùng & phân quyền; quản lý hồ sơ đại lý; quản lý đơn vị tính & cấu hình hệ thống; validate tương thích linh kiện (sau MVP); gửi lại/theo dõi trạng thái gửi Zalo OA và nhà máy; lịch sử giá vật tư & phiên bản BOM; báo cáo/thống kê báo giá; tìm kiếm & sao chép báo giá làm mẫu; công cụ kiểm thử công thức auto-fill và BOM.
