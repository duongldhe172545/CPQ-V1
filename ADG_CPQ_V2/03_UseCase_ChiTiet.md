# ADG CPQ — Đặc tả Use Case chi tiết (v2)

> **PHIÊN BẢN 2 (02/07/2026)** — cập nhật sau review adversarial + phỏng vấn nghiệp vụ.
> Bản gốc của BA: `docs/_ban-goc-BA/ADG_CPQ_UseCase_ChiTiet.md`.
> Mọi thay đổi truy vết về mã `D-x / F-x / O-x` trong `_bmad-output/planning-artifacts/ADG-CPQ-decision-log.md`.
> Các điểm chưa chốt được đánh dấu **[OPEN — Ox]** ngay tại chỗ.

Đọc kèm:

- `ADG_CPQ_ERD.dbml` (v2) — mô hình dữ liệu.
- `ADG_CPQ_Swimlane_v2.drawio` — luồng nghiệp vụ 4 bước (mở bằng draw.io / diagrams.net).
- `ADG_CPQ_NghiepVu_UseCase.md` (v2) — diễn giải triết lý thiết kế.

## Quy ước

- **Actor:** Đại lý, Admin (tổng công ty), Hệ thống CPQ, Dịch vụ AI, Kênh gửi khách [OPEN — O5], Cổng nhà máy [OPEN — O4]. **Chỉ 2 vai đăng nhập: admin | dealer [D9]** — khách cuối không truy cập hệ thống.
- **Ký hiệu luồng phụ:** *3a* là nhánh rẽ tại bước 3; *E-* là ngoại lệ.
- **Cấu trúc báo giá [D1]:** một báo giá = một khách hàng + **nhiều hạng mục** (mỗi hạng mục = một sản phẩm với cấu hình, ảnh hiện trường, số lượng và giá riêng). Pipeline gồm 4 bước lớn: **(1) Khách hàng → (2) Hạng mục (lặp theo từng hạng mục) → (3) Giá → (4) Hoàn thiện**.

## Bản đồ use case

| Nhóm | Mã | Tên use case | Actor chính | Bước |
|---|---|---|---|---|
| Cấu hình | UC-A1 | Quản lý danh mục & sản phẩm | Admin | — |
| Cấu hình | UC-A2 | Định nghĩa thông số dự án cho sản phẩm | Admin | — |
| Cấu hình | UC-A3 | Quản lý loại linh kiện, thông số & SKU linh kiện | Admin | — |
| Cấu hình | UC-A4 | Gán loại linh kiện vào sản phẩm: công thức auto-fill & quy tắc tương thích SKU | Admin | — |
| Cấu hình | UC-A5 | Quản lý phụ kiện & gán vào sản phẩm | Admin | — |
| Cấu hình | UC-A6 | Quản lý vật tư & BOM template | Admin | — |
| Cấu hình | UC-A7 | Cấu hình giá (gross, range min–max, VAT, hiệu lực) | Admin | — |
| Cấu hình | UC-A8 | Nhập công thức từ bảng tính (IT/Data Analyst) & nghiệm thu golden tests | Admin | — |
| Pipeline | UC-01 | Khởi tạo báo giá & chọn/tạo khách hàng | Đại lý | 1 |
| Pipeline | UC-02 | Quản lý hạng mục báo giá (thêm/xoá/sao chép) | Đại lý | 2 |
| Pipeline | UC-03 | Chọn sản phẩm cho hạng mục | Đại lý | 2 |
| Pipeline | UC-04 | Upload ảnh hiện trường cho hạng mục | Đại lý | 2 |
| Pipeline | UC-05 | Nhập thông số dự án của hạng mục | Đại lý | 2 |
| Pipeline | UC-06 | Chọn SKU linh kiện (đã lọc tương thích) & auto-fill thông số | Đại lý | 2 |
| Pipeline | UC-07 | Chọn phụ kiện cho hạng mục | Đại lý | 2 |
| Pipeline | UC-08 | Tính BOM & giá nhập từng hạng mục | Hệ thống | 3 |
| Pipeline | UC-09 | Chọn giá bán từng hạng mục; tính VAT & tổng | Đại lý | 3 |
| Pipeline | UC-10 | Tạo & duyệt ảnh mock-up AI theo hạng mục | Đại lý | 4 |
| Pipeline | UC-11 | Phát hành báo giá (tính lại, snapshot, PDF, gửi khách, gửi BOM) | Hệ thống | 4 |
| Pipeline | UC-12 | Lưu nháp & tiếp tục / quay lại sửa | Đại lý | 1–4 |
| Phụ trợ | UC-P1 | Quản trị người dùng & phân quyền | Admin | — |
| Phụ trợ | UC-P2 | Quản lý hồ sơ đại lý & gross đại lý | Admin | — |
| Phụ trợ | UC-P3 | Quản lý đơn vị tính & cấu hình hệ thống | Admin | — |
| Phụ trợ | UC-P4 | Theo dõi & gửi lại (khách / nhà máy) | Đại lý, Admin | — |
| Phụ trợ | UC-P5 | Tìm kiếm & sao chép báo giá làm mẫu | Đại lý | — |
| Phụ trợ | UC-P6 | Xem nhật ký thay đổi cấu hình (audit log) | Admin | — |
| Phụ trợ | UC-P7 | Báo cáo / thống kê báo giá *(sau MVP)* | Admin | — |

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
| **Bảng liên quan** | `category`, `product`, `config_audit_log` |

**Luồng chính**

1. Admin chọn tạo hoặc chỉnh danh mục; nhập tên, danh mục cha (nếu có), thứ tự.
2. Admin chọn tạo sản phẩm; nhập tên, SKU, thumbnail, danh mục, mô tả, trạng thái.
3. Hệ thống kiểm tra tính hợp lệ (SKU không trùng) và lưu.
4. Hệ thống hiển thị sản phẩm trong danh sách.

**Luồng phụ & ngoại lệ**

- **3a (SKU trùng):** báo lỗi trùng SKU, giữ nguyên dữ liệu đang nhập để admin sửa.
- **4a (ẩn sản phẩm):** admin đặt `is_active = false`; sản phẩm không xuất hiện cho đại lý nhưng báo giá cũ vẫn giữ snapshot [F3].
- **E-1 (danh mục cha bị xoá/vô hiệu):** chặn xoá danh mục còn sản phẩm/con; đề nghị chuyển hoặc vô hiệu hoá.

**Business rules**

- SKU sản phẩm là duy nhất toàn hệ thống.
- Vô hiệu hoá thay cho xoá cứng để bảo toàn dữ liệu báo giá lịch sử.
- Sản phẩm giữ tối giản: màu sắc/kích thước... là thông số dự án (UC-A2), không phải cột của sản phẩm. *Lưu ý ghi nhận:* nếu màu là thông số chọn lúc báo giá, ảnh dùng cho mock-up cần theo màu — xử lý khi có catalog thật.

---

