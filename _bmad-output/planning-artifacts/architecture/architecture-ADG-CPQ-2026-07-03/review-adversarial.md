---
name: 'Review đối kháng — ARCHITECTURE-SPINE ADG CPQ'
type: adversarial-review
target: ARCHITECTURE-SPINE.md (2026-07-03, draft)
method: 'Dựng cặp unit build độc lập, mỗi unit tuân thủ mọi AD đúng từng chữ, nhưng ghép vào thì vỡ'
reviewer: adversary
date: '2026-07-03'
---

# Review đối kháng — Architecture Spine ADG CPQ

Cách tấn công: với mỗi lỗ hổng, tôi dựng **hai unit A và B**, mỗi unit do một dev/agent build độc lập, **đọc spine và tuân thủ mọi AD theo nghĩa đen**. Nếu tồn tại hai cách build hợp lệ mà ghép lại không chạy → spine thiếu một AD hoặc một Rule chưa đủ chặt.

Kết quả: **12 lỗ hổng** — 4 nghiêm trọng (GAP-1..4), 4 cao (GAP-5..8), 4 trung bình/thấp (GAP-9..12).

---

## GAP-1 (NGHIÊM TRỌNG) — Số nháp lưu ở đâu? AD-5 nghĩa đen mâu thuẫn với chính pipeline

**Cặp unit:** `quote` (phase `pricing` của pipeline, F5) × `pricing` (F4).

**Bối cảnh dữ liệu:** ERD có sẵn cột tiền trên `quote_item` (`cost_material`, `applied_gross_percent`, `import_price`, `selling_price`, `line_total`) và trên `quote` (`subtotal_amount`, `vat_amount`, `total_amount`). Enum `quote_phase.pricing` yêu cầu đại lý **chọn `selling_price` trong [min,max]** ngay khi còn nháp, và D4 (ghi ngay trong ERD Project Note) yêu cầu khi phát hành: *"nếu giá nhập đổi so với nháp, báo đại lý xác nhận lại giá bán"* — tức hệ thống **bắt buộc phải nhớ được giá nhập lúc nháp** để so sánh.

**Hai build đều đúng luật:**
- **Unit A (quote):** đọc AD-5 nghĩa đen — *"Không service nào ghi số tiền vào quote ngoài đường publish"* → tuyệt đối không persist gì lúc nháp; mỗi lần mở lại nháp gọi pricing tính lại on-the-fly. Các cột tiền của draft luôn NULL. Hệ quả: (a) `selling_price` đại lý đã chọn — vốn là **input của người dùng, không phải số dẫn xuất** — cũng không được lưu, mất khi thoát phiên; (b) không còn "giá nhập lúc nháp" nào để publish so sánh theo D4 → tính năng "báo đại lý xác nhận lại" không thể tồn tại.
- **Unit B (pricing):** đọc vế sau của chính AD-5 — *"nháp chỉ giữ số tham khảo"* → hiểu là **được phép** ghi số tham khảo, và vì ERD có sẵn cột nên ghi thẳng `import_price`/`selling_price`/`line_total` vào `quote_item` mỗi lần tính nháp.

**Vỡ khi ghép:** guard "chỉ publish được ghi tiền" của A chặn write của B (hoặc không chặn — tùy ai viết guard trước), và không ai định nghĩa được lúc đọc: số trong cột là "tham khảo" hay "chân lý"? PDF nháp/màn hình review đọc từ đâu? Cùng một cột mang hai semantics tùy status — không có cờ nào phân biệt.

**Đề xuất siết (AD mới):** tách rõ **input vs derived** trên `quote_item`:
- `selling_price` (và quantity, chọn SKU, tick phụ kiện) là **input của đại lý** → được persist ở draft như mọi input khác, bởi duy nhất module `quote`.
- Các cột dẫn xuất (`cost_material`, `applied_gross_percent`, `import_price`, `line_total`, `subtotal/vat/total`) **được phép ghi ở draft như cache tham khảo**, nhưng chỉ qua một hàm duy nhất `QuoteService.refreshDraftPricing()` (gọi pricing thuần túy), kèm cột `priced_at timestamp` để publish so sánh theo D4; đọc cho hiển thị nháp luôn kèm nhãn "tham khảo". Publish transaction là nơi **duy nhất** ghi số cuối cùng và set `submitted`. Sửa câu chữ AD-5 cho khớp.

