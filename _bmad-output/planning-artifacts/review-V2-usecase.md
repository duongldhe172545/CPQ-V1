# Review độ đủ bộ Use Case V2 — ADG CPQ

> Ngày review: 03/07/2026. Người review: reviewer độ đủ (AI).
> Đầu vào: `ADG_CPQ_V2/03_UseCase_ChiTiet.md` (27 UC) + `ADG_CPQ_V2/04_ERD.dbml` (v2).
> Bối cảnh hướng đã chốt 03/07: hệ thống có **bộ máy tính BOM nội bộ hỗ trợ mọi công thức**; **IT/Data Analyst đọc file Excel của công ty rồi nhập công thức vào hệ thống** (không còn nhấn mạnh AI-import).

## VERDICT: **GẦN ĐỦ**

Xương sống (cấu hình → pipeline 4 bước → phát hành → gửi lại) phủ tốt, đối chiếu UC↔ERD gần như khớp hai chiều. Các khoảng trống tập trung ở: (1) vòng đời báo giá ngoài happy-path (huỷ, hết hạn, xoá nháp), (2) bộ công cụ cho người nhập công thức theo hướng mới 03/07 (bảng tra, phiên bản, hồi quy), (3) nhóm xác thực tài khoản.

---

## (a) Đối chiếu UC ↔ ERD hai chiều

### Bảng ERD → UC quản trị/ghi (kết quả: phủ đủ, có 1 lưu ý)

| Nhóm bảng | UC ghi/quản trị | Đánh giá |
|---|---|---|
| `system_setting`, `unit_of_measure` | UC-A7 (bước 3), UC-P3 | ✅ |
| `config_audit_log` | Ghi bởi A3/A5/A6/A7/P2/P3; đọc bởi UC-P6 | ✅ |
| `dealer`, `dealer_product_gross` | UC-P2, UC-A7 | ✅ |
| `user_account` | UC-P1 | ✅ (nhưng thiếu UC tự phục vụ mật khẩu — xem (d)) |
| `customer` | UC-01 (tạo/sửa) | ✅ |
| `category`, `product` | UC-A1 | ✅ |
| `project_param_def`, `project_param_option` | UC-A2 | ✅ |
| `component_type`, `component_param_def`, `component_param_option`, `component_sku`, `component_sku_param_value` | UC-A3 | ✅ |
| `product_component_type`, `component_param_formula`, `component_sku_applicability` | UC-A4 (+A8) | ✅ |
| `accessory`, `product_accessory` | UC-A5 | ✅ |
| `material`, `bom_template`, `bom_template_line` | UC-A6, UC-A8 | ✅ |
| `product_pricing_config` | UC-A7 | ✅ |
| `quote`, `quote_item`, `quote_item_param/_component/_component_param/_accessory/_bom_line/_ai_image` | UC-01…UC-12 | ✅ |
| `customer_delivery`, `factory_bom_dispatch` | UC-11 (tạo), UC-P4 (gửi lại) | ✅ |

**Không có bảng mồ côi.**

### UC → bảng: tham chiếu bảng không tồn tại?

Không có UC nào tham chiếu bảng ma — mọi tên bảng trong mục "Bảng liên quan" của 27 UC đều tồn tại trong DBML. **NHƯNG có chiều ngược đáng chú ý:**

- **[GAP-A1] Golden tests không có thực thể trong ERD.** UC-A8 bước 3–5 nói "bộ golden tests được giữ lại chạy hồi quy mỗi khi sửa công thức" — tức là dữ liệu tồn tại lâu dài (bộ input mẫu + kết quả mong đợi + kết quả chạy), nhưng ERD không có bảng nào (`golden_test_case`, `golden_test_run`…). Nếu chủ đích lưu ngoài DB (file) thì phải ghi rõ; hiện tại UC hứa một khả năng mà mô hình dữ liệu không đỡ.
- **[Minor] Quy tắc sinh `quote.code`** (`<mã đại lý>-<năm>-<số chạy>`) chỉ nằm ở note ERD, UC-01 không nói hệ thống sinh mã lúc nào — nên bổ vào luồng chính UC-01.
- **[Minor] Cơ chế "cảnh báo admin"** ở UC-06 E-2/E-3, UC-08 E-1/E-2 không có bảng notification/alert nào và không có UC "admin xem cảnh báo cấu hình". Cảnh báo bắn đi đâu?

