# PRD Quality Review — ADG CPQ: Hệ thống báo giá tự động cho đại lý Austdoor

> **Reviewer:** độc lập, đối chiếu ngược với `docs/ADG_CPQ_UseCase_ChiTiet.md` (v2) và `_bmad-output/planning-artifacts/ADG-CPQ-decision-log.md`.
> **Ngày review:** 03/07/2026. **Đối tượng:** `prd.md` + `addendum.md` (cùng thư mục).

## Overall verdict — **ĐẠT CÓ ĐIỀU KIỆN**

PRD này có xương sống tốt hiếm thấy: mọi quyết định lớn truy vết được về mã D/F/O, câu hỏi mở có chủ và có điều kiện chặn, counter-metrics thật, phần kỹ thuật tách đúng sang addendum. Nhưng đối chiếu ngược phát hiện **một mảng chức năng bị bỏ sót hoàn toàn (quản trị phụ kiện — UC-A5)** và **một mâu thuẫn giữa mục tiêu G4 "100% trong range — ràng buộc cứng" với chính tài liệu nguồn (range được phép để trống)**. Hai lỗi này phải sửa trước khi chốt; các lỗi medium (mất vế "khoá Excel" của D10, rơi rủi ro O7, thiếu Assumptions Index và Glossary) nên sửa cùng đợt vì chi phí thấp.

---

## 1. Decision-readiness — **strong**

Đây là điểm mạnh nhất của PRD. Các quyết định được nêu là quyết định, kèm mã truy vết: FR-012 "*hệ thống lọc, đại lý chọn* [D2]", FR-032 "chỉ dùng min–max [D3]", §4 "Ngoài phạm vi vĩnh viễn" nêu thẳng những gì bị loại. Bảng Câu hỏi mở (§10) có đủ ba cột: câu hỏi — ai chốt — chặn gì; phân biệt rõ "chặn build" và "chặn go-live" (bảng tính công thức). Trade-off của D10 (bỏ Excel-as-engine) được ghi lại trong addendum kèm lý do loại. Người ra quyết định đọc PRD này biết mình đang cam kết cái gì và còn nợ cái gì.

### Findings

- **low** Gate toàn quốc lệch mốc thời gian với G2 (§9 vs §2) — Gate "Toàn quốc" yêu cầu pilot đạt "BOM sai ≤ 1%" sau 4–6 tuần, trong khi G2 đặt target ≤ 1% "sau 6 tháng". Hai con số giống nhau nhưng hai cửa sổ đo khác nhau; pilot 15 đại lý có thể không đủ mẫu để kết luận ≤ 1% có ý nghĩa thống kê. *Fix:* ghi rõ gate pilot đo trên tập phiếu BOM của pilot (và cỡ mẫu tối thiểu), G2 là target vận hành dài hạn.

## 2. Substance over theater — **strong**

Không có persona thừa: 3 hành trình (đại lý, admin, khách nhận PDF) đều gánh quyết định thật — UJ-1 chứng minh lọc tương thích, sao chép hạng mục, chọn giá trong range, lối thoát mock-up; UJ-2 chứng minh triết lý config-driven không cần dev. Counter-metrics (§2) là hàng thật: "<10% phát hành không kèm mock-up" giám sát đúng lối thoát D6, "số sự cố công thức sai = 0" gắn thẳng FR-090. NFR phần lớn có ngưỡng sản phẩm-cụ-thể (≤ 3 giây tính giá, ≤ 60 giây mock-up, 5.000 quote/tháng) chứ không phải boilerplate.

### Findings