## UC-A2 — Định nghĩa thông số dự án cho sản phẩm

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A2 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Khai báo bộ thông số dự án riêng của từng sản phẩm — chính là "thiết kế phiếu nhập liệu" mà đại lý điền ở Bước 2 |
| **Tiền điều kiện** | Sản phẩm đã tồn tại (UC-A1) |
| **Trigger** | Admin mở tab "Thông số dự án" của một sản phẩm |
| **Hậu điều kiện (thành công)** | Thông số được lưu, dùng ở Bước 2 và làm biến `proj.*` cho công thức |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi cụ thể |
| **Bảng liên quan** | `project_param_def`, `project_param_option`, `unit_of_measure` |

**Luồng chính**

1. Admin thêm một thông số: nhập tên, `code`, kiểu dữ liệu (number/text/boolean/date/select), kiểu nhập (freeform/dropdown), đơn vị tính, bắt buộc, thứ tự.
2. Nếu kiểu nhập là dropdown, admin nhập danh sách option (value, label). `value` là giá trị dùng trong công thức (so sánh bằng `if(proj.x = "value", ...)`).
3. Hệ thống kiểm tra `code` không trùng trong cùng sản phẩm và lưu.

**Luồng phụ & ngoại lệ**

- **1a (đơn vị chưa có):** admin mở UC-P3 để thêm rồi quay lại.
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
| **Hậu điều kiện (thành công)** | SKU linh kiện sẵn sàng để lọc & chọn ở Bước 2; thông số mặc định làm biến `comp.*` |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `component_type`, `component_param_def`, `component_param_option`, `component_sku`, `component_sku_param_value`, `config_audit_log` |

**Luồng chính**

1. Admin tạo loại linh kiện (`component_type`): tên, code.
2. Admin khai bộ thông số của loại (`component_param_def`): tên, code, kiểu dữ liệu, kiểu nhập, đơn vị, bắt buộc.
3. Admin tạo các SKU (`component_sku`): SKU, tên, đơn giá, đơn vị, thuộc loại linh kiện.
4. Với mỗi SKU, admin nhập giá trị thông số mặc định (`component_sku_param_value`).
5. Hệ thống kiểm tra và lưu; thay đổi đơn giá được ghi audit log.

**Luồng phụ & ngoại lệ**

- **2a (thông số dropdown):** admin nhập option (`component_param_option`).
- **4a (SKU không cần đủ mọi thông số):** để trống thông số không áp dụng; công thức nào tham chiếu thông số trống phải xử lý mặc định.
- **E-1 (SKU trùng):** báo lỗi.
- **E-2 (xoá loại linh kiện đang gán vào sản phẩm):** chặn; yêu cầu gỡ gán ở UC-A4 trước.

**Business rules**

- Loại linh kiện và thông số của nó dùng chung toàn hệ thống.
- [D2] Việc SKU nào hiện ra cho đại lý do **quy tắc tương thích** ở UC-A4 quyết định.

---

## UC-A4 — Gán loại linh kiện vào sản phẩm: công thức auto-fill & quy tắc tương thích SKU

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A4 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Xác định sản phẩm cần loại linh kiện nào, công thức auto-fill thông số, và điều kiện tương thích của từng SKU [D2] |
| **Tiền điều kiện** | Đã có sản phẩm (UC-A1), thông số dự án (UC-A2), loại linh kiện & SKU (UC-A3) |
| **Trigger** | Admin mở tab "Linh kiện của sản phẩm" |
| **Hậu điều kiện (thành công)** | `product_component_type`, `component_param_formula`, `component_sku_applicability` được lưu; Bước 2 lọc & auto-fill hoạt động |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi cú pháp/biến |
| **Bảng liên quan** | `product_component_type`, `component_param_formula`, `component_sku_applicability`, `project_param_def`, `component_param_def` |

**Luồng chính**

1. Admin gán một hoặc nhiều loại linh kiện vào sản phẩm; đặt bắt buộc/không, thứ tự.
2. Với mỗi thông số linh kiện cần auto-fill, admin nhập biểu thức công thức (biến `proj.<code>`, `comp.<code>`, `calc.<code>`; hỗ trợ `if(...)`, `lookup(...)` [D10]).
3. **[D2]** Với mỗi SKU của loại linh kiện, admin khai **điều kiện tương thích** (`condition_expression` — biểu thức true/false theo `proj.*`/`comp.*`; để trống = luôn khả dụng cho sản phẩm này). SKU không được khai = không khả dụng cho sản phẩm.
4. Admin bấm kiểm thử: nhập bộ giá trị thông số mẫu → hệ thống hiển thị danh sách SKU lọt qua bộ lọc và kết quả auto-fill.
5. Hệ thống validate cú pháp và sự tồn tại của các biến, rồi lưu.

**Luồng phụ & ngoại lệ**

- **2a (thông số nhập tay hoàn toàn):** admin bỏ trống công thức; Bước 2 để đại lý tự nhập.
- **4a (kết quả kiểm thử sai kỳ vọng):** admin sửa biểu thức, lặp lại bước 4.
- **E-1 (biến không tồn tại):** hệ thống chỉ ra biến sai, chặn lưu.
- **E-2 (biểu thức lỗi cú pháp / chia cho 0 khi test):** báo lỗi tại dòng công thức.
- **E-3 (bộ lọc cho ra 0 SKU với bộ thông số hợp lệ khi test):** cảnh báo admin — đại lý sẽ bị kẹt ở tổ hợp thông số đó.

**Business rules**

- Công thức auto-fill và điều kiện tương thích đặt tại cặp *sản phẩm ↔ loại linh kiện*: cùng loại linh kiện có thể khác công thức/điều kiện giữa các sản phẩm.
- Đầu ra một công thức auto-fill là **một giá trị của một thông số linh kiện**; đầu ra điều kiện tương thích là **true/false**.
- [D2] Hệ thống **lọc** SKU tương thích; trong danh sách đã lọc, **đại lý tự chọn** (hệ thống không tự quyết).

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
| **Hậu điều kiện (thành công)** | Phụ kiện hiển thị để tick chọn ở Bước 2 |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `accessory`, `product_accessory`, `config_audit_log` |

**Luồng chính**

1. Admin tạo phụ kiện: tên, thumbnail, đơn giá (thay đổi giá ghi audit log).
2. Admin gán phụ kiện vào một hoặc nhiều sản phẩm (`product_accessory`).
3. Hệ thống lưu.

**Luồng phụ & ngoại lệ**

- **2a (gỡ gán):** admin bỏ gán phụ kiện khỏi sản phẩm; báo giá cũ vẫn giữ snapshot phụ kiện đã chọn.
- **E-1 (gán trùng cặp sản phẩm–phụ kiện):** hệ thống bỏ qua bản trùng.

**Business rules**

- Ở báo giá, phụ kiện chỉ tick chọn, số lượng luôn = 1 (mỗi hạng mục).

---

