---
title: "PRD — ADG CPQ: Hệ thống báo giá tự động cho đại lý Austdoor"
status: final
created: 2026-07-03
updated: 2026-07-03
---

# PRD — ADG CPQ: Hệ thống báo giá tự động cho đại lý Austdoor

> **Nguồn:** Bộ tài liệu nghiệp vụ v2 (`docs/`) + sổ quyết định `ADG-CPQ-decision-log.md` (mã D/F/O) + phỏng vấn chủ dự án 02–03/07/2026.
> **Quy ước:** `[ASSUMPTION]` = giả định của người soạn, cần chủ dự án xác nhận. `[OPEN — Ox]` = quyết định treo có chủ, đã có cách xử lý tạm.

## 1. Bối cảnh & vấn đề

Đại lý của Austdoor hiện báo giá cửa (cửa cuốn, cửa nhôm...) bằng cách tính tay dựa trên bảng giá Excel và kinh nghiệm: chậm, dễ sai, mỗi nơi một giá, và công ty không kiểm soát được giá bán lẻ lẫn thông tin giá vốn. Phiếu vật tư gửi nhà máy lập thủ công nên đơn sản xuất sai theo.

**ADG CPQ** là hệ thống báo giá tự động: đại lý đứng tại công trình, nhập thông số trên điện thoại, hệ thống tính vật tư (BOM) và giá theo công thức công ty quản trị tập trung, sinh báo giá PDF chuyên nghiệp kèm **ảnh mock-up AI** (ghép sản phẩm vào ảnh hiện trường nhà khách), gửi khách và đẩy phiếu BOM về nhà máy — khép kín từ khảo sát đến sản xuất.

## 2. Mục tiêu & chỉ số thành công

Mục tiêu xếp theo thứ tự ưu tiên (chốt với chủ dự án 03/07/2026):

| # | Mục tiêu | Chỉ số đo | Mục tiêu cụ thể |
|---|---|---|---|
| G1 | **Tăng tỷ lệ chốt đơn** | Tỷ lệ báo giá phát hành → thành đơn hàng | Baseline chưa có — hệ thống phải **đo được từ ngày đầu**; đặt target sau 3 tháng dữ liệu `[ASSUMPTION]` |
| G2 | **Giảm sai sót BOM về nhà máy** | Tỷ lệ phiếu BOM bị nhà máy trả lại/sửa | ≤ 1% sau 6 tháng `[ASSUMPTION]` |
| G3 | **Rút ngắn thời gian báo giá** | Thời gian trung bình tạo → phát hành | ≤ 15 phút cho báo giá 1–3 hạng mục `[ASSUMPTION]` |
| G4 | **Chuẩn hoá & kiểm soát giá** | % báo giá phát hành **tuân thủ chính sách giá hiện hành**: nằm trong min–max nếu sản phẩm có cấu hình range, và không dưới giá nhập trừ khi cờ `allow_below_import` được bật | 100% (ràng buộc cứng của hệ thống) |

**Chỉ số đối trọng (counter-metrics)** — để mục tiêu không bị đạt "bằng mọi giá":

- Tỷ lệ báo giá phát hành **không kèm mock-up** < 10% (giám sát lạm dụng lối thoát D6; cao hơn nghĩa là dịch vụ AI kém hoặc hướng dẫn chụp không hiệu quả).
- Tỷ lệ đại lý **bỏ dở pipeline** (tạo nháp nhưng không phát hành trong 7 ngày) — theo dõi để phát hiện UX gây tắc.
- Số sự cố "công thức sai làm giá sai hàng loạt" = 0 (bảo vệ bằng golden tests, FR-090).

## 3. Người dùng & vai trò

Đúng **hai vai đăng nhập** [D9]:

| Vai | Là ai | Làm gì | Không được làm |
|---|---|---|---|
| **Admin** | Nhân viên tổng công ty Austdoor (phòng sản phẩm, ban giá) | Quản trị catalog, thông số, công thức, tương thích linh kiện, vật tư/BOM, giá (gross, min–max), cấu hình hệ thống; xem audit log | Không thao tác hộ pipeline báo giá của đại lý `[ASSUMPTION]` |
| **Đại lý (dealer)** | Chủ/nhân viên đại lý — một đại lý có nhiều tài khoản, dùng chung dữ liệu đại lý đó | Chạy pipeline báo giá 4 bước; quản lý kho khách riêng; tải/gửi PDF; xem lại báo giá của đại lý mình | **Tuyệt đối không thấy**: giá vốn, đơn giá vật tư, chi tiết BOM, mọi % gross |