- **medium** NFR-03 "dùng mượt trên điện thoại" là tính từ, không phải ngưỡng (§7 NFR-03) — "mượt" và "thao tác một tay" không kiểm chứng được; đây là NFR gánh bối cảnh sử dụng quan trọng nhất (đại lý đứng tại công trình). *Fix:* đặt tiêu chí đo được, ví dụ: pipeline hoàn chỉnh dùng được ở viewport 360px không cuộn ngang; các nút hành động chính nằm trong vùng ngón cái; chụp ảnh gọi camera trực tiếp; mỗi bước ≤ N thao tác chạm.
- **low** FR-042 "độ phân giải tối thiểu" không có giá trị và không nói là cấu hình được (§6 F5) — engineer không biết ngưỡng nào để validate, tester không biết ca nào fail. *Fix:* nêu giá trị mặc định (VD ≥ 1280×720) hoặc ghi rõ là `system_setting` cấu hình được, thêm vào danh sách setting của FR-072.

## 3. Strategic coherence — **strong**

PRD có luận đề rõ: khép kín khảo sát → báo giá → sản xuất, công ty kiểm soát giá tập trung, mock-up AI phục vụ trực tiếp mục tiêu số 1 (chốt đơn). Thứ tự ưu tiên mục tiêu (chốt đơn > BOM đúng > tốc độ > kiểm soát giá) khớp D13 và được dùng thật: D12 đưa mock-up vào MVP *vì* gắn G1 — đó là prioritization theo luận đề, không phải theo "cái gì dễ làm trước". Scope MVP thuộc loại problem-solving, logic cắt Phase 2 (khuyến mãi, báo cáo nâng cao, ZMA) nhất quán với logic đó.

### Findings

- **low** Vế "chừa hook, không hard-code giá" của D5 chỉ còn ngầm trong tiêu đề §4 "Phase 2+, thiết kế phải chừa chỗ" — decision log D5 yêu cầu tường minh "thiết kế phải chừa hook (không hard-code giá)" cho chính sách khuyến mãi tương lai, nhưng cả PRD lẫn addendum (mục "Cho chặng Kiến trúc") đều không nhắc lại ràng buộc này ở chỗ kiến trúc sư sẽ đọc. *Fix:* thêm một dòng vào addendum § Kiến trúc: "D5 — pipeline giá phải có điểm nối cho chính sách khuyến mãi công ty ở Phase 2; không hard-code công thức giá bán cuối."

## 4. Done-ness clarity — **adequate**

Phần lớn FR có hệ quả kiểm chứng được: FR-023 "không bao giờ ra giá sai âm thầm", FR-032 "validate min ≤ max", FR-060 "giá nhập lệch → buộc xác nhận lại", FR-090 "khớp 100% mới go-live". Chiến lược "PRD không lặp lại luồng từng bước, cột UC trỏ về đặc tả hành vi" (§6 header) là hợp lệ *vì* tài liệu UC v2 thực sự có luồng chính/phụ/ngoại lệ đầy đủ — acceptance criteria sống ở đó. Nhưng chính vì lệ thuộc cơ chế cross-ref này, chỗ nào UC không được FR nào trỏ tới thì chức năng đó rơi khỏi phạm vi cam kết — và đã có chỗ rơi thật (xem finding critical/high bên dưới).

### Findings