---

## GAP-2 (NGHIÊM TRỌNG) — AD-6 "tiền là số nguyên VND" đối đầu AD-12 "Prisma sinh từ ERD" (decimal 18,2)

**Cặp unit:** người sinh Prisma schema từ ERD (AD-12) × người viết `pricing`/`formula-engine` (AD-6).

**Hai build đều đúng luật:**
- **Unit A:** tuân AD-12 + convention *"Naming DB đúng theo ERD dbml; không đổi khi sinh Prisma schema"* → sinh mọi cột tiền là `Decimal @db.Decimal(18,2)` đúng như ERD viết (`decimal(18,2)` — cho phép 2 số lẻ!). Repository trả về `Prisma.Decimal` object.
- **Unit B:** tuân AD-6 → toàn bộ pipeline tính toán bằng **integer đồng** (JS `number`), `roundVnd()` trả `number`, zod schema trong `packages/shared` khai `z.number().int()` cho tiền.

**Vỡ khi ghép:** (a) `Decimal("1500000.00") !== 1500000` — so sánh/serialize lệch; (b) một dev khác thấy cột chứa được `.5` đồng thì tin rằng lưu lẻ là hợp lệ; (c) serialize JSON: convention nói `bigint → string`, vậy nếu ai đó "sửa đúng" cột tiền thành `bigint` thì tiền ra JSON là **string**, trong khi zod shared khai `number` → FE parse fail; (d) `applied_gross_percent decimal(7,4)` và `quantity decimal(18,4)` là decimal thật — ranh giới "đâu là Decimal, đâu là int" không được kẻ ở đâu.

**Đề xuất siết:** thêm dòng vào AD-6 hoặc AD-12: *"Khi sinh physical schema, mọi cột tiền trong ERD (`decimal(18,2)`) ánh xạ thành `bigint` (đồng nguyên); ERD logical được cập nhật kèm decision-log entry. Tiền serialize ra JSON là `number` (an toàn < 2^53), KHÔNG theo quy tắc bigint→string của cột id. Percent và quantity giữ `decimal`, vào formula-engine dưới dạng number."* Kèm một type `MoneyVnd = number & brand` trong `packages/shared`.

---

## GAP-3 (NGHIÊM TRỌNG) — "Sau submitted là bất biến" (AD-5) chặn chính các job hậu-phát-hành (AD-7)

**Cặp unit:** người viết guard bất biến (AD-5, module `quote`) × worker render PDF / mockup / delivery (AD-7).

**Bối cảnh:** flow chuẩn là publish → set `submitted` → **sau đó** job render PDF ghi `quote.quote_pdf_url`, job gửi khách ghi `customer_delivery.status`, job nhà máy ghi `factory_bom_dispatch`. Ngoài ra job mockup có thể còn đang chạy khi đại lý bấm phát hành, và khi xong sẽ insert `quote_item_ai_image` + update `quote_item.ai_generation_count`.

**Hai build đều đúng luật:**
- **Unit A:** đọc AD-5 nghĩa đen — *"MỌI mutation lên quote/quote_item có guard chung từ chối khi status = submitted"* → đặt guard ở tầng repository (chỗ chắc nhất, đúng tinh thần "guard chung"). Kết quả: worker PDF không ghi được `quote_pdf_url`, job mockup về muộn bị từ chối → ảnh mất, job retry vô hạn.
- **Unit B (worker):** tuân AD-7 — *"job tự đọc trạng thái mới nhất, cập nhật status vào bảng tương ứng"* → mặc nhiên cho rằng mình được ghi; hoặc tệ hơn, tự mở một đường bypass guard ("system context") mà không AD nào quy định — và đường bypass đó sau này bị dev khác dùng trong request path để "sửa nhanh" quote đã gửi.

**Vỡ khi ghép:** hoặc PDF/ảnh không bao giờ gắn được vào quote đã phát hành, hoặc tồn tại backdoor phá luôn ý nghĩa của AD-5.