**Khách hàng cuối không truy cập hệ thống** — chỉ nhận báo giá PDF qua kênh gửi.

Quy mô: triển khai **toàn mạng lưới >100 đại lý** trong năm đầu (chốt 03/07/2026); rollout theo đợt — pilot ~15 đại lý & 1–2 dòng sản phẩm trong 4–6 tuần đầu trước khi mở toàn quốc `[ASSUMPTION]`.

## 4. Phạm vi

### Trong MVP

1. Toàn bộ pipeline báo giá 4 bước, **nhiều hạng mục/báo giá** [D1].
2. Cấu hình sản phẩm data-driven: thông số, linh kiện + **lọc tương thích** [D2], phụ kiện, BOM công thức (có rẽ nhánh if/else [D10]), giá (gross 2 tầng, min–max [D3]).
3. **Ảnh mock-up AI trong MVP** (chốt 03/07/2026) với quy tắc 3 tầng không gây tắc [D6].
4. Phát hành: tính lại giá [D4], snapshot đóng băng, PDF nhiều hạng mục, hạn hiệu lực [O3], xuất phiếu BOM sản xuất.
5. Công cụ import công thức từ Excel + **golden tests** [D10].
6. Quản trị: người dùng, đại lý, đơn vị tính, cấu hình hệ thống, audit log.

### Ngoài MVP (Phase 2+, thiết kế phải chừa chỗ)

- Vỏ **Zalo Mini App** bọc web đại lý (web-first — quyết định D11; ZMA chỉ là cổng, dùng chung API/UI web).
- Chính sách khuyến mãi/chiết khấu từ công ty [D5]; lịch sử giá vật tư; phiên bản BOM.
- Theo dõi phản hồi khách sau gửi; báo cáo/thống kê nâng cao (UC-P7); VAT phân hoá theo sản phẩm [D7].
- Tự động đánh giá chất lượng nội dung ảnh hiện trường.

### Ngoài phạm vi vĩnh viễn (của hệ thống này)

- Quản lý đơn hàng/hợp đồng/thanh toán sau báo giá; điều chỉnh giá trong thi công [D4].

## 5. Hành trình người dùng

### UJ-1 — Anh Tùng, chủ đại lý ở Hải Phòng, báo giá tại công trình

Anh Tùng đến nhà khách đang xây, khách muốn lắp 2 cửa cuốn mặt tiền (giống nhau) và 1 cửa nhôm bên hông. Trên **điện thoại**, anh mở CPQ, tạo báo giá: nhập SĐT khách — hệ thống nhận ra khách cũ từng mua. Anh thêm **hạng mục 1** (cửa cuốn, số lượng 2): chụp ảnh mặt tiền theo hướng dẫn, nhập cao × rộng, hệ thống chỉ hiện đúng 2 bộ tời tương thích với kích thước đó — anh chọn 1, tick thêm bộ lưu điện. Anh bấm "sao chép hạng mục" cho cửa nhôm rồi sửa lại thông số và ảnh. Hệ thống tính ra **giá nhập từng hạng mục** (anh không thấy giá vốn hay gross); anh chọn giá bán trong khoảng cho phép — hạ giá cửa nhôm một chút vì khách quen. Hệ thống ghép mock-up AI: cửa cuốn lên ảnh mặt tiền trông ổn ngay lần đầu, cửa nhôm phải tạo lại 1 lần. Anh bấm phát hành: PDF một trang tổng + chi tiết từng hạng mục kèm ảnh, có hạn hiệu lực, gửi cho khách; phiếu BOM tự về nhà máy. Tổng thời gian: **dưới 15 phút**, ngay tại công trình.

### UJ-2 — Chị Mai, phòng sản phẩm Austdoor, lên sản phẩm mới không cần dev

Công ty ra dòng cửa cuốn mới. Chị Mai (admin) tạo sản phẩm, khai bộ thông số (cao, rộng, màu — màu là dropdown), gán các loại linh kiện kèm công thức auto-fill và điều kiện tương thích ("bộ tời X: diện tích ≤ 25m²"), dựng BOM bằng công thức có rẽ nhánh ("cao > 3m → ray loại nặng"), đặt gross và range min–max. Chị bấm **kiểm thử** với bộ thông số mẫu: xem danh sách SKU lọt bộ lọc, lượng vật tư, giá vốn → giá nhập. Sai chỗ nào sửa chỗ đó, không cần release phần mềm. Mọi thay đổi giá đều có vết trong audit log.