- **high** [BỎ SÓT NGUỒN] Không có FR nào cho quản trị phụ kiện — UC-A5 mất tích (§6 F1–F2) — UC-A5 quy định admin tạo phụ kiện (tên, thumbnail, đơn giá, audit log khi đổi giá) và gán vào sản phẩm. PRD chỉ có phía tiêu thụ: FR-045 (đại lý tick chọn), FR-030 (phụ kiện cộng vào cost_material). Không FR nào tạo ra dữ liệu đó → đội build có thể bỏ qua toàn bộ màn hình quản trị phụ kiện mà vẫn "đủ FR", và giá phụ kiện đổi không được cam kết ghi audit (thủng FR-073 vốn chỉ nói "giá/công thức/cấu hình"). *Fix:* thêm FR (VD FR-014 trong F2 hoặc F1): "Quản lý phụ kiện dùng chung (tên, thumbnail, đơn giá — đổi giá ghi audit log) và gán phụ kiện khả dụng theo sản phẩm | UC-A5".
- **medium** FR-060 không liệt kê đủ điều kiện chặn phát hành đã có trong nguồn — UC-02 quy định "≥ 1 hạng mục mới phát hành", UC-11 bước 2 + UC-02 E-1 quy định chặn khi có hạng mục dở bước, UC-03 E-1 chặn khi sản phẩm thiếu BOM active/giá. FR-060/FR-042/FR-051 mỗi cái giữ một mảnh; không chỗ nào gom "điều kiện phát hành" thành một danh sách kiểm chứng được. *Fix:* bổ sung vào FR-060: "Điều kiện phát hành: ≥ 1 hạng mục; mọi hạng mục hoàn tất bước; đủ ảnh hiện trường; mock-up theo quy tắc FR-051 — vi phạm thì chặn và chỉ rõ hạng mục/bước."
- **low** Hai lớp lỗi công thức bị gộp làm một — FR-023 quy định lỗi công thức → "dừng tính", nhưng nguồn UC-06 E-3 cho phép công thức *auto-fill* lỗi thì để trống cho đại lý nhập tay (không dừng). PRD như đang viết sẽ khiến engineer chặn cả luồng auto-fill. *Fix:* giới hạn FR-023 vào công thức BOM/giá; thêm chú thích ở FR-044: auto-fill lỗi → cho nhập tay + cảnh báo admin (UC-06 E-3).

## 5. Scope honesty — **adequate**

Ba tầng phạm vi (§4: Trong MVP / Phase 2+ / Ngoài vĩnh viễn) là mẫu mực; các ghi chú phạm vi trong decision log mục D (không theo dõi phản hồi khách, không lịch sử giá vật tư) đều được chuyển thành dòng Phase 2 tường minh — không có de-scope âm thầm ở đây. `[ASSUMPTION]` được gắn đúng chỗ suy diễn (target G2/G3, quy mô NFR-05, pilot). Nhưng mật độ open-items khá cao cho một PRD chuẩn bị green-light (≈10 ASSUMPTION + 7 dòng câu hỏi mở) mà **không có Assumptions Index** — không ai được giao xác nhận từng giả định trước mốc nào.

### Findings