## UC-A6 — Quản lý vật tư & BOM template

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A6 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Quản lý danh mục vật tư (đơn giá), **bảng tra**, **biến trung gian** và BOM template theo công thức tính số lượng [D10] |
| **Tiền điều kiện** | Sản phẩm và thông số dự án đã có |
| **Trigger** | Admin (IT/Data Analyst — dùng vai admin) mở màn hình "Vật tư / BOM" |
| **Hậu điều kiện (thành công)** | BOM template active cho sản phẩm; Bước 3 tính được cost_material |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi công thức |
| **Bảng liên quan** | `material`, `bom_template`, `bom_template_line`, `lookup_table`, `lookup_entry`, `product_calc_var`, `project_param_def`, `config_audit_log` |

**Luồng chính**

1. Admin tạo/sửa vật tư (`material`): code, tên, đơn vị, đơn giá hiện hành (thay đổi giá ghi audit log).
2. Admin tạo các **bảng tra** khi Excel gốc có ma trận định mức/VLOOKUP (`lookup_table` + `lookup_entry`: khoảng trục 1 [× trục 2] → giá trị; các khoảng không chồng lấn).
3. Admin khai các **biến trung gian** của sản phẩm (`product_calc_var`): `calc.<code>` với công thức riêng, tính theo thứ tự — thay kiểu "ô tham chiếu ô" của Excel (vd `calc.so_nan = ceil(proj.height / 0.08)`).
4. Admin tạo BOM template cho sản phẩm (`bom_template`).
5. Admin thêm các dòng BOM (`bom_template_line`): chọn vật tư, nhập `quantity_formula` (biến `proj.*`/`calc.*`; hỗ trợ `if(...)`, `lookup(...)` và các hàm `round/ceil/floor/min/max` [D10]).
6. Admin kiểm thử với bộ thông số mẫu để xem tổng lượng và giá vốn NVL.
7. Hệ thống validate và lưu; khi đặt template `is_active`, hệ thống tự bỏ active template cũ của sản phẩm **[F4: đúng một template active mỗi sản phẩm]**.

**Luồng phụ & ngoại lệ**

- **5a (số lượng cố định):** công thức là hằng số.
- **6a (kết quả bất hợp lý):** admin sửa công thức/bảng tra/đơn giá, lặp lại bước 6.
- **E-1 (công thức tham chiếu thông số/biến/bảng tra không tồn tại):** chặn lưu, chỉ đích danh phần tử sai.
- **E-2 (vật tư thiếu đơn giá):** cảnh báo, không cho active template.
- **E-3 (khoảng bảng tra chồng lấn nhau):** chặn lưu.
- **E-4 (sửa công thức/bảng tra của sản phẩm ĐÃ go-live):** hệ thống tự chạy lại bộ golden tests (UC-A8) — lệch thì không cho lưu thành bản dùng thật.

**Business rules**

- Đơn giá vật tư chỉ lưu **một giá hiện hành** (MVP, không lịch sử).
- Linh kiện và phụ kiện **không** tác động BOM.
- Đơn giá vật tư và dòng BOM là dữ liệu mật, ẩn với đại lý — thực thi ở tầng API.
- Tra ngoài khoảng bảng tra / chia 0 / biến thiếu → công thức trả **lỗi có mã**, không trả 0 âm thầm.
- **[OPEN — chờ công ty]** Nội dung công thức cụ thể chưa chốt; khi có bảng tính chính thức, **IT/Data Analyst đọc file Excel và nhập công thức + bảng tra vào hệ thống**, nghiệm thu qua UC-A8.

---

## UC-A7 — Cấu hình giá (gross, range min–max, VAT, hiệu lực)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A7 |
| **Actor chính** | Admin |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Đặt lợi nhuận gộp, khoảng giá bán min–max [D3], VAT chung [D7] và các cấu hình vòng đời báo giá |
| **Tiền điều kiện** | Sản phẩm đã tồn tại; đại lý đã tồn tại (cho gross riêng) |
| **Trigger** | Admin mở màn hình "Cấu hình giá" |
| **Hậu điều kiện (thành công)** | Có global gross + range min–max theo sản phẩm, gross đại lý theo sản phẩm (nếu có), VAT chung, cấu hình hiệu lực/mock-up |
| **Hậu điều kiện (thất bại)** | Không lưu; báo lỗi |
| **Bảng liên quan** | `product_pricing_config`, `dealer_product_gross`, `dealer`, `system_setting`, `config_audit_log` |

**Luồng chính**

1. Admin đặt `global_gross_percent` và range `min_selling_price`–`max_selling_price` cho sản phẩm **[D3: chỉ một cơ chế min–max, đã bỏ ±%] [OPEN — O1: giá trị cụ thể chờ công ty ban hành]**.
2. Admin đặt mức gross riêng cho cặp đại lý–sản phẩm (`dealer_product_gross`) khi cần.
3. Admin đặt các cấu hình hệ thống: `vat_percent` [D7], `quote_validity_days` [OPEN — O3], `mockup_required` [D6], `allow_below_import` [OPEN — O2].
4. Hệ thống lưu; mọi thay đổi ghi audit log.

**Luồng phụ & ngoại lệ**

- **1a (không giới hạn range):** để trống min–max; đại lý nhập giá bán tự do nhưng vẫn áp sàn "giá bán ≥ giá nhập" trừ khi `allow_below_import` bật [OPEN — O2].
- **2a (không đặt gross riêng):** hệ thống dùng `dealer.default_gross_percent`.
- **E-1 (min > max):** báo lỗi khoảng giá không hợp lệ.

**Business rules**

- `gross% = global_gross% + gross đại lý`, trong đó gross đại lý theo-sản-phẩm **thay thế** mức chung của đại lý nếu tồn tại (không cộng dồn).
- Toàn bộ cấu hình giá ẩn với đại lý — thực thi ở tầng API.
- [D5] Việc đại lý chọn giá trong min–max chính là công cụ điều chỉnh giá/chiết khấu ở MVP; các chính sách khuyến mãi từ công ty là hạng mục sau MVP — thiết kế không hard-code giá để chừa hook.

---

## UC-A8 — Nhập công thức từ bảng tính (IT/Data Analyst) & nghiệm thu golden tests *(MỚI — [D10], cập nhật 03/07: nhập tay thay AI-import)*

| Trường | Nội dung |
|---|---|
| **Mã** | UC-A8 |
| **Actor chính** | Admin — cụ thể là **IT/Data Analyst của công ty, dùng vai admin** |
| **Actor phụ** | Hệ thống CPQ, người chủ file Excel (người làm giá) |
| **Mục tiêu** | Chuyển toàn bộ logic tính BOM/thông số từ bảng tính Excel của công ty vào cấu hình hệ thống (**nhập tay có phương pháp**), nghiệm thu bằng đối chiếu tự động |
| **Tiền điều kiện** | Bảng tính chính thức đã được công ty chốt **[OPEN — chờ công ty]**; sản phẩm & thông số dự án đã khai (UC-A1/A2) |
| **Trigger** | Có bảng tính chốt cho một sản phẩm |
| **Hậu điều kiện (thành công)** | Công thức trong hệ thống cho ra kết quả **khớp 100%** với Excel trên bộ golden tests; sản phẩm sẵn sàng go-live |
| **Hậu điều kiện (thất bại)** | Sản phẩm chưa go-live; danh sách ca lệch được ghi lại để xử lý |
| **Bảng liên quan** | `bom_template`, `bom_template_line`, `lookup_table`, `lookup_entry`, `product_calc_var`, `component_param_formula`, `component_sku_applicability`, `material`, `golden_test_case` |