**Đề xuất siết:** AD-5 thêm danh sách đóng (closed list) **field hệ thống được ghi sau submitted**: `quote.quote_pdf_url`, `quote_item_ai_image.*` (chỉ khi job enqueue trước submitted — hoặc cấm hẳn, xem GAP-4), `customer_delivery.*`, `factory_bom_dispatch.*`. Freeze áp lên **mọi field nghiệp vụ/giá/cấu hình**; ghi field hệ thống đi qua method riêng có tên tường minh (`attachPublishArtifact()`), không có bypass tổng quát.

---

## GAP-4 (NGHIÊM TRỌNG) — Ai sở hữu `ai_generation_count` và `generation_no`: API lúc enqueue hay worker lúc thành công?

**Cặp unit:** endpoint enqueue của module `mockup` (API) × worker xử lý job mockup.

**Bối cảnh:** ERD [D6]: *"lần gen lỗi kỹ thuật (không ra ảnh) KHÔNG tính lượt"*, tối đa `ai_max_regenerate`, `generation_no` unique theo `(quote_item_id, generation_no)`, và *"thay ảnh hiện trường → reset lượt gen"*.

**Hai build đều đúng luật (AD-7 chỉ nói job idempotent, không nói ai đếm):**
- **Unit A (API-side):** tăng `ai_generation_count` ngay lúc enqueue để enforce limit đồng bộ (chặn double-click). Vi phạm D6 khi job fail kỹ thuật (lượt bị đốt) — nhưng không AD nào cấm.
- **Unit B (worker-side):** chỉ tăng khi ảnh ra thành công (đúng D6), tính `generation_no = count + 1` lúc job chạy. Vỡ ba kiểu: (1) hai job song song (double-click, hoặc BullMQ retry sau timeout nhưng lần chạy đầu thực ra đã insert) cùng tính ra `generation_no` giống nhau → unique violation hoặc — tệ hơn — đếm đôi, "idempotent" bị vi phạm mà không ai định nghĩa idempotency key là gì; (2) limit chỉ được check lúc chạy → user enqueue 5 job khi count=2, cả 5 hợp lệ tại thời điểm enqueue; (3) đại lý **thay ảnh hiện trường** (endpoint thuộc module `quote` — vì `field_photo_url` nằm trên `quote_item`) → ai reset count? ai xử lý các dòng `quote_item_ai_image` cũ? Nếu giữ nguyên dòng cũ mà reset count về 0, lần gen tiếp theo tính `generation_no = 1` → **đâm vào unique index** với ảnh của đợt trước.

**Vỡ khi ghép:** module quote reset count, module mockup tính generation_no từ count, dữ liệu cũ còn nguyên — ghép lại là crash hoặc đếm sai lượt (ảnh hưởng trực tiếp chi phí AI).

**Đề xuất siết (AD mới hoặc phụ lục AD-7):** một chủ sở hữu duy nhất — `MockupService` (được cả API lẫn worker gọi): (a) **reserve lượt tại enqueue** trong transaction (insert dòng `quote_item_ai_image` trạng thái `pending` với `generation_no` cấp phát tại đó — cần thêm cột `status` cho bảng này vào ERD); (b) job fail kỹ thuật → đánh dấu dòng `failed` và **không** đếm vào count (count = COUNT(ảnh succeeded), bỏ cột đếm tay hoặc chỉ coi nó là cache); (c) thay `field_photo_url` là method của MockupService (module quote gọi qua service export, đúng AD-1): hủy job in-flight, đánh dấu ảnh cũ `superseded`, generation_no tiếp tục tăng đơn điệu — **không reset về 1**, chỉ reset quota; idempotency key của job = id dòng `quote_item_ai_image`.

---

## GAP-5 (CAO) — Formula-engine trả kiểu gì: hệ kiểu boolean/select/varchar không được đặc tả

**Cặp unit:** `component` (auto-fill + applicability, F2) × `import-tools`/golden tests (F9) — hoặc `material` (BOM, F3).

**Bối cảnh:** AD-4 chỉ nói chữ ký `(expression, variables) → value | typedError`. Nhưng: `quote_item_param.value` và `quote_item_component_param.value` là **varchar**; param có kiểu `number | text | boolean | date | select`; applicability expression trả *"true/false"*; select *"so sánh theo value của option"*.