- **high** [MÂU THUẪN NGUỒN] G4 "100% báo giá trong range công ty — ràng buộc cứng" xung đột với hành vi range-trống của nguồn (§2 G4 vs UC-A7 1a / UC-09 1a) — Nguồn cho phép sản phẩm **không cấu hình min–max** ("để trống min–max; đại lý nhập giá bán tự do, chỉ áp sàn giá nhập"), và FR-033 còn có cờ `allow_below_import` nới cả sàn. PRD không nhắc khả năng range trống ở FR-032/FR-046, trong khi G4 tuyên bố 100% là "ràng buộc cứng của hệ thống". Nếu range trống hợp lệ thì G4 không đo được (trong range nào?) hoặc mặc nhiên fail; nếu G4 đúng là ràng buộc cứng thì phải cấm range trống — PRD đang im lặng đứng giữa. *Fix:* chọn một: (a) FR-032 bổ sung "range được phép trống → giá tự do có sàn giá nhập; các báo giá này loại khỏi mẫu số G4" và hạ G4 thành "100% báo giá *của sản phẩm có range* nằm trong range"; hoặc (b) quy định range là điều kiện go-live sản phẩm (thêm vào UC-03 E-1 / FR-060) để G4 giữ nguyên nghĩa cứng. Đề xuất (b) — nó khớp tinh thần "kiểm soát giá" của luận đề.
- **medium** [BỎ SÓT NGUỒN] Rủi ro O7 (màu ↔ ảnh mock-up) biến mất khỏi PRD — Decision log O7 (đã đóng) ghi nhận rủi ro tường minh: "nếu màu chọn lúc báo giá thì ảnh dùng cho mock-up AI phải theo màu đã chọn — một `thumbnail_url` không đủ", UC-A1 business rules cũng nhắc lại. PRD đưa mock-up vào MVP (F6, gắn G1) và UJ-2 có ví dụ "màu là dropdown", nhưng FR-050 chỉ nói "ảnh sản phẩm" — rủi ro khách nhận mock-up sai màu cửa họ đã chọn không xuất hiện ở đâu. *Fix:* thêm một dòng vào §10 Câu hỏi mở ("Ảnh sản phẩm theo biến thể màu cho mock-up — chốt khi có catalog thật — Phòng sản phẩm") hoặc ràng buộc trong FR-050: "ảnh sản phẩm đưa vào AI phải khớp giá trị thông số ảnh hưởng ngoại hình (VD màu) nếu catalog có ảnh biến thể".
- **medium** [BỎ SÓT NGUỒN] Mất vế "sau go-live, hệ thống là nguồn chân lý duy nhất — khoá sửa Excel" của D10 (§6 F9) — Đây là business rule của UC-A8 và là nửa sau của quyết định D10; thiếu nó thì golden tests chỉ bảo vệ lúc import, còn hai nguồn chân lý (Excel sống + hệ thống) vẫn có thể lệch dần sau go-live — đúng cái rủi ro D10 sinh ra để giết. Quy tắc này là cam kết *tổ chức* (công ty khoá file), thuộc PRD chứ không thuộc addendum. *Fix:* thêm vào FR-090 hoặc §9: "Sau go-live một sản phẩm, hệ thống là nguồn chân lý duy nhất về công thức/giá; công ty khoá việc dùng file Excel làm căn cứ; mọi sửa đổi thực hiện trên hệ thống (audit log)."
- **medium** Thiếu Assumptions Index (toàn tài liệu) — ~10 thẻ `[ASSUMPTION]` rải từ §2 đến §7 nhưng không có bảng gom cuối PRD giao ai xác nhận, trước mốc nào. Với launch >100 đại lý, các giả định như NFR-05 (quy mô), FR-035 (làm tròn tiền — ảnh hưởng kế toán), pilot 15 đại lý — mỗi cái cần một cái tên và một deadline, không thể trôi. Addendum đã làm mẫu tốt cho NFR-05 ("cần chủ dự án xác nhận khi có số thật") — nhưng chỉ cho đúng một giả định. *Fix:* thêm §11 "Bảng giả định": mã, nội dung, ai xác nhận, hạn (gợi ý: trước khi kết thúc chặng kiến trúc).
- **medium** Rollout >100 đại lý không có vế vận hành (§9) — Kế hoạch phát hành chỉ có gate kỹ thuật (golden tests, số đo G2–G4). Onboard hàng trăm tài khoản, đào tạo đại lý dùng pipeline + chụp ảnh đúng chuẩn (điều kiện sống còn cho mock-up và counter-metric <10%), kênh hỗ trợ khi đại lý kẹt tại công trình — không dòng nào. Nếu đây là chủ đích (thuộc kế hoạch kinh doanh, không thuộc PRD) thì phải nói ra. *Fix:* thêm một dòng non-goal tường minh ("kế hoạch đào tạo/hỗ trợ đại lý nằm ngoài PRD, chủ dự án sở hữu") hoặc một hàng điều kiện vận hành trong bảng §9.

## 6. Downstream usability — **adequate**

Đây là PRD chain-top (nuôi UX → kiến trúc → epics) và phần lớn cơ chế truy vết hoạt động: FR đánh số ổn định theo nhóm, cột UC trỏ về tài liệu đặc tả tồn tại thật (đã kiểm tra: mọi mã UC-A1…UC-P7 được FR tham chiếu đều có mặt trong `ADG_CPQ_UseCase_ChiTiet.md`), addendum chia đúng ngăn cho ba chặng sau. Hai lỗ hổng: không có Glossary, và cách ly dữ liệu *giữa các* đại lý chỉ tồn tại ngầm.

### Findings