**Luồng chính**

1. IT/DA nhận bảng tính + buổi khai thác logic ngầm với người chủ file (ô sửa tay, bảng tra phụ, ngoại lệ không ghi trong file).
2. IT/DA **đọc và dịch logic Excel, nhập tay vào hệ thống** qua UC-A6/UC-A4: dòng BOM + công thức số lượng, **bảng tra** (thay VLOOKUP/ma trận định mức), **biến trung gian** (thay ô-tham-chiếu-ô), công thức auto-fill, điều kiện tương thích SKU.
3. IT/DA xây bộ **golden tests** (`golden_test_case`): N bộ thông số thật (bao gồm các ca biên) kèm kết quả mong đợi lấy từ Excel gốc, có xác nhận của người làm giá.
4. Hệ thống chạy toàn bộ golden tests, so kết quả từng dòng vật tư/giá trị.
5. Khớp 100% → admin duyệt go-live cho sản phẩm; bộ golden tests lưu lại và **tự chạy hồi quy mỗi khi công thức/bảng tra/BOM của sản phẩm bị sửa** — lệch thì chặn (UC-A6 E-4).

**Luồng phụ & ngoại lệ**

- **4a (có ca lệch):** liệt kê từng ca (input, kết quả Excel, kết quả hệ thống); IT/DA sửa công thức/bảng tra rồi chạy lại.
- **E-1 (logic Excel mâu thuẫn nội bộ):** ghi nhận, đưa người chủ file quyết định đâu là đúng — không tự đoán.
- **E-2 (Excel có logic ngôn ngữ công thức chưa biểu diễn được):** ghi nhận đích danh, báo đội phát triển mở rộng ngôn ngữ công thức — không "ước lượng gần đúng".

**Business rules**

- Sau go-live, **hệ thống là nguồn chân lý duy nhất** — công ty khoá việc sửa file Excel làm căn cứ giá; sửa công thức = sửa trên hệ thống (có audit log).
- Không go-live sản phẩm nào chưa qua golden tests.
- Nhập tay là phương pháp chính thức (chốt 03/07) — không phụ thuộc công cụ import tự động.

---

# Nhóm pipeline báo giá (Đại lý)

## UC-01 — Khởi tạo báo giá & chọn/tạo khách hàng (Bước 1)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-01 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Tạo báo giá mới và gắn khách hàng |
| **Tiền điều kiện** | Đại lý đã đăng nhập |
| **Trigger** | Đại lý bấm "Tạo báo giá" |
| **Hậu điều kiện (thành công)** | `quote` `draft` gắn `customer_id`, `current_phase = customer` → chuyển `items` |
| **Hậu điều kiện (thất bại)** | Không tạo được khách/báo giá; báo lỗi |
| **Bảng liên quan** | `quote`, `customer`, `dealer` |

**Luồng chính**

1. Đại lý bấm tạo báo giá; hệ thống tạo bản nháp `quote` **(chưa yêu cầu ảnh — ảnh hiện trường thuộc từng hạng mục [F1][D1])**.
2. Đại lý nhập SĐT khách.
3. Hệ thống tra kho khách của đại lý theo SĐT.
4. Nếu chưa có, đại lý nhập tên, địa chỉ (số nhà, đường, phường/xã, tỉnh/thành).
5. Hệ thống lưu khách, gắn vào báo giá, chuyển sang Bước 2 (hạng mục).

**Luồng phụ & ngoại lệ**

- **3a (khách đã tồn tại):** nạp lại thông tin khách cũ; đại lý xác nhận hoặc chỉnh sửa.
- **4a (chỉnh khách cũ):** cập nhật `customer`; báo giá đã phát hành không bị ảnh hưởng nhờ snapshot [F2].
- **E-1 (thiếu trường bắt buộc):** chặn lưu, chỉ rõ trường thiếu.

**Business rules**

- Khách được định danh trong phạm vi đại lý bằng SĐT (`unique(dealer_id, phone)`).
- Thông tin khách được **snapshot vào quote tại thời điểm phát hành** [F2].

---

## UC-02 — Quản lý hạng mục báo giá (Bước 2) *(MỚI — [D1])*

| Trường | Nội dung |
|---|---|
| **Mã** | UC-02 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Thêm/xoá/sao chép các hạng mục sản phẩm trong một báo giá |
| **Tiền điều kiện** | Báo giá đã có khách hàng (UC-01), trạng thái `draft` |
| **Trigger** | Đại lý ở màn hình danh sách hạng mục |
| **Hậu điều kiện (thành công)** | Báo giá có ≥ 1 hạng mục; mỗi hạng mục theo dõi bước dở (`step`) riêng |
| **Hậu điều kiện (thất bại)** | Danh sách hạng mục không đổi |
| **Bảng liên quan** | `quote_item` |

**Luồng chính**

1. Đại lý bấm "Thêm hạng mục"; hệ thống tạo `quote_item` mới (`step = product`) và đưa vào luồng cấu hình UC-03 → UC-07.
2. Đại lý lặp lại cho các sản phẩm khác của cùng đơn hàng (VD 2 cửa cuốn + 1 cửa nhôm).
3. Với mỗi hạng mục, đại lý nhập **số lượng** (VD 2 cửa giống hệt nhau = 1 hạng mục, số lượng 2).
4. Hệ thống hiển thị danh sách hạng mục kèm trạng thái hoàn thành từng cái.

**Luồng phụ & ngoại lệ**

- **1a (sao chép hạng mục):** nhân bản toàn bộ cấu hình của một hạng mục đã có (trừ ảnh hiện trường) rồi chỉnh lại — tiện cho các cửa gần giống nhau.
- **1b (xoá hạng mục):** xác nhận rồi xoá `quote_item` và dữ liệu con.
- **E-1 (phát hành khi có hạng mục chưa hoàn tất):** chặn ở UC-11; chỉ rõ hạng mục nào đang dở bước nào.

**Business rules**

- Một báo giá phải có **ít nhất một** hạng mục để phát hành.
- Giá và mock-up tính/tạo **theo từng hạng mục**; tổng tiền và VAT tính ở cấp báo giá.

---

## UC-03 — Chọn sản phẩm cho hạng mục (Bước 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-03 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn sản phẩm cho hạng mục và nạp cấu hình của nó |
| **Tiền điều kiện** | Hạng mục đã được tạo (UC-02) |
| **Trigger** | Đại lý mở hạng mục ở bước `product` |
| **Hậu điều kiện (thành công)** | `quote_item.product_id` được gán; cấu hình sản phẩm đã nạp; `step = photo` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; cảnh báo |
| **Bảng liên quan** | `category`, `product`, `project_param_def`, `product_component_type`, `product_accessory`, `bom_template`, `product_pricing_config` |