**Hai build đều đúng luật:**
- **Unit A (component):** truyền biến vào engine dạng raw varchar (`proj.chieu_cao = "2400"`), engine của A coerce chuỗi số khi gặp toán tử số học; boolean param serialize `"true"`; applicability chấp nhận truthy (số ≠ 0 là true).
- **Unit B (import-tools/golden tests):** parse varchar → typed value trước khi gọi engine (`2400` là number), boolean serialize `"1"` (vì Excel gốc là 1/0), applicability đòi đúng kiểu boolean, kết quả number từ formula param ghi lại varchar theo format có/không có số 0 thừa (`"1.20"` vs `"1.2"`).

**Vỡ khi ghép:** cùng một công thức, golden test pass ở unit B nhưng runtime unit A cho kết quả khác (so sánh `proj.x = "2400"` chuỗi vs số; `if(proj.co_motor, ...)` với `"true"`/`"1"`); giá trị auto-fill ghi vào varchar hai format khác nhau → điều kiện applicability so bằng `=` trượt. Golden tests — chốt chặn go-live của F9 — trở nên vô nghĩa vì **không cùng semantics với production**, đúng cái AD-4 tuyên bố prevent.

**Đề xuất siết:** nâng đặc tả AD-4 thành hệ kiểu tường minh trong `packages/formula-engine`: value ∈ {number, boolean, string}; **cấm coercion ngầm** (so sánh khác kiểu → typedError); ranh giới duy nhất chuyển đổi varchar↔typed là hai hàm `parseParamValue(dataType, varchar)` / `serializeParamValue(value)` nằm trong chính package (canonical form: number không đuôi 0 thừa, boolean `"true"|"false"`, select = option.value nguyên văn); applicability bắt buộc trả boolean, trả kiểu khác = typedError. Golden test harness dùng đúng hai hàm này.

---

## GAP-6 (CAO) — "Recompute toàn bộ" lúc publish vs input đại lý đã override

**Cặp unit:** transaction publish (AD-5, module `quote`) × luồng cấu hình hạng mục (module `component`, cột `is_overridden` của `quote_item_component_param`).

**Hai build đều đúng luật:**
- **Unit A (publish):** đọc AD-5 nghĩa đen — *"recompute toàn bộ → ghi snapshot (khách, sản phẩm, **thông số**, BOM, giá)"* → chạy lại cả auto-fill công thức thông số linh kiện theo cấu hình mới nhất, **đè lên giá trị đại lý đã chỉnh tay** (`is_overridden = true` bị ghi đè), và lọc lại applicability — SKU đại lý đã chọn nay không còn tương thích thì tự thay/loại bỏ.
- **Unit B (component):** coi `is_overridden` là hợp đồng — giá trị đại lý chỉnh là input bất khả xâm phạm; SKU đã chọn là lựa chọn của người dùng, publish chỉ được tính lại **giá** từ các input đó.

**Vỡ khi ghép:** báo giá phát hành khác với cái đại lý nhìn thấy ở bước review (thông số bị đổi ngầm, SKU bị swap) — chính xác loại "số liệu không nhất quán" mà AD-5 sinh ra để chống; hoặc ngược lại nếu A thắng, cột `is_overridden` thành dead data đánh lừa dev sau.

**Đề xuất siết:** AD-5 định nghĩa rõ hai tập: **INPUT giữ nguyên khi publish** = thông số dự án đã nhập, SKU/param linh kiện đã chọn/override, phụ kiện đã tick, quantity, selling_price; **DERIVED tính lại** = BOM lines, cost_material, gross resolve, import_price, VAT, totals. SKU không còn applicability hoặc selling_price lọt ra ngoài range mới, hoặc import_price đổi so với `priced_at` → publish **fail có kiểm soát** với error code (`SKU_INCOMPATIBLE`, `PRICE_OUT_OF_RANGE`, `IMPORT_PRICE_CHANGED`) bắt đại lý xác nhận lại — không tự sửa ngầm.

---

## GAP-7 (CAO) — Không ai được giao sở hữu cây bảng `quote_item_*`: bốn module cùng ghi