## (b) Vòng đời đối tượng chính

| Thực thể | Tạo | Sửa | Vô hiệu | Duyệt | Nhận xét |
|---|---|---|---|---|---|
| Sản phẩm | UC-A1 | UC-A1 | UC-A1 4a | UC-A8 (go-live) | ✅ đủ |
| Danh mục | UC-A1 | UC-A1 | E-1 (chặn xoá có con) | — | ✅ đủ |
| Thông số dự án | UC-A2 | E-2 (đổi/xoá có cảnh báo) | — | — | ✅ chấp nhận được |
| Loại linh kiện / SKU | UC-A3 | UC-A3 | **thiếu** | — | ⚠️ `component_sku.is_active`, `component_type.is_active` có trong ERD nhưng UC-A3 không có luồng vô hiệu hoá SKU; hệ quả với nháp đang chọn SKU đó chưa đặc tả |
| Phụ kiện | UC-A5 | UC-A5 | **thiếu** | — | ⚠️ `accessory.is_active` có trong ERD, UC-A5 chỉ có "gỡ gán", không có vô hiệu phụ kiện |
| Vật tư | UC-A6 | UC-A6 | **thiếu** | — | ⚠️ `material.is_active` có trong ERD; vô hiệu vật tư đang nằm trong BOM template active thì sao? Chưa đặc tả |
| BOM template | UC-A6 | UC-A6 | is_active (F4) | UC-A8 | ✅ tốt nhất nhóm |
| Công thức (auto-fill, tương thích, BOM line) | UC-A4/A6/A8 | UC-A4/A6 | — | UC-A8 lần đầu | ⚠️ **sửa sau go-live** chỉ là 1 câu business rule trong A8, không có UC riêng (xem (c)) |
| Đại lý | UC-P2 | UC-P2 | UC-P2 (4) | — | ✅ đủ |
| Khách hàng | UC-01 | UC-01 4a | **không có** | — | ⚠️ chấp nhận được cho MVP (kho khách theo đại lý), nên ghi nhận chủ đích |
| Tài khoản | UC-P1 | UC-P1 (khoá/mở) | UC-P1 | — | ⚠️ thiếu sửa hồ sơ & mật khẩu tự phục vụ — xem (d) |
| Báo giá | UC-01 | UC-12 | **thiếu huỷ + xoá nháp** | UC-11 (phát hành) | ❌ gap lớn nhất — xem (d) |

**Mẫu số chung:** các thực thể cấu hình đều mạnh ở "tạo/sửa" nhưng nhánh **vô hiệu hoá khi đang bị tham chiếu** (SKU trong nháp, vật tư trong BOM active, phụ kiện đã gán) chỉ được A1 làm mẫu tốt; A3/A5/A6 chưa có luồng tương ứng dù ERD đã có cột `is_active`.

## (c) Vai IT/Data Analyst nhập công thức (theo hướng chốt 03/07)

Đây là nhóm lệch nhiều nhất so với hướng mới:

1. **[GAP-C1] UC-A8 viết theo kịch bản AI-import, không phải người nhập tay.** Actor là "Admin (phối hợp đội phát triển/AI)", bước 2 là "Logic được dịch (AI-assisted)". Theo hướng 03/07, actor thực là **IT/Data Analyst đọc Excel và nhập công thức thủ công qua UI** — UC-A8 cần viết lại actor + luồng chính (nhập từng công thức, lưu nháp dở dang giữa chừng, đánh dấu tiến độ "đã nhập x/y dòng BOM").
2. **[GAP-C2] Không có vai/quyền cho IT/Data Analyst.** Enum `user_role` chỉ có `admin | dealer` [D9]. Nếu Data Analyst dùng chung vai admin thì được toàn quyền giá/gross/đại lý — nên hoặc ghi rõ chủ đích "DA dùng vai admin", hoặc cân nhắc quyền hẹp (chỉ công thức/BOM). Tối thiểu phải ghi nhận quyết định.
3. **[GAP-C3] Thiếu UC "nhập bảng tra" (lookup table).** File Excel của công ty gần như chắc chắn có VLOOKUP/bảng tra (khoảng kích thước → hệ số, độ dày → định mức). Ngôn ngữ công thức [D10] chỉ có `if/round/ceil/floor/min/max` — DA sẽ phải dịch mỗi bảng tra 20 dòng thành 20 tầng `if()` lồng nhau: khó nhập, khó soát, dễ sai. Thiếu: bảng `lookup_table`/`lookup_row` + hàm `lookup(...)` + UC quản trị bảng tra. **Đây là gap ảnh hưởng trực tiếp tuyên bố "hỗ trợ mọi công thức".**
4. **[GAP-C4] Thiếu UC quản lý phiên bản công thức & sửa sau go-live.** `bom_template` có bán-phiên-bản qua is_active, nhưng `component_param_formula` và `component_sku_applicability` sửa là đè (chỉ còn dấu vết trong audit log, không rollback được). Business rule A8 nói "sửa công thức = sửa trên hệ thống (có audit log)" và "golden tests chạy hồi quy mỗi khi sửa" — nhưng không UC nào đặc tả luồng đó: ai chạy hồi quy, chạy lúc nào, fail thì chặn lưu hay chỉ cảnh báo?
5. **[GAP-C5] Thiếu UC so sánh kết quả 2 phiên bản** (chạy cùng bộ thông số trên template cũ vs mới, diff từng dòng vật tư) — công cụ nghiệm thu tự nhiên khi DA sửa công thức; hiện chỉ có "kiểm thử 1 bộ giá trị mẫu" (A4 bước 4, A6 bước 4).
6. **[Điểm tốt]** UC-A4/A6 đã có sandbox kiểm thử với bộ thông số mẫu và validate biến/cú pháp — nền tảng tốt, các gap trên là phần mở rộng chứ không phải làm lại.

## (d) Ngoại lệ & vận hành

1. **[GAP-D1] Huỷ báo giá — enum `quote_status.cancelled` tồn tại nhưng KHÔNG có UC nào.** Ai được huỷ (đại lý? admin?), huỷ được ở trạng thái nào (draft/submitted?), huỷ báo giá đã gửi BOM về nhà máy thì có thu hồi/thông báo nhà máy không? Đây là khoảng trống nghiệp vụ thật, không chỉ thiếu giấy tờ.
2. **[GAP-D2] Xoá nháp báo giá:** UC-02 có xoá hạng mục nhưng không UC nào cho xoá/bỏ cả bản nháp — đại lý sẽ tồn đọng nháp rác. (Có thể gộp vào UC huỷ.)
3. **[GAP-D3] Báo giá hết hạn hiệu lực:** `valid_until` có trong ERD và được in vào PDF (UC-11), nhưng không UC nào định nghĩa chuyện gì xảy ra sau hạn — hệ thống có đánh dấu/hiển thị "hết hạn" không, đại lý gia hạn bằng cách nào (chấp nhận trả lời "sao chép qua UC-P5" nhưng phải ghi tường minh).
4. **[GAP-D4] Nhóm xác thực:** không có UC đăng nhập, đăng xuất, đổi mật khẩu, quên mật khẩu. UC-P1 chỉ có "gửi link đặt mật khẩu ban đầu". Với hệ thống chứa dữ liệu giá mật, ít nhất cần UC đổi/quên mật khẩu (và ERD chưa có chỗ cho reset token nếu làm theo hướng token).
5. **Xem chi tiết báo giá đã gửi:** phủ được một phần qua UC-P4 (trạng thái gửi + tải PDF) và UC-P5 (tìm kiếm) — chấp nhận được, nhưng chưa có UC "xem lại chi tiết snapshot báo giá submitted" tường minh (màn hình đại lý xem lại hạng mục/giá đã đóng băng). Mức minor.
6. **[Minor] Admin xem danh sách báo giá toàn hệ thống:** UC-P4 cho admin theo dõi kênh gửi, UC-P7 (sau MVP) mới có thống kê — khoảng giữa (admin tra một báo giá cụ thể khi đại lý gọi điện hỏi) chưa rõ thuộc UC nào.