**Luồng chính**

1. Đại lý chọn danh mục (cây nhiều cấp).
2. Đại lý chọn sản phẩm (tên, SKU, thumbnail).
3. Hệ thống nạp thông số dự án, loại linh kiện, phụ kiện, BOM và cấu hình giá của sản phẩm cho hạng mục.

**Luồng phụ & ngoại lệ**

- **2a (đổi sản phẩm sau khi đã nhập dữ liệu bước sau):** cảnh báo sẽ xoá dữ liệu thông số/linh kiện/phụ kiện của hạng mục; yêu cầu xác nhận.
- **E-1 (sản phẩm chưa cấu hình đủ — thiếu BOM active/giá):** cảnh báo và không cho đi tiếp.
- **E-2 (sản phẩm đã bị vô hiệu):** không hiển thị trong danh sách.

**Business rules**

- Một hạng mục gắn đúng **một** sản phẩm; muốn thêm sản phẩm khác thì thêm hạng mục (UC-02).
- Thông tin sản phẩm được snapshot vào hạng mục khi phát hành [F3].

---

## UC-04 — Upload ảnh hiện trường cho hạng mục (Bước 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-04 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Đính đúng một ảnh hiện trường vào hạng mục (vị trí lắp của cửa đó) — đầu vào cho mock-up AI |
| **Tiền điều kiện** | Hạng mục đã chọn sản phẩm (UC-03) |
| **Trigger** | Đại lý ở bước `photo` của hạng mục |
| **Hậu điều kiện (thành công)** | `quote_item.field_photo_url` được set; `step = params`; đã lưu nháp |
| **Hậu điều kiện (thất bại)** | Ảnh không được lưu; giữ ảnh cũ nếu có |
| **Bảng liên quan** | `quote_item` |

**Luồng chính**

1. Hệ thống hiển thị **hướng dẫn chụp** [D6]: chụp chính diện vị trí lắp, đủ sáng, không che khuất.
2. Đại lý chọn/chụp một ảnh hiện trường.
3. Hệ thống kiểm tra định dạng, dung lượng ≤ 8MB và **độ phân giải tối thiểu** [D6].
4. Hệ thống lưu ảnh, cập nhật `field_photo_url`, lưu nháp.

**Luồng phụ & ngoại lệ**

- **2a (thay ảnh):** tải ảnh mới thay ảnh cũ (luôn giữ tối đa 1 ảnh/hạng mục); nếu hạng mục đã có ảnh mock-up, cảnh báo lượt gen sẽ **reset** và ảnh mock-up cũ bị bỏ chọn [D6].
- **E-1 (quá 8MB):** báo lỗi dung lượng, không lưu.
- **E-2 (sai định dạng / dưới độ phân giải tối thiểu):** báo lỗi cụ thể.
- **E-3 (bỏ qua tạm thời):** cho phép đi tiếp các bước cấu hình khác, nhưng **bắt buộc có ảnh trước khi phát hành** [F1].

**Business rules**

- Ảnh hiện trường là **bắt buộc trước khi phát hành**, tối đa 1 ảnh/hạng mục, ≤ 8MB [F1][D1].
- MVP chưa tự đánh giá chất lượng nội dung ảnh — giới hạn đã ghi nhận; hướng dẫn chụp + kiểm tra kỹ thuật là hàng rào thứ nhất [D6].

---

## UC-05 — Nhập thông số dự án của hạng mục (Bước 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-05 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Nhập các thông số dự án của sản phẩm trong hạng mục |
| **Tiền điều kiện** | Hạng mục đã chọn sản phẩm (UC-03) |
| **Trigger** | Đại lý ở bước `params` |
| **Hậu điều kiện (thành công)** | `quote_item_param` đầy đủ; `step = components` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; báo lỗi validate |
| **Bảng liên quan** | `project_param_def`, `project_param_option`, `quote_item_param` |

**Luồng chính**

1. Hệ thống hiển thị các thông số theo `project_param_def` của sản phẩm.
2. Đại lý nhập freeform (đúng kiểu dữ liệu) hoặc chọn từ dropdown.
3. Hệ thống validate và lưu vào `quote_item_param` kèm snapshot tên/đơn vị.

**Luồng phụ & ngoại lệ**

- **2a (dropdown):** đại lý chọn từ `project_param_option`.
- **E-1 (sai kiểu dữ liệu):** báo lỗi tại trường.
- **E-2 (thiếu thông số bắt buộc):** chặn chuyển bước.

**Business rules**

- Giá trị thông số dự án là nguồn biến `proj.*` cho công thức auto-fill, điều kiện tương thích SKU và BOM.
- Sửa thông số sau khi đã cấu hình linh kiện → hệ thống chạy lại bộ lọc tương thích và auto-fill; SKU đã chọn mà không còn tương thích thì cảnh báo buộc chọn lại (xem UC-12).

---

## UC-06 — Chọn SKU linh kiện (đã lọc tương thích) & auto-fill thông số (Bước 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-06 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn SKU cho từng loại linh kiện **trong danh sách đã lọc tương thích [D2]** và điền/chỉnh thông số linh kiện |
| **Tiền điều kiện** | Đã nhập đủ thông số dự án (UC-05) |
| **Trigger** | Đại lý ở bước `components` |
| **Hậu điều kiện (thành công)** | `quote_item_component` + `quote_item_component_param`; `step = accessories` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước |
| **Bảng liên quan** | `product_component_type`, `component_sku`, `component_sku_applicability`, `component_sku_param_value`, `component_param_formula`, `quote_item_component`, `quote_item_component_param` |

**Luồng chính**

1. Hệ thống liệt kê các loại linh kiện của sản phẩm.
2. **[D2]** Với mỗi loại, hệ thống chạy `condition_expression` trên thông số dự án đã nhập và **chỉ hiển thị các SKU tương thích** (VD kích thước lớn → còn đúng 2 bộ tời hợp lệ).
3. Đại lý chọn một SKU trong danh sách đã lọc và nhập số lượng.
4. Hệ thống chạy `component_param_formula` (biến `proj.*` + `comp.*`) để auto-fill thông số linh kiện vào `quote_item_component_param`.
5. Đại lý xem lại; chỉnh giá trị nếu cần (đặt `is_overridden = true`).
6. Hệ thống lưu.

**Luồng phụ & ngoại lệ**

- **4a (thông số nhập tay không có công thức):** đại lý tự nhập.
- **5a (đại lý sửa auto-fill):** lưu giá trị mới, đánh dấu overridden.
- **E-1 (loại linh kiện bắt buộc chưa chọn SKU):** chặn chuyển bước.
- **E-2 (bộ lọc ra 0 SKU tương thích):** thông báo đại lý kiểm tra lại thông số; đồng thời cảnh báo admin lỗ hổng cấu hình (tổ hợp thông số không có SKU nào phủ).
- **E-3 (công thức lỗi khi chạy):** ghi log, để trống giá trị và cho đại lý nhập tay, cảnh báo admin.