**Cặp unit:** `quote` × (`component` | `pricing` | `mockup`) — hai chủ sở hữu cho một entity.

**Bối cảnh:** AD-1 chỉ cấm import **repository** của nhau, không cấm hai module cùng **có repository riêng trỏ vào cùng bảng**. Bước "components" của pipeline thuộc domain component (chọn SKU, auto-fill), nhưng dữ liệu ghi vào `quote_item_component`/`quote_item_component_param`. Tương tự: pricing muốn ghi cache giá (GAP-1), mockup ghi `quote_item_ai_image` + count (GAP-4), material ghi `quote_item_bom_line`.

**Hai build đều đúng luật:**
- **Unit A:** module `component` tạo `QuoteItemComponentRepository` **của riêng nó** (không import repo của quote — đúng chữ AD-1) và ghi thẳng bảng.
- **Unit B:** module `quote` coi mọi bảng `quote_*` là của mình, đặt guard submitted (AD-5), dealer-scope check (AD-8) và validation `step` transition trong service của mình.

**Vỡ khi ghép:** đường ghi của A **đi vòng qua toàn bộ guard của B** — ghi được vào quote đã submitted, không check dealer scope (SQL của A join thẳng theo `quote_item_id` client gửi lên — AD-8 chỉ nói "customer, quote", A hiểu bảng con không thuộc phạm vi), không cập nhật `item_step`. Mỗi AD đều được A "tuân thủ" theo nghĩa đen vì chữ trong AD không phủ bảng con.

**Đề xuất siết (Rule mới cho AD-1):** *"Mỗi bảng có đúng một module chủ sở hữu — chỉ module đó có repository trỏ vào bảng. Toàn bộ TableGroup 8 (quote + mọi quote_item_*) thuộc module `quote`. Module khác muốn ghi phải gọi service export của quote (vd `QuoteItemService.setComponents(...)`), nơi guard submitted + dealer-scope + step áp một chỗ."* Đồng thời sửa AD-8: phạm vi dealer-scope là *"mọi bảng có đường dẫn FK về dealer"*, không liệt kê hai bảng.

---

## GAP-8 (CAO) — AD-3 (DTO whitelist theo vai) × AD-10 (một schema zod chung cho FE+BE): hai cách hiểu một convention

**Cặp unit:** BE viết response DTO × FE (app dealer và app admin) import schema từ `packages/shared`.

**Hai build đều đúng luật:**
- **Unit A (BE):** tuân AD-3 — mỗi endpoint quote có hai DTO: `QuoteItemDealerDto` (không có `cost_material`, `applied_gross_percent`, BOM) và `QuoteItemAdminDto` (đủ). Nhưng AD-10 nói schema khai trong shared và "không định nghĩa type trùng" → A khai **một** `quoteItemSchema` đầy đủ trong shared rồi BE `.parse()` bằng bản dealer là bản `.omit()` tạo tại chỗ trong apps/api (vì để bản omit trong shared thì... cũng là "định nghĩa trùng"?).
- **Unit B (FE dealer):** import `quoteItemSchema` từ shared — schema đầy đủ — và `parse()` response để validate (thói quen zod chuẩn). Field mật khai `not optional` trong schema đầy đủ → **parse fail ngay khi BE làm đúng AD-3**. Hoặc chiều ngược: dev shared khai mọi field mật là `.optional()` cho cả hai vai dùng chung → type dealer "nhìn thấy" field mật, một dev FE viết code hiển thị `cost_material ?? …`, và một dev BE tin rằng "optional nghĩa là được phép gửi khi có".

**Vỡ khi ghép:** hoặc runtime validation fail hàng loạt, hoặc whitelist AD-3 bị xói mòn về "optional = tùy tâm". Ngoài ra e2e test AD-3 assert "không có field mật trong JSON" sẽ pass, nhưng type-level thì cả hai app cùng tin một shape sai.

**Đề xuất siết (Rule cho AD-10):** *"Schema response tách theo vai ngay trong `packages/shared`, đặt tên `<entity><Role>ResponseSchema`; bản dealer build bằng `.pick()` whitelist từ bản admin (không dùng `.omit()` — thêm field mới vào admin không tự lọt sang dealer). BE serialize bằng đúng schema của vai trong token; FE mỗi app chỉ được import schema đúng vai của nó (lint rule chặn app/(dealer) import schema Admin)."*