### UJ-3 — Khách nhận báo giá

Khách nhận PDF: thông tin từng hạng mục, ảnh "nhà mình sau khi lắp cửa", đơn giá — thành tiền — tổng — VAT, hạn hiệu lực. Không thấy bất kỳ thông tin giá vốn/nội bộ nào.

## 6. Yêu cầu chức năng

> FR đánh số toàn cục, ổn định. Nguồn chi tiết: cột "UC" trỏ về `docs/ADG_CPQ_UseCase_ChiTiet.md`; đây là bản đặc tả hành vi đầy đủ — PRD không lặp lại luồng từng bước.

### F1 — Catalog & cấu hình sản phẩm (Admin)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-001 | Quản lý cây danh mục nhiều cấp và hồ sơ sản phẩm (tên, SKU duy nhất, thumbnail, mô tả, trạng thái); vô hiệu hoá thay cho xoá cứng | UC-A1 |
| FR-002 | Khai báo bộ thông số dự án **riêng theo từng sản phẩm** (kiểu dữ liệu, kiểu nhập freeform/dropdown, đơn vị, bắt buộc); option dropdown có `value` dùng được trong công thức | UC-A2 |
| FR-003 | Thêm sản phẩm/thông số mới hoàn toàn bằng dữ liệu cấu hình — **không cần thay đổi schema hay code** | A0 |
| FR-004 | Quản lý phụ kiện dùng chung (tên, thumbnail, đơn giá — thay đổi giá ghi audit log) và gán danh sách phụ kiện khả dụng vào từng sản phẩm | UC-A5 |

### F2 — Linh kiện & tương thích (Admin + Hệ thống)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-010 | Quản lý loại linh kiện dùng chung, bộ thông số của loại, SKU linh kiện kèm đơn giá và giá trị thông số mặc định | UC-A3 |
| FR-011 | Gán loại linh kiện vào sản phẩm; khai công thức auto-fill thông số linh kiện **theo cặp sản phẩm↔loại** | UC-A4 |
| FR-012 | Khai **điều kiện tương thích** cho từng SKU theo cặp sản phẩm↔loại; hệ thống chỉ hiển thị SKU hợp lệ theo thông số đã nhập — *hệ thống lọc, đại lý chọn* [D2] | UC-A4, UC-06 |
| FR-013 | Công cụ kiểm thử cho admin: nhập bộ thông số mẫu → xem SKU lọt bộ lọc + kết quả auto-fill; cảnh báo tổ hợp thông số cho ra 0 SKU | UC-A4 |

### F3 — Vật tư & BOM (Admin + Hệ thống)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-020 | Quản lý danh mục vật tư với một đơn giá hiện hành; đơn giá ẩn với đại lý | UC-A6 |
| FR-021 | BOM template theo sản phẩm, **đúng một bản active mỗi sản phẩm** [F4]; mỗi dòng có công thức số lượng theo thông số dự án | UC-A6 |
| FR-022 | Ngôn ngữ công thức hỗ trợ: số học, so sánh, logic, `if(...)`, `round/ceil/floor/min/max`, biến `proj.*`/`comp.*`, so sánh giá trị select — đặc tả trong ERD Project Note [D10] | — |
| FR-023 | Công thức lỗi khi chạy (chia 0, biến thiếu, vật tư thiếu giá) → dừng tính, báo lỗi rõ cho đại lý và cảnh báo admin; không bao giờ ra giá sai âm thầm | UC-08 |

### F4 — Định giá (Hệ thống + Admin)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-030 | Tính giá **theo từng hạng mục**: cost_material (BOM + linh kiện + phụ kiện) → giá nhập = cost × (1 + gross%) | UC-08 |
| FR-031 | Gross 2 tầng: global theo sản phẩm + gross đại lý; mức đại lý-theo-sản-phẩm **thay thế** mức chung nếu có | UC-A7 |
| FR-032 | Range giá bán chỉ dùng **min–max theo sản phẩm** [D3]; giá trị do công ty nhập `[OPEN — O1]`; validate min ≤ max | UC-A7, UC-09 |
| FR-033 | Sàn giá: mặc định **giá bán ≥ giá nhập**, cờ cấu hình `allow_below_import` để nới `[OPEN — O2]` | UC-09 |
| FR-034 | VAT một mức chung toàn hệ thống [D7]; tổng báo giá = Σ thành tiền hạng mục + VAT | UC-09 |
| FR-035 | Tiền VND làm tròn về đồng tại từng dòng; tổng = cộng giá trị đã làm tròn `[ASSUMPTION — chờ kế toán xác nhận]` | — |
| FR-036 | Toàn bộ dữ liệu mật (cost, đơn giá vật tư, BOM, gross) **ẩn với vai dealer ở tầng API** — không chỉ ẩn trên giao diện | NFR-S |