**Business rules**

- Hệ thống **lọc**, đại lý **chọn** — hệ thống không tự quyết SKU [D2].
- Số lượng linh kiện do đại lý nhập; đơn giá × số lượng cộng vào cost_material của hạng mục.

---

## UC-07 — Chọn phụ kiện cho hạng mục (Bước 2)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-07 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Tick chọn các phụ kiện đi kèm hạng mục |
| **Tiền điều kiện** | Đã hoàn tất linh kiện (UC-06) |
| **Trigger** | Đại lý ở bước `accessories` |
| **Hậu điều kiện (thành công)** | `quote_item_accessory` cho các phụ kiện đã tick; `step = done` |
| **Hậu điều kiện (thất bại)** | — (bước không bắt buộc chọn) |
| **Bảng liên quan** | `product_accessory`, `accessory`, `quote_item_accessory` |

**Luồng chính**

1. Hệ thống liệt kê phụ kiện khả dụng của sản phẩm (tên, thumbnail, đơn giá).
2. Đại lý tick chọn các phụ kiện muốn thêm.
3. Hệ thống ghi mỗi phụ kiện được chọn thành một dòng `quote_item_accessory` (số lượng = 1); hạng mục chuyển `done`.

**Luồng phụ & ngoại lệ**

- **2a (không chọn phụ kiện nào):** cho phép; không tạo dòng nào.
- **2b (bỏ tick phụ kiện đã chọn):** xoá dòng tương ứng.

**Business rules**

- Phụ kiện luôn số lượng = 1 trong một hạng mục; đơn giá cộng một lần vào cost_material của hạng mục.

---

## UC-08 — Tính BOM & giá nhập từng hạng mục (Bước 3)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-08 |
| **Actor chính** | Hệ thống CPQ |
| **Actor phụ** | Đại lý (kích hoạt) |
| **Mục tiêu** | Tính giá vốn NVL và giá nhập cho **từng hạng mục** (ẩn phần mật) |
| **Tiền điều kiện** | Mọi hạng mục đã `done` (đủ thông số, linh kiện, phụ kiện) |
| **Trigger** | Đại lý chuyển sang Bước 3 |
| **Hậu điều kiện (thành công)** | Mỗi hạng mục có `quote_item_bom_line`, `cost_material`, `applied_gross_percent`, `import_price`; `current_phase = pricing` |
| **Hậu điều kiện (thất bại)** | Dừng tính; báo lỗi cấu hình cho admin |
| **Bảng liên quan** | `bom_template`, `bom_template_line`, `material`, `quote_item_bom_line`, `dealer`, `dealer_product_gross`, `product_pricing_config`, `quote_item` |

**Luồng chính** *(lặp cho từng hạng mục)*

1. Hệ thống chạy `quantity_formula` từng dòng BOM (biến `proj.*` của hạng mục) để ra số lượng vật tư cho **một đơn vị** hạng mục.
2. Hệ thống tính tiền NVL = Σ(số lượng × đơn giá vật tư), ghi `quote_item_bom_line` (ẩn).
3. Hệ thống cộng tiền linh kiện (Σ đơn giá × số lượng) và tiền phụ kiện (Σ đơn giá) → `cost_material`.
4. Hệ thống resolve gross: `global_gross%` + (gross đại lý theo sản phẩm nếu có, ngược lại `default_gross%`).
5. Hệ thống tính `import_price = cost_material × (1 + gross%/100)`, làm tròn về đồng, và lưu (ẩn cost_material/gross).

**Luồng phụ & ngoại lệ**

- **1a (công thức hằng số):** trả về số lượng cố định.
- **E-1 (công thức lỗi / chia 0):** dừng, thông báo lỗi cấu hình, không cho đi tiếp; cảnh báo admin.
- **E-2 (vật tư thiếu đơn giá):** dừng, cảnh báo admin.

**Business rules**

- `cost_material`, đơn giá vật tư, dòng BOM và % gross **ẩn hoàn toàn** với đại lý (thực thi tầng API); đại lý chỉ xem `import_price` từng hạng mục.
- Số tiền làm tròn về đồng tại từng dòng (quy tắc làm tròn — xác nhận với kế toán).

---

## UC-09 — Chọn giá bán từng hạng mục; tính VAT & tổng (Bước 3)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-09 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Chọn giá bán hợp lệ cho từng hạng mục và tính tổng báo giá |
| **Tiền điều kiện** | Mọi hạng mục đã có `import_price` (UC-08) |
| **Trigger** | Đại lý mở bước định giá |
| **Hậu điều kiện (thành công)** | Mỗi hạng mục có `selling_price`, `line_total`; quote có `subtotal`, `vat_percent`, `vat_amount`, `total_amount`; `current_phase = finalize` |
| **Hậu điều kiện (thất bại)** | Không chuyển bước; báo lỗi range |
| **Bảng liên quan** | `product_pricing_config`, `system_setting`, `quote_item`, `quote` |

**Luồng chính**

1. Hệ thống hiển thị từng hạng mục: giá nhập + range min–max của sản phẩm [D3].
2. Đại lý nhập/chọn giá bán cho từng hạng mục trong range **[D5: đây chính là công cụ điều chỉnh giá/chiết khấu của đại lý ở MVP]**.
3. Hệ thống tính `line_total = selling_price × quantity` từng hạng mục, `subtotal` toàn báo giá.
4. Hệ thống lấy VAT chung [D7], tính `vat_amount`, `total_amount` và lưu.

**Luồng phụ & ngoại lệ**

- **1a (sản phẩm không cấu hình range):** đại lý nhập giá tự do nhưng vẫn áp sàn "giá bán ≥ giá nhập" trừ khi `allow_below_import` bật **[OPEN — O2]**.
- **E-1 (giá ngoài min–max):** chặn, hiển thị khoảng hợp lệ.

**Business rules**

- VAT là một mức chung toàn hệ thống [D7]; đơn vị tiền là VND, làm tròn về đồng.
- Giá bán chọn **theo từng hạng mục**; khách nhìn thấy đơn giá từng hạng mục + tổng.

---

## UC-10 — Tạo & duyệt ảnh mock-up AI theo hạng mục (Bước 4)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-10 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ, Dịch vụ AI |
| **Mục tiêu** | Sinh và chọn ảnh minh hoạ "sau khi lắp" cho từng hạng mục |
| **Tiền điều kiện** | Đã chốt giá (UC-09); hạng mục có ảnh hiện trường (UC-04) |
| **Trigger** | Đại lý mở bước hoàn thiện |
| **Hậu điều kiện (thành công)** | Mỗi hạng mục có tối đa một `quote_item_ai_image.is_selected = true` |
| **Hậu điều kiện (thất bại)** | Hạng mục không có ảnh được chọn — xử lý theo quy tắc D6 khi phát hành |
| **Bảng liên quan** | `quote_item_ai_image`, `quote_item`, `system_setting` |