---

## GAP-9 (TRUNG BÌNH) — AD-9 audit interceptor (HTTP) × import-tools ghi công thức hàng loạt

**Cặp unit:** `audit` (interceptor) × `import-tools` (F9).

- **Unit A:** hiện thực AD-9 đúng chữ "interceptor" = NestJS HTTP interceptor trên các controller admin.
- **Unit B:** import-tools chạy AI-import công thức qua CLI/script/job queue (hợp lý cho bulk hàng trăm formula) — **không đi qua HTTP** → hàng trăm mutation trên bảng `formula` (bảng nhạy cảm nhất hệ thống, quyết định giá) không có dòng `config_audit_log` nào. Cả hai đều tuân AD-9 nghĩa đen ("mutation ... đi qua một interceptor" — B không có mutation qua HTTP nên không vi phạm gì).

**Vỡ khi ghép:** đúng kịch bản AD-9 sinh ra để chống — khiếu nại giá, tra log, thấy công thức đổi ngày X mà không ai đổi.

**Đề xuất siết:** đổi chữ AD-9 từ "interceptor" thành *"audit-service chung, gọi ở tầng repository/service của các bảng nhạy cảm"* — interceptor HTTP chỉ là một caller; mọi đường ghi (HTTP, job, CLI import, seed) đều phải qua audit-service với `changed_by` bắt buộc (import gán user admin thực hiện lệnh import).

---

## GAP-10 (TRUNG BÌNH) — AD-8 dealer-scope guard "từ phiên đăng nhập" × worker không có phiên

**Cặp unit:** guard/decorator dealer-scope (module `auth`/common) × worker jobs (AD-7).

- **Unit A:** hiện thực AD-8 chắc tay — scope filter bắt buộc ở tầng repository, `dealer_id` chỉ lấy từ request context; không có context → throw.
- **Unit B (worker):** job PDF/delivery đọc quote theo id, **không có session** → hoặc crash, hoặc B tự chế `SYSTEM_CONTEXT` bỏ scope — một cửa hậu không AD nào quản, sau này bị gọi từ request path. Tương tự: admin xem quote của đại lý — endpoint admin nhận `dealer_id` từ query có "sai kiến trúc" không? AD-8 chỉ nói "cho vai dealer", nhưng dev A cẩn thận đã chặn ở repository chung cho mọi vai.

**Đề xuất siết:** AD-8 định nghĩa 3 execution context tường minh: `DealerCtx(dealer_id từ token)`, `AdminCtx(cross-dealer, chỉ đọc + endpoint admin)`, `SystemCtx(chỉ khởi tạo được từ job processor, cấm trong controller — enforce bằng lint/DI scope)`. Mọi repository nhận context, không có đường gọi "không context".

---

## GAP-11 (THẤP) — "Làm tròn tại từng dòng": "dòng" là dòng nào?

**Cặp unit:** `material` (BOM, F3) × `pricing` (F4).

AD-6 nói "áp tại từng dòng; tổng = cộng các dòng đã làm tròn". Unit A hiểu "dòng" = `quote_item_bom_line.line_cost` (round từng dòng BOM rồi cộng ra cost_material); Unit B hiểu "dòng" = `quote_item` (giữ full precision suốt chuỗi BOM→cost→import, chỉ round `line_total` cuối). Với `quantity decimal(18,4)` × đơn giá, hai cách lệch nhau vài đồng mỗi hạng mục — đủ để golden test khớp Excel (làm kiểu B) fail khi production làm kiểu A, hoặc PDF không khớp DB. Chuỗi nhân tiếp `import_price = cost × (1+gross/100)` — round trước hay sau nhân gross cũng chưa chốt.

**Đề xuất siết:** liệt kê chuỗi làm tròn chuẩn ngay trong AD-6 (làm chuẩn cho golden tests): round tại `bom_line.line_cost`, `component.line_total`, `accessory.unit_price` → `cost_material` = tổng số đã round → `import_price = roundVnd(cost × (1+g))` → `line_total = roundVnd(selling × qty)` → totals = tổng không round lại. Xác nhận với kế toán rồi khóa trong formula-engine (một hàm `computeItemPricing()` duy nhất).