### F5 — Pipeline báo giá (Đại lý)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-040 | Tạo báo giá gắn một khách hàng; kho khách riêng theo đại lý, định danh bằng SĐT (unique trong đại lý) | UC-01 |
| FR-041 | Báo giá chứa **nhiều hạng mục** [D1]; mỗi hạng mục: một sản phẩm + số lượng + cấu hình riêng; thêm/xoá/**sao chép** hạng mục | UC-02 |
| FR-042 | Mỗi hạng mục có **một ảnh hiện trường** (≤ 8MB, kiểm tra định dạng + độ phân giải tối thiểu, kèm hướng dẫn chụp); cho trống khi nháp, bắt buộc trước phát hành [F1][D6] | UC-04 |
| FR-043 | Nhập thông số dự án theo form sinh từ cấu hình; validate kiểu dữ liệu và bắt buộc | UC-05 |
| FR-044 | Chọn SKU linh kiện trong danh sách đã lọc tương thích; auto-fill thông số theo công thức; đại lý chỉnh được (đánh dấu overridden) | UC-06 |
| FR-045 | Tick chọn phụ kiện (số lượng luôn 1/hạng mục) | UC-07 |
| FR-046 | Chọn giá bán từng hạng mục trong range; xem giá nhập, thành tiền, tổng, VAT theo thời gian thực | UC-09 |
| FR-047 | **Lưu nháp mọi bước**, khôi phục đúng chỗ đang dở (bước của quote + bước của từng hạng mục); sửa bước trước → tự tính lại phần phụ thuộc, SKU mất tương thích buộc chọn lại | UC-12 |
| FR-048 | Tìm kiếm báo giá (SĐT khách, mã, trạng thái, ngày); **sao chép báo giá cũ** thành nháp mới (không copy ảnh/giá đã chốt) | UC-P5 |

### F6 — Ảnh mock-up AI (Đại lý + Dịch vụ AI) — trong MVP

| ID | Yêu cầu | UC |
|---|---|---|
| FR-050 | Sinh mock-up **theo hạng mục** từ ảnh hiện trường + ảnh sản phẩm qua dịch vụ AI; tối đa N lần/hạng mục (mặc định 3, cấu hình được); gen lỗi kỹ thuật không tính lượt | UC-10 |
| FR-051 | Quy tắc 3 tầng [D6]: bắt buộc theo nguyên tắc (cờ `mockup_required`); hết lượt → thay ảnh hiện trường (reset lượt) **hoặc** phát hành không kèm ảnh với lý do bắt buộc, có log giám sát | UC-10 |
| FR-052 | Đại lý duyệt tối đa một ảnh/hạng mục để đính vào PDF | UC-10 |

### F7 — Phát hành & gửi đi (Hệ thống)

| ID | Yêu cầu | UC |
|---|---|---|
| FR-060 | Phát hành = **tính lại toàn bộ** theo cấu hình mới nhất [D4]; giá nhập lệch so với nháp → buộc đại lý xác nhận lại giá bán | UC-11 |
| FR-061 | Snapshot đóng băng vĩnh viễn khi phát hành: khách hàng [F2], sản phẩm từng hạng mục [F3], thông số, linh kiện, phụ kiện, BOM, mọi con số giá | UC-11 |
| FR-062 | Sinh PDF: thông tin khách, bảng hạng mục (sản phẩm — SL — đơn giá — thành tiền), ảnh mock-up từng hạng mục, tổng + VAT, **hạn hiệu lực** (validity_days cấu hình, trống = không in `[OPEN — O3]`) | UC-11 |
| FR-063 | Gửi PDF cho khách: MVP tối thiểu = đại lý **tải PDF gửi thủ công** (bắt buộc); gửi tự động qua Zalo OA là should-have (theo D11); trạng thái gửi + gửi lại khi lỗi | UC-11, UC-P4 |
| FR-064 | Xuất **phiếu BOM sản xuất** gộp mọi hạng mục × số lượng; MVP = xuất file chuẩn (PDF/Excel), kênh gửi tự động về nhà máy chốt sau `[OPEN — O4]`; trạng thái gửi + gửi lại | UC-11, UC-P4 |
| FR-065 | Báo giá đã phát hành: chỉ đọc; muốn sửa → sao chép thành báo giá mới | UC-12 |

### F8 — Quản trị, phân quyền & audit

| ID | Yêu cầu | UC |
|---|---|---|
| FR-070 | Quản lý tài khoản 2 vai (admin/dealer); tài khoản dealer gắn đại lý; nhiều tài khoản/đại lý dùng chung dữ liệu đại lý | UC-P1 |
| FR-071 | Quản lý hồ sơ đại lý (onboard, gross mặc định, gross theo sản phẩm, vô hiệu hoá) | UC-P2 |
| FR-072 | Quản lý đơn vị tính và cấu hình hệ thống (VAT, lượt gen AI, mockup_required, allow_below_import, validity_days) | UC-P3 |
| FR-073 | **Audit log** mọi thay đổi dữ liệu giá/công thức/cấu hình: ai, lúc nào, đổi gì; chỉ đọc | UC-P6 |
| FR-074 | Số liệu vận hành tối thiểu cho G1–G4: đếm báo giá theo trạng thái/đại lý/sản phẩm, thời gian tạo→phát hành, tỷ lệ không kèm mock-up `[ASSUMPTION — dashboard lớn là Phase 2, MVP cần số thô]` | UC-P7 |

### F9 — Import công thức & golden tests (Admin) — trong MVP

| ID | Yêu cầu | UC |
|---|---|---|
| FR-090 | Quy trình import logic BOM/công thức từ bảng tính công ty (AI hỗ trợ) khi bảng tính được chốt; **nghiệm thu bằng golden tests**: chạy song song Excel ↔ hệ thống trên bộ thông số thật, khớp 100% mới go-live sản phẩm. Sau go-live, **hệ thống là nguồn chân lý duy nhất** — file Excel không còn là căn cứ giá, sửa công thức chỉ trên hệ thống [D10] | UC-A8 |
| FR-091 | Bộ golden tests lưu lại, chạy hồi quy mỗi khi công thức của sản phẩm bị sửa | UC-A8 |

## 7. Yêu cầu phi chức năng

| ID | Nhóm | Yêu cầu |
|---|---|---|
| NFR-01 | **Bảo mật dữ liệu giá** | Ẩn dữ liệu mật thực thi ở tầng API theo vai (field-level); kiểm thử bảo mật phải có ca "dealer cố đọc giá vốn qua API" |
| NFR-02 | Bảo mật chung | Đăng nhập có phiên an toàn; mật khẩu băm; HTTPS toàn bộ; dữ liệu cá nhân khách hàng (SĐT, địa chỉ) tuân thủ Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân |
| NFR-03 | **Mobile-first** | Toàn bộ pipeline đại lý dùng mượt trên điện thoại (thao tác một tay tại công trình, chụp ảnh trực tiếp); admin tối ưu cho desktop [D8] |
| NFR-04 | Nền tảng | Web app responsive [D11 — web trước]; kiến trúc frontend phải **bọc được vào Zalo Mini App ở Phase 2** mà không viết lại (ZMA chỉ là cổng/vỏ) |
| NFR-05 | Quy mô | ≥ 300 tài khoản đại lý hoạt động, ~50 người dùng đồng thời, ~5.000 báo giá/tháng năm đầu `[ASSUMPTION]` — không cần kiến trúc phân tán phức tạp, cần đúng và ổn |
| NFR-06 | Hiệu năng | Tính BOM + giá một báo giá ≤ 3 giây; sinh mock-up AI chấp nhận ≤ 60 giây/lần có hiển thị tiến trình `[ASSUMPTION]` |
| NFR-07 | Độ tin cậy gửi | Gửi khách/nhà máy thất bại không mất dữ liệu, không chặn phát hành, luôn gửi lại được [UC-P4] |
| NFR-08 | Ngôn ngữ | Toàn bộ UI, PDF, thông báo bằng tiếng Việt; tiền tệ VND |
| NFR-09 | Toàn vẹn số liệu | Báo giá đã phát hành bất biến vĩnh viễn (snapshot); mọi thay đổi cấu hình có audit trail |
| NFR-10 | Đồng thời | Hai tài khoản cùng đại lý sửa cùng một nháp: khoá lạc quan + cảnh báo, không ghi đè âm thầm `[ASSUMPTION]` |

## 8. Ràng buộc & phụ thuộc

| Phụ thuộc | Ảnh hưởng | Trạng thái |
|---|---|---|
| **Dịch vụ AI ghép ảnh** (bên thứ ba hoặc model tự chọn) | FR-050..052, G1 | Chưa chọn nhà cung cấp — việc của chặng kiến trúc; PRD chỉ yêu cầu năng lực |
| **Bảng tính công thức chính thức** từ công ty | FR-090 go-live từng sản phẩm | `[OPEN]` — công ty chưa chốt; không chặn build, chặn go-live |
| **Giá trị min–max, sàn giá, hiệu lực** | FR-032/033/062 | `[OPEN — O1, O2, O3]` — giá trị cấu hình, nhập sau |
| **Quy trình nhà máy** (kênh, format, định tuyến) | FR-064 phần gửi tự động | `[OPEN — O4]` — MVP xuất file chuẩn trước |
| **Zalo OA / Mini App** | FR-063 kênh tự động; NFR-04 Phase 2 | Web-first [D11]; OA là should-have MVP |

## 9. Kế hoạch phát hành

| Giai đoạn | Nội dung | Điều kiện ra cổng |
|---|---|---|
| **MVP — build** | Toàn bộ mục 4 "Trong MVP" | Vượt reviewer gate + test |
| **Pilot** `[ASSUMPTION]` | ~15 đại lý, 1–2 dòng sản phẩm, 4–6 tuần | Sản phẩm pilot qua golden tests (FR-090); đo được G2–G4 |
| **Toàn quốc** | >100 đại lý, thêm dòng sản phẩm dần theo golden tests | Pilot đạt: BOM sai ≤ 1%, thời gian báo giá ≤ 15 phút, không sự cố giá |
| **Phase 2** | Vỏ Zalo Mini App, khuyến mãi công ty, báo cáo nâng cao, lịch sử giá | Theo backlog sau pilot |

## 10. Câu hỏi mở

| # | Câu hỏi | Ai chốt | Chặn gì |
|---|---|---|---|
| O1 | Giá trị min–max theo sản phẩm | Ban giá | Không chặn build — nhập cấu hình |
| O2 | Cho bán dưới giá nhập? (mặc định: không) | Ban giá/sếp | Không chặn |
| O3 | Số ngày hiệu lực báo giá | Sếp | Không chặn |
| O4 | Kênh/format gửi BOM nhà máy | BP sản xuất | Chặn phần gửi tự động, không chặn xuất file |
| — | Bảng tính công thức chính thức | Công ty | **Chặn go-live từng sản phẩm** (không chặn build) |
| — | Quy tắc làm tròn tiền (đề xuất FR-035) | Kế toán | Không chặn — sửa cấu hình được |
| — | Nhà cung cấp dịch vụ AI ghép ảnh | Chặng kiến trúc | Chặn FR-050 |
| — | Ảnh sản phẩm dùng ghép mock-up phải theo **màu đã chọn** khi màu là thông số dropdown (một thumbnail không đủ — rủi ro đã ghi nhận ở O7) | Chủ dự án + catalog thật | Chặn chất lượng FR-050, không chặn build |

## 11. Danh mục giả định & thuật ngữ

**Giả định chờ xác nhận** (tổng hợp các thẻ `[ASSUMPTION]`): target G1 đặt sau 3 tháng dữ liệu; BOM sai ≤ 1% (G2); thời gian báo giá ≤ 15 phút (G3); admin không thao tác hộ pipeline; rollout pilot ~15 đại lý 4–6 tuần; quy mô 300 tài khoản / 50 đồng thời / 5.000 báo giá·tháng; mock-up ≤ 60 giây; làm tròn về đồng từng dòng (chờ kế toán); MVP cần số liệu thô cho G1–G4 (dashboard là Phase 2); khoá lạc quan khi đồng thời sửa nháp.

**Thuật ngữ:** *Hạng mục* = một dòng sản phẩm trong báo giá (một cửa/nhóm cửa giống nhau). *Giá nhập* = giá đại lý mua từ công ty (= giá vốn + gross, đại lý xem được). *Gross* = % lợi nhuận gộp (mật). *BOM* = danh sách vật tư sản xuất tính từ thông số. *Mock-up* = ảnh AI ghép sản phẩm vào ảnh hiện trường. *Golden tests* = bộ ca đối chiếu Excel ↔ hệ thống để nghiệm thu công thức. *Snapshot* = bản chụp dữ liệu đóng băng khi phát hành.