**Luồng chính** *(lặp cho từng hạng mục)*

1. Đại lý yêu cầu tạo ảnh; hệ thống gọi Dịch vụ AI với ảnh hiện trường của hạng mục + thumbnail sản phẩm.
2. Dịch vụ AI trả ảnh; hệ thống lưu `quote_item_ai_image` (`generation_no` tăng), tăng `ai_generation_count` của hạng mục.
3. Đại lý xem ảnh; duyệt thì đặt `is_selected = true`.

**Luồng phụ & ngoại lệ**

- **3a (chưa ưng, còn lượt < `ai_max_regenerate`):** yêu cầu tạo lại; quay lại bước 1.
- **E-1 (đã hết lượt, không ảnh nào đạt) [D6]:** đại lý chọn một trong hai lối thoát: **(i)** thay ảnh hiện trường khác (UC-04) → lượt gen reset; **(ii)** tiếp tục không kèm mock-up cho hạng mục này → bắt buộc nhập `no_mockup_reason`, hệ thống log để công ty giám sát.
- **E-2 (Dịch vụ AI lỗi/timeout):** cho thử lại; lần gen không ra ảnh **không tính lượt** [D6].

**Business rules**

- Giới hạn `ai_max_regenerate` lần sinh ảnh **cho mỗi hạng mục**; thay ảnh hiện trường → reset lượt [D6].
- Chỉ một ảnh được chọn mỗi hạng mục để đính vào PDF.

---

## UC-11 — Phát hành báo giá (Bước 4)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-11 |
| **Actor chính** | Hệ thống CPQ |
| **Actor phụ** | Đại lý (kích hoạt), Kênh gửi khách [OPEN — O5], Cổng nhà máy [OPEN — O4] |
| **Mục tiêu** | Tính lại giá chốt, snapshot, xuất PDF, gửi khách và gửi BOM về nhà máy |
| **Tiền điều kiện** | Mọi hạng mục `done`, đã có giá bán (UC-09), có ảnh hiện trường; mock-up theo quy tắc D6 |
| **Trigger** | Đại lý bấm "Phát hành báo giá" |
| **Hậu điều kiện (thành công)** | `quote.status = submitted`; snapshot đóng băng; PDF đã tạo; các bản ghi gửi được tạo |
| **Hậu điều kiện (thất bại)** | Quote vẫn `draft` (nếu fail trước snapshot) hoặc `submitted` với bản ghi gửi `failed` cho gửi lại (UC-P4) |
| **Bảng liên quan** | `quote`, `quote_item`, `quote_item_ai_image`, `customer_delivery`, `factory_bom_dispatch` |

**Luồng chính**

1. **[D4]** Hệ thống **tính lại toàn bộ** BOM + giá của mọi hạng mục theo cấu hình mới nhất; nếu `import_price` bất kỳ hạng mục nào lệch so với nháp, hiển thị so sánh và yêu cầu đại lý xác nhận lại giá bán trước khi tiếp tục.
2. Hệ thống kiểm tra điều kiện phát hành: mọi hạng mục đủ dữ liệu + ảnh hiện trường [F1]; nếu `mockup_required` bật, hạng mục thiếu mock-up phải có `no_mockup_reason` [D6].
3. Hệ thống chốt snapshot: thông tin khách [F2], sản phẩm từng hạng mục [F3], mọi con số giá; tính `valid_until = ngày phát hành + quote_validity_days` nếu có cấu hình **[OPEN — O3]**.
4. Hệ thống sinh PDF báo giá (thông tin khách, bảng hạng mục: sản phẩm — số lượng — đơn giá — thành tiền, ảnh mock-up từng hạng mục, subtotal, VAT, tổng, hạn hiệu lực nếu có), lưu `quote_pdf_url`.
5. Hệ thống gửi PDF cho khách (`customer_delivery`) **[OPEN — O5: kênh gửi chốt cùng quyết định nền tảng; đại lý luôn tải được PDF để gửi thủ công]**.
6. Hệ thống gửi phiếu BOM sản xuất (gộp BOM mọi hạng mục × số lượng) về cổng nhà máy (`factory_bom_dispatch`) **[OPEN — O4]**.
7. Hệ thống đặt `quote.status = submitted`, ghi `submitted_at`. Từ thời điểm này mọi con số **đóng băng vĩnh viễn** [D4].

**Luồng phụ & ngoại lệ**

- **1a (giá không đổi so với nháp):** đi thẳng bước 2, không cần xác nhận lại.
- **E-1 (gửi khách lỗi):** ghi `customer_delivery.status = failed`; cho gửi lại (UC-P4); không rollback trạng thái submitted.
- **E-2 (gửi nhà máy lỗi):** ghi `factory_bom_dispatch.status = failed`; cho gửi lại; không chặn PDF/gửi khách.
- **E-3 (hạng mục chưa hoàn tất):** chặn phát hành, chỉ rõ hạng mục + bước đang dở.

**Business rules**

- Không bao giờ phát hành giá lỗi thời: bước tính lại (1) là bắt buộc [D4].
- Sau phát hành, quy trình kết thúc; không theo dõi phản hồi khách (phạm vi MVP đã xác nhận).

---

## UC-12 — Lưu nháp & tiếp tục / quay lại sửa (Bước 1–4)

| Trường | Nội dung |
|---|---|
| **Mã** | UC-12 |
| **Actor chính** | Đại lý |
| **Actor phụ** | Hệ thống CPQ |
| **Mục tiêu** | Lưu tiến độ và quay lại chỉnh sửa các bước/hạng mục đã qua |
| **Tiền điều kiện** | Báo giá đang ở trạng thái `draft` |
| **Trigger** | Đại lý lưu nháp, mở lại báo giá dở, hoặc quay lại bước trước |
| **Hậu điều kiện (thành công)** | Nháp được giữ; khôi phục đúng `current_phase` + `step` từng hạng mục; dữ liệu phụ thuộc được tính lại |
| **Hậu điều kiện (thất bại)** | Giữ nguyên trạng thái trước đó |
| **Bảng liên quan** | `quote`, `quote_item` và toàn bộ nhóm `quote_item_*` |

**Luồng chính**

1. Đại lý lưu nháp ở bất kỳ đâu; hệ thống lưu dữ liệu, `current_phase` của quote và `step` của từng hạng mục.
2. Lần sau, đại lý mở lại báo giá dở; hệ thống khôi phục đúng chỗ đang làm.
3. Đại lý quay lại một bước trước (của quote hoặc của một hạng mục) để chỉnh sửa.

**Luồng phụ & ngoại lệ**

- **3a (đổi thông số dự án của hạng mục):** hệ thống chạy lại bộ lọc tương thích [D2] + auto-fill; SKU đã chọn không còn tương thích → cảnh báo buộc chọn lại; đánh dấu cần tính lại BOM/giá của hạng mục.
- **3b (đổi linh kiện/phụ kiện/số lượng):** đánh dấu cần tính lại giá hạng mục và tổng.
- **3c (thay ảnh hiện trường):** reset lượt gen mock-up của hạng mục [D6].
- **E-1 (báo giá đã `submitted`):** không cho sửa; đề nghị sao chép làm báo giá mới (UC-P5).