---

## GAP-12 (THẤP) — `quote.code` "<mã đại lý>-<năm>-<số chạy>": ai cấp số, cấp lúc nào?

**Cặp unit:** hai instance API (hoặc API × chính nó dưới concurrent request).

ERD chỉ ghi "đề xuất" quy tắc; spine không nói gì. Unit A cấp code lúc **tạo nháp** bằng `MAX(số chạy)+1` trong app — hai request đồng thời của cùng đại lý đụng unique; nháp bị xóa để lại lỗ số. Unit B cấp lúc **phát hành** (trong transaction AD-5) — nháp không có code, nhưng UI/danh sách nháp của A đã hiển thị code. Hai cách đều "tuân thủ" vì không có luật.

**Đề xuất siết:** thêm vào AD-5: code cấp **trong transaction publish**, bằng sequence/bảng đếm theo `(dealer, năm)` khóa hàng (SELECT ... FOR UPDATE); nháp hiển thị id tạm. (Hoặc quyết ngược lại — nhưng phải quyết một lần trong spine.)

---

## Tổng kết ma trận

| # | Cặp unit | Loại lệch | Mức | Sửa ở |
|---|---|---|---|---|
| GAP-1 | quote × pricing | hai đường mutate cột tiền draft + hai semantics một cột | Nghiêm trọng | AD-5 viết lại + AD mới input/derived |
| GAP-2 | prisma-gen × pricing | shape dữ liệu tiền: Decimal(18,2) vs int VND vs string JSON | Nghiêm trọng | AD-6 + AD-12 + convention serialize |
| GAP-3 | quote-guard × worker | freeze submitted chặn job hậu phát hành / sinh backdoor | Nghiêm trọng | AD-5 closed-list field hệ thống |
| GAP-4 | mockup-api × mockup-worker (+quote reset ảnh) | hai chủ sở hữu ai_generation_count; generation_no đụng unique khi reset | Nghiêm trọng | AD mới: reserve-at-enqueue, status trên quote_item_ai_image |
| GAP-5 | component × import-tools | hệ kiểu formula-engine (boolean/select/varchar) không đặc tả | Cao | AD-4 nâng đặc tả kiểu + parse/serialize canonical |
| GAP-6 | publish × component | "recompute toàn bộ" đè input đại lý override | Cao | AD-5 định nghĩa INPUT vs DERIVED + fail có kiểm soát |
| GAP-7 | quote × component/pricing/mockup | không ai sở hữu bảng quote_item_* → ghi vòng qua guard | Cao | Rule ownership bảng cho AD-1, mở rộng AD-8 |
| GAP-8 | BE-DTO × FE-shared-schema | một schema zod chung vs whitelist theo vai | Cao | Rule AD-10: schema theo vai bằng .pick() trong shared |
| GAP-9 | audit-interceptor × import-tools | audit HTTP-only, import bulk né audit | Trung bình | AD-9: audit-service tầng service/repo |
| GAP-10 | scope-guard × worker/admin | guard "từ phiên" vs job không phiên → backdoor SystemCtx | Trung bình | AD-8: 3 execution context tường minh |
| GAP-11 | material × pricing | "làm tròn từng dòng" — dòng nào, round trước/sau gross | Thấp | AD-6: chuỗi làm tròn liệt kê đủ |
| GAP-12 | API × API (concurrent) | ai cấp quote.code, lúc nào | Thấp | AD-5: cấp trong publish transaction |

Nhận xét chung: spine mạnh về **hàng rào tĩnh** (dependency, che dữ liệu, một engine) nhưng mỏng ở **quyền sở hữu ghi theo bảng** và **vòng đời dữ liệu theo thời gian** (draft → publish → hậu-publish, job async cắt ngang state machine). Gần như mọi lỗ nghiêm trọng đều xoay quanh AD-5 bị phát biểu tuyệt đối hơn mức hệ thống thật cho phép — nên tách AD-5 thành hai AD: một cho *ownership ghi tiền* (draft cache vs publish truth) và một cho *bất biến sau submitted* (với closed-list ngoại lệ hệ thống).