## (e) Soi tiền/hậu điều kiện vs luồng chính (chỉ lỗi thật)

1. **UC-04:** Hậu điều kiện thành công ghi "`field_photo_url` được set; `step = params`", nhưng E-3 cho phép **bỏ qua ảnh và vẫn đi tiếp các bước sau**. Vậy khi bỏ qua, `step` có chuyển `params` không? Nếu có thì hậu điều kiện "field_photo_url được set" không phải điều kiện để chuyển bước — cần tách rõ 2 kết cục (có ảnh / tạm bỏ qua) để dev không chặn nhầm.
2. **UC-02:** Hậu điều kiện thành công "Báo giá có ≥ 1 hạng mục" nhưng luồng 1b cho xoá hạng mục — xoá đến 0 hạng mục vẫn là kết cục hợp lệ của UC này (chỉ bị chặn ở UC-11). Hậu điều kiện nên hạ xuống "danh sách hạng mục phản ánh đúng thao tác" và để "≥1" là điều kiện phát hành.
3. **UC-10:** Tiền điều kiện "Đã chốt giá (UC-09)" ổn, nhưng hậu điều kiện thất bại "xử lý theo quy tắc D6 khi phát hành" — hợp lệ vì UC-11 bước 2 bắt `no_mockup_reason`. Không lỗi, chỉ ghi nhận là ràng buộc liên-UC đúng.
4. Các UC còn lại: chuỗi `step` (product→photo→params→components→accessories→done) và `current_phase` khớp nhất quán giữa UC-03→07, UC-08/09; hậu điều kiện thất bại UC-11 (draft nếu fail trước snapshot / submitted + bản ghi failed) khớp E-1/E-2. **Không phát hiện mâu thuẫn thật nào khác.**

## Tổng hợp khoảng trống theo mức quan trọng

| # | Mức | Khoảng trống | Đề xuất |
|---|---|---|---|
| 1 | Cao | Huỷ báo giá (enum `cancelled` không có UC) + xoá nháp | Thêm UC-13 "Huỷ / xoá báo giá" |
| 2 | Cao | Bảng tra (lookup) cho công thức — Excel có VLOOKUP, ngôn ngữ công thức không có | Thêm bảng `lookup_table` + hàm `lookup()` + UC-A9 |
| 3 | Cao | UC-A8 lệch hướng 03/07 (viết cho AI-import, không phải DA nhập tay); vai DA không tồn tại | Viết lại UC-A8; chốt DA dùng vai admin hay vai riêng |
| 4 | Cao | Golden tests không có bảng ERD; không UC sửa công thức sau go-live + chạy hồi quy | Thêm bảng golden test + UC "Sửa công thức & hồi quy" |
| 5 | Trung | Đổi/quên mật khẩu, đăng nhập/đăng xuất không có UC | Thêm UC-P8 xác thực tự phục vụ |
| 6 | Trung | Báo giá hết hạn `valid_until` — không UC định nghĩa hành vi sau hạn | Bổ rule vào UC-P4/P5 hoặc UC riêng |
| 7 | Trung | Vô hiệu hoá SKU linh kiện / vật tư / phụ kiện đang bị tham chiếu (ERD có `is_active`, UC không có luồng) | Bổ luồng phụ vào A3/A5/A6 |
| 8 | Thấp | So sánh kết quả 2 phiên bản công thức; kênh "cảnh báo admin" (UC-06/08) không có UC/bảng; sinh `quote.code`; admin tra cứu báo giá | Bổ sung dần theo ưu tiên |