**Business rules**

- Thay đổi ở bước trước làm mất hiệu lực kết quả tính ở bước sau **của hạng mục đó**; hệ thống buộc tính lại trước khi phát hành (và UC-11 luôn tính lại lần cuối [D4]).

---

# Nhóm phụ trợ

## UC-P1 — Quản trị người dùng & phân quyền

| Trường | Nội dung |
|---|---|
| **Actor chính** | Admin |
| **Mục tiêu** | Tạo/khoá tài khoản, gán vai trò (admin / dealer [D9]) và gắn tài khoản dealer vào đại lý |
| **Bảng liên quan** | `user_account`, `dealer` |

**Luồng chính:** (1) Admin tạo tài khoản: họ tên, email, SĐT, vai trò; nếu vai `dealer` thì bắt buộc chọn đại lý. (2) Hệ thống gửi thông tin đăng nhập ban đầu / link đặt mật khẩu. (3) Admin khoá/mở tài khoản (`is_active`).
**Ngoại lệ:** email trùng → báo lỗi; khoá tài khoản không ảnh hưởng báo giá đã tạo.
**Business rules:** các tài khoản cùng đại lý thấy chung dữ liệu đại lý đó (kho khách, báo giá); dữ liệu mật về giá ẩn với mọi tài khoản `dealer` ở tầng API.

## UC-P2 — Quản lý hồ sơ đại lý & gross đại lý

| Trường | Nội dung |
|---|---|
| **Actor chính** | Admin |
| **Mục tiêu** | Onboard đại lý mới, cập nhật hồ sơ, đặt `default_gross_percent` và gross theo sản phẩm |
| **Bảng liên quan** | `dealer`, `dealer_product_gross`, `config_audit_log` |

**Luồng chính:** (1) Admin tạo đại lý: mã, tên, SĐT, gross mặc định. (2) Tạo tài khoản cho đại lý (UC-P1). (3) Đặt gross riêng theo sản phẩm khi có thoả thuận (UC-A7 bước 2). (4) Vô hiệu hoá đại lý ngừng hợp tác — báo giá cũ giữ nguyên, không đăng nhập/tạo mới được.
**Business rules:** mã đại lý duy nhất; mọi thay đổi gross ghi audit log.

## UC-P3 — Quản lý đơn vị tính & cấu hình hệ thống

| Trường | Nội dung |
|---|---|
| **Actor chính** | Admin |
| **Mục tiêu** | Quản lý `unit_of_measure` và các `system_setting` (VAT, số lượt gen AI, mockup_required, allow_below_import, quote_validity_days) |
| **Bảng liên quan** | `unit_of_measure`, `system_setting`, `config_audit_log` |

**Luồng chính:** (1) Admin thêm/sửa đơn vị tính (code duy nhất). (2) Admin sửa giá trị setting theo key; hệ thống validate kiểu dữ liệu từng key. (3) Mọi thay đổi ghi audit log.
**Ngoại lệ:** xoá đơn vị đang được tham chiếu → chặn.

## UC-P4 — Theo dõi & gửi lại (khách / nhà máy)

| Trường | Nội dung |
|---|---|
| **Actor chính** | Đại lý (gửi khách), Admin (cả hai kênh) |
| **Mục tiêu** | Xem trạng thái gửi và gửi lại các bản ghi `failed` |
| **Bảng liên quan** | `customer_delivery`, `factory_bom_dispatch` |

**Luồng chính:** (1) Người dùng mở báo giá đã phát hành, xem trạng thái từng kênh (pending/sent/failed + thông báo lỗi). (2) Bấm "Gửi lại" cho bản ghi failed → hệ thống tạo lần gửi mới trên cùng bản ghi (cập nhật status). (3) Đại lý luôn có nút **tải PDF** để gửi thủ công ngoài hệ thống [OPEN — O5].
**Business rules:** gửi lại không thay đổi nội dung PDF/BOM đã snapshot; số lần gửi lại không giới hạn nhưng có log thời điểm.

## UC-P5 — Tìm kiếm & sao chép báo giá làm mẫu

| Trường | Nội dung |
|---|---|
| **Actor chính** | Đại lý |
| **Mục tiêu** | Tìm báo giá theo khách/mã/trạng thái/ngày; sao chép một báo giá cũ thành nháp mới |
| **Bảng liên quan** | `quote`, `quote_item` và nhóm `quote_item_*` |

**Luồng chính:** (1) Đại lý tìm theo SĐT khách, mã báo giá, trạng thái, khoảng ngày. (2) Chọn "Sao chép": hệ thống tạo quote `draft` mới, copy hạng mục + cấu hình (không copy ảnh hiện trường, mock-up, số giá đã chốt). (3) Nháp mới đi lại pipeline từ Bước 2; giá tính theo cấu hình hiện hành [D4].
**Business rules:** bản sao là báo giá độc lập, không link ngược về bản gốc (MVP).

## UC-P6 — Xem nhật ký thay đổi cấu hình (audit log)

| Trường | Nội dung |
|---|---|
| **Actor chính** | Admin |
| **Mục tiêu** | Truy vết "ai đổi gì lúc nào" trên dữ liệu giá/công thức khi có khiếu nại hoặc rà soát |
| **Bảng liên quan** | `config_audit_log` |

**Luồng chính:** (1) Admin lọc theo bảng, bản ghi, người đổi, khoảng thời gian. (2) Hệ thống hiển thị danh sách thay đổi kèm diff tóm tắt.
**Business rules:** log chỉ đọc — không sửa/xoá được từ UI.

## UC-P7 — Báo cáo / thống kê báo giá *(sau MVP — ghi nhận phạm vi)*

Số lượng báo giá theo đại lý/sản phẩm/thời gian, tỷ lệ phát hành, tỷ lệ phát hành không kèm mock-up (giám sát D6), thống kê giá bán so với range. Chi tiết đặc tả khi vào phạm vi.

---

# Phụ lục — Các điểm treo còn hiệu lực

| Mã | Vấn đề | Chờ ai |
|---|---|---|
| O1 | Giá trị min–max cụ thể theo sản phẩm | Công ty ban hành |
| O2 | Có cho bán dưới giá nhập không (mặc định: chặn) | Công ty / sếp |
| O3 | Số ngày hiệu lực báo giá (mặc định: chưa in hạn) | Công ty / sếp |
| O4 | Quy trình & kênh gửi BOM nhà máy | Bộ phận sản xuất |
| O5 | Nền tảng (Zalo Mini App / web) & kênh gửi khách | Anh DUong / công ty |
| — | Bảng tính BOM chính thức (đầu vào UC-A8) | Công ty |
| — | Quy tắc làm tròn tiền (đề xuất: về đồng từng dòng) | Kế toán xác nhận |
