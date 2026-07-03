# ADG CPQ — Diễn giải nghiệp vụ & triết lý thiết kế

Tài liệu này đi kèm ba file thiết kế:

- `ADG_CPQ_ERD.dbml` — sơ đồ dữ liệu (import vào dbdiagram.io).
- `ADG_CPQ_Swimlane.drawio` — luồng nghiệp vụ 8 phase (mở bằng draw.io).
- `ADG_CPQ_UseCase_ChiTiet.md` — **đặc tả use case chi tiết** (bảng metadata, luồng chính, luồng phụ/ngoại lệ, hậu điều kiện).

Tài liệu này giải thích *vì sao* ERD được dựng như vậy để trợ lý thiết kế nắm được ý đồ kiến trúc trước khi đọc đặc tả use case. Bộ use case đầy đủ nằm ở file `ADG_CPQ_UseCase_ChiTiet.md`.

---

## Phần A — Triết lý thiết kế & diễn giải nghiệp vụ

### A0. Nguyên tắc xuyên suốt: cấu hình bằng dữ liệu, không đụng schema

Yêu cầu cốt lõi của hệ thống là một đại lý bất kỳ có thể được cấp thêm sản phẩm mới, với bộ thông số và công thức hoàn toàn khác, mà đội phát triển không phải sửa cấu trúc cơ sở dữ liệu hay viết thêm code. Toàn bộ ERD được tổ chức quanh nguyên tắc này qua ba cơ chế:

Thứ nhất là tách *định nghĩa* khỏi *giá trị*. Mỗi sản phẩm không có các cột "chiều dài", "chiều cao" cứng trong bảng, mà khai báo động các thông số của nó trong `project_param_def`; giá trị đại lý nhập vào một báo giá cụ thể nằm ở `quote_project_param`. Thêm một thông số mới cho sản phẩm chỉ là thêm một dòng cấu hình. Cùng mô hình đó áp dụng cho thông số linh kiện (`component_param_def` / `quote_component_param`).

Thứ hai là công thức lưu dưới dạng biểu thức chuỗi do admin nhập, không phải logic biên dịch trong code. `component_param_formula.formula_expression` và `bom_template_line.quantity_formula` là các biểu thức tham chiếu biến qua `code` ổn định (quy ước `proj.<code>` cho thông số dự án, `comp.<code>` cho thông số mặc định linh kiện). Khi nghiệp vụ đổi cách tính, admin sửa biểu thức chứ không cần release phần mềm.

Thứ ba là snapshot khi chốt. Mọi giá trị đã nhập hoặc đã tính của một báo giá (thông số, linh kiện, phụ kiện, dòng BOM, các con số giá) được sao lưu ngay trong nhóm bảng `quote_*`. Nhờ vậy, admin có thể thoải mái đổi cấu hình, đổi công thức, đổi đơn giá vật tư về sau mà các báo giá cũ vẫn giữ nguyên số liệu tại thời điểm phát hành.

### A1. Catalog & thông số dự án

`category` là cây danh mục nhiều cấp (tự tham chiếu `parent_id`). `product` mang thông tin nhận diện tối thiểu mà đại lý nhìn thấy ở Phase 2: tên, SKU, thumbnail. Mọi thứ đặc thù của một sản phẩm — bộ thông số, danh sách loại linh kiện, danh sách phụ kiện, BOM, cấu hình giá — đều gắn ngoài vào `product` qua các bảng liên kết, nên khai thêm sản phẩm không làm phình bảng `product`.

Điểm nghiệp vụ cần lưu ý: **thông số dự án là riêng theo từng sản phẩm** (`project_param_def.product_id`). Mỗi thông số khai kiểu dữ liệu (`data_type`), cách nhập (`input_type`: freeform hay dropdown), đơn vị tính, bắt buộc hay không. Với thông số dạng dropdown, danh sách chọn nằm ở `project_param_option`.

### A2. Linh kiện & công thức auto-fill

Linh kiện được mô hình hoá ở ba tầng, **dùng chung toàn hệ thống**: `component_type` (loại linh kiện, ví dụ "bộ tời"), `component_param_def` (bộ thông số của loại đó, ví dụ "công suất"), và `component_sku` (mã linh kiện cụ thể với đơn giá, đơn vị). Giá trị thông số mặc định của từng SKU nằm ở `component_sku_param_value` — đây chính là nguồn biến `comp.*` cho công thức.

Chỗ tinh tế nhất của thiết kế nằm ở việc **công thức auto-fill được đặt tại bảng gán `product_component_type`** thông qua `component_param_formula`, chứ không đặt ở `component_type`. Lý do: loại linh kiện dùng chung, nhưng công thức lại phải tham chiếu thông số dự án vốn riêng theo sản phẩm. Vậy cùng một "bộ tời" khi gắn vào sản phẩm "cửa cuốn A" và sản phẩm "cửa cuốn B" có thể có công thức tính công suất khác nhau. Đại lý vẫn tự chọn SKU (hệ thống không gợi ý SKU) và được chỉnh lại giá trị auto-fill; cờ `quote_component_param.is_overridden` ghi nhận đại lý đã sửa khác giá trị máy tính. Việc kiểm tra tính hợp lệ/tương thích của lựa chọn là hạng mục sau MVP.