- **medium** Không có Glossary trong khi thuật ngữ nghiệp vụ dày đặc và dễ lẫn — "linh kiện" vs "phụ kiện" vs "vật tư", "giá vốn/cost_material" vs "giá nhập/import_price" vs "giá bán", "hạng mục", "gross 2 tầng" — downstream (đặc biệt UX writer và người viết story) sẽ tự suy và suy lệch; ranh giới "linh kiện tính theo số lượng đại lý nhập, phụ kiện luôn số lượng 1, vật tư ẩn hoàn toàn" là loại tri thức phải tra được một chỗ. *Fix:* thêm Glossary ~10 mục ngay sau §1.
- **medium** Cách ly dữ liệu giữa các đại lý không là yêu cầu kiểm chứng được — §3 nói đại lý "xem lại báo giá của đại lý mình" và FR-040 nói "kho khách riêng theo đại lý", nhưng NFR-01 chỉ yêu cầu ca test field-level ("dealer cố đọc giá vốn"). Với >100 đại lý cạnh tranh nhau, đại lý A đọc được kho khách/báo giá của đại lý B là sự cố kinh doanh ngang rò giá vốn. *Fix:* bổ sung NFR-01: kiểm thử bảo mật phải có ca "tài khoản đại lý A truy vấn tài nguyên (khách, báo giá, PDF) của đại lý B qua API → bị chặn".
- **low** Không FR nào trỏ UC-03 (chọn sản phẩm cho hạng mục) — hành vi chọn sản phẩm từ cây danh mục, cảnh báo xoá dữ liệu khi đổi sản phẩm (UC-03 2a), ẩn sản phẩm vô hiệu (E-2) chỉ tồn tại bên nguồn; FR-041 mới nói "mỗi hạng mục: một sản phẩm". *Fix:* thêm UC-03 vào cột UC của FR-041 và một vế "đổi sản phẩm sau khi đã cấu hình → cảnh báo mất dữ liệu bước sau".

## 7. Shape fit — **strong**

Đúng khổ: B2B đa vai + UX di động quan trọng → UJ có nhân vật tên riêng gánh bối cảnh (anh Tùng tại công trình, chị Mai không cần dev); thân PRD giữ ở mức yêu cầu, chi tiết luồng đẩy về đặc tả UC, chi tiết kỹ thuật đẩy về addendum — ba tầng đúng độ cao. Độ dài thân PRD (~220 dòng) là *xứng tầm* chứ không mỏng, **với điều kiện** tài liệu UC v2 được coi là phụ lục hành vi chính thức của PRD — điều mà header §6 đã tuyên bố tường minh. Không có dấu hiệu over-formalize.

### Findings

- Không có finding riêng; điều kiện "UC v2 là phụ lục chính thức" cần giữ nguyên khi UC doc thay đổi — mọi sửa UC phải sync mã D/F/O như quy ước decision log đã đặt.

---

## Đối chiếu ngược nguồn (reconciliation D1–D13, F1–F5, O1–O4)

| Mã | Trạng thái trong PRD | Ghi chú |
|---|---|---|
| D1 | ✅ FR-041, §4, UJ-1 | Đủ, kể cả sao chép hạng mục |
| D2 | ✅ FR-012, FR-044 | Giữ đúng "hệ thống lọc, đại lý chọn" |
| D3 | ✅ FR-032 | Bỏ band_percent đúng; **nhưng** hành vi range-trống của nguồn bị bỏ → xung đột G4 (finding high §5) |
| D4 | ✅ FR-060/061/065, §4 ngoài phạm vi | Đủ cả hai vế tính lại + đóng băng |
| D5 | ⚠️ §4 Phase 2 | Có scope, **thiếu vế "chừa hook, không hard-code giá"** ở cả PRD lẫn addendum (finding low §3) |
| D6 | ✅ FR-042/050/051/052 + counter-metric | Mẫu mực — có cả giám sát lạm dụng lối thoát |
| D7 | ✅ FR-034, §4 Phase 2 (VAT phân hoá) | Đủ |
| D8 | ✅ NFR-03 | Có nhưng chưa đo được (finding medium §2) |
| D9 | ✅ §3 | Đủ, kể cả "khách không truy cập" |
| D10 | ⚠️ FR-022/090/091, addendum | Import + golden tests đủ; **thiếu vế "khoá Excel sau go-live"** (finding medium §5) |
| D11 | ✅ NFR-04, §4, §8, addendum (nguyên văn) | Đủ; riêng FR-063 gắn nhầm nhãn ASSUMPTION (mechanical) |
| D12 | ✅ §4, F6 | Đủ |
| D13 | ✅ §2 (thứ tự ưu tiên), §3, §9 | Đủ |
| F1 | ✅ FR-042 | Ảnh nullable nháp, bắt buộc phát hành |
| F2 | ✅ FR-061 | Snapshot khách |
| F3 | ✅ FR-061 | Snapshot sản phẩm từng hạng mục |
| F4 | ✅ FR-021 | Đúng một BOM active |
| F5 | ✅ FR-051 | Quy tắc D6 thay tiền điều kiện cũ |
| O1–O4 | ✅ §10 + §8 | Đủ chủ chốt + điều kiện chặn, khớp decision log |
| O7 (đã đóng) | ❌ | Rủi ro màu ↔ mock-up bị rơi (finding medium §5) |
| UC-A5 | ❌ | **Không FR nào phủ quản trị phụ kiện** (finding high §4) |
| UC-03 | ⚠️ | Không FR nào trỏ tới (finding low §6) |

## Mechanical notes

- **Nhãn sai:** FR-063 gắn `[ASSUMPTION — kênh tự động là should-have MVP]` nhưng đây là nội dung **đã chốt trong D11** ("Zalo OA tự động (should-have)"). Đổi thành `[D11]` — để nguyên sẽ làm loãng giá trị của thẻ ASSUMPTION.
- **Assumptions Index roundtrip:** fail — có thẻ inline, không có index (đã nêu ở §5).
- **ID continuity:** FR nhảy số theo nhóm (001→010→020…) là chủ đích, chấp nhận được; không trùng, không gãy. Hai hàng cuối bảng §10 dùng "—" thay mã — nên cấp mã (O5/O6 đã dùng rồi trong decision log; gợi ý O-A, O-B hoặc tiếp số O8, O9 kèm chú thích để không đụng O5–O8 lịch sử).
- **Cross-refs:** mọi tham chiếu UC trong cột UC của FR đều resolve về `ADG_CPQ_UseCase_ChiTiet.md`. FR-022 trỏ đặc tả ngôn ngữ công thức sang "ERD Project Note" — hợp lệ nhưng là tham chiếu ra ngoài bộ PRD; addendum đã nhắc lại đường dẫn (`docs/ADG_CPQ_ERD.dbml`), giữ đồng bộ khi ERD đổi.
- **UJ protagonist:** cả 3 UJ có nhân vật/ngôi rõ (UJ-3 là "khách" vô danh — chấp nhận được vì khách không phải người dùng hệ thống).
- **Glossary drift tiềm ẩn:** "giá vốn" (§3, UJ-2) vs "cost_material" (FR-030) vs "giá vốn NVL" (nguồn UC-08) — sẽ hết khi có Glossary.

## Tổng hợp findings

| Mức | Số lượng | Danh sách |
|---|---|---|
| Critical | 0 | — |
| High | 2 | UC-A5 không có FR; G4 vs range-trống |
| Medium | 8 | FR-060 thiếu điều kiện phát hành gom; O7 rơi; khoá Excel (D10) rơi; thiếu Assumptions Index; rollout vận hành; NFR-03 không đo được; thiếu Glossary; cách ly đa đại lý chưa kiểm chứng được |
| Low | 5 | Gate pilot vs G2; D5 hook; hai lớp lỗi công thức; FR-042 độ phân giải; UC-03 không được trỏ |

**Điều kiện để ĐẠT:** sửa 2 finding high; khuyến nghị mạnh xử lý luôn 4 medium đầu (điều kiện phát hành, O7, khoá Excel, Assumptions Index) — tổng chi phí sửa ước tính dưới một buổi làm việc.