### A3. Phụ kiện

`accessory` là danh mục dùng chung (tên, thumbnail, đơn giá), gán vào sản phẩm qua `product_accessory`. Ở báo giá, phụ kiện chỉ là thao tác tick chọn và **số lượng luôn bằng 1**, nên `quote_accessory` không có cột số lượng — đơn giá cộng thẳng một lần vào giá vốn.

### A4. Vật tư & BOM

`material` là danh mục vật tư do admin quản lý, mỗi vật tư giữ **một đơn giá hiện hành** (không lưu lịch sử giá theo yêu cầu MVP). `bom_template` gắn theo từng sản phẩm; mỗi dòng `bom_template_line` chỉ định một vật tư kèm `quantity_formula` — công thức tính số lượng theo thông số dự án. Linh kiện và phụ kiện không tác động tới BOM, nên biến trong công thức BOM chỉ gồm `proj.*`.

### A5. Định giá & che giấu dữ liệu nhạy cảm

Toàn bộ công thức giá được gom mô tả trong `Project Note` của file DBML. Diễn giải nghiệp vụ:

Giá vốn nguyên vật liệu (`cost_material`) là tổng của ba khối: tiền vật tư từ BOM (Σ số lượng × đơn giá vật tư), tiền linh kiện (Σ đơn giá × số lượng), và tiền phụ kiện (Σ đơn giá, mỗi cái một lần). Trên nền đó cộng lợi nhuận gộp để ra **giá nhập**: `giá nhập = cost_material × (1 + gross%/100)`, trong đó `gross% = global_gross% (theo sản phẩm) + gross% của đại lý`. Riêng phần gross của đại lý có hai mức — một mức chung ở `dealer.default_gross_percent` và một mức theo từng sản phẩm ở `dealer_product_gross`; **mức theo sản phẩm nếu tồn tại sẽ thay thế mức chung** (không cộng dồn).

Từ giá nhập, hệ thống mở ra một khoảng giá bán ("range bình ổn") cấu hình **theo từng sản phẩm** ở `product_pricing_config` gồm biên độ ±% và/hoặc chặn min–max (đều không bắt buộc). Đại lý chọn một mức giá bán trong khoảng này. VAT áp dụng **một mức chung toàn hệ thống** (lưu ở `system_setting`), tính trên giá bán. Tổng cuối cùng gửi khách là giá bán cộng VAT.

Ranh giới hiển thị rất quan trọng về mặt nghiệp vụ: `cost_material`, đơn giá vật tư, các dòng BOM và toàn bộ % gross là **thông tin mật, ẩn hoàn toàn với đại lý**. Đại lý chỉ nhìn thấy giá nhập, range cho phép, giá bán mình chọn và VAT. Vì thế các trường nhạy cảm được đánh dấu ẩn ngay trong `quote` và tách khỏi phần dữ liệu đại lý đọc được.

### A6. Báo giá & vòng đời

Một `quote` gắn đúng **một sản phẩm**, một khách hàng (chọn lại từ kho khách của đại lý theo SĐT — ràng buộc `unique(dealer_id, phone)` ở `customer`), và một ảnh hiện trường bắt buộc (tối đa một ảnh, ≤ 8MB, ràng buộc thực thi ở tầng ứng dụng). Báo giá hỗ trợ **lưu nháp và quay lại sửa các phase trước**: trường `status` (`draft`/`submitted`) cùng `current_phase` cho phép khôi phục đúng bước đang dở. Sau khi phát hành, trạng thái chuyển `submitted` và quy trình kết thúc — hệ thống không theo dõi phản hồi của khách sau khi gửi.

### A7. Ảnh AI, gửi khách, gửi nhà máy

Ở Phase 8, dịch vụ AI ghép ảnh mock-up từ ảnh hiện trường và thumbnail sản phẩm; mỗi lần sinh lưu một dòng `quote_ai_image` với `generation_no` (1–3) và `is_selected`. Đại lý duyệt hoặc yêu cầu tạo lại **tối đa 3 lần**; ảnh được chọn sẽ đính vào PDF. Sau khi phát hành, PDF gửi cho khách cuối qua Zalo OA (`customer_delivery`) và phiếu BOM gửi về nhà máy qua API hoặc email (`factory_bom_dispatch`) — chi tiết cổng nhà máy do trợ lý hoàn thiện.

---

## Đặc tả use case

Bộ use case cốt lõi (7 UC cấu hình cho Admin + 11 UC pipeline cho Đại lý) và danh sách use case phụ trợ được đặc tả chi tiết ở file **`ADG_CPQ_UseCase_ChiTiet.md`**. Mỗi UC gồm bảng metadata (actor chính/phụ, mục tiêu, tiền điều kiện, trigger, hậu điều kiện thành công/thất bại, bảng liên quan), luồng chính đánh số, luồng phụ/ngoại lệ ký hiệu theo bước (a/b/c, E-) và business rules.
