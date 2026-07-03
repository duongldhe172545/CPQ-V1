# Review nghiệp vụ — Bộ tài liệu ADG CPQ V2

> **Người review:** Reviewer nghiệp vụ (adversarial), 03/07/2026
> **Phạm vi:** `ADG_CPQ_V2\02_NghiepVu_TrietLyThietKe.md`, `ADG_CPQ_V2\01_TomTat_ThayDoi.md`, `ADG_CPQ_V2\05_SoQuyetDinh_DecisionLog.md` (đối chiếu thêm `04_ERD.dbml` Project Note và `03_UseCase_ChiTiet.md` UC-A8/A6 vì 02 trỏ tới đó làm đặc tả chuẩn).
> **Chuẩn đối chiếu — hướng đã chốt 03/07:** bộ máy tính BOM nội bộ **hỗ trợ được MỌI công thức**; **IT/Data Analyst đọc file Excel giá của công ty rồi NHẬP CÔNG THỨC VÀO HỆ THỐNG bằng tay** — không phải AI tự import, không phải Excel-as-engine.

## VERDICT: **ỔN CÓ ĐIỀU KIỆN**

Bộ tài liệu nhất quán nội bộ ở mức tốt (các quyết định D1–D9 chắc), nhưng (1) toàn bộ tuyến "AI-import" phải viết lại theo hướng nhập tay đã chốt 03/07, và (2) ngôn ngữ công thức hiện đặc tả **chưa đủ** cho tham vọng "MỌI công thức" — thiếu bảng tra, biến trung gian/tham chiếu dòng, hằng số dùng chung, ngữ nghĩa lỗi. Chưa sửa 2 nhóm này thì chưa nên coi tài liệu là baseline để làm tiếp.

---

## (a) Nhất quán nội bộ giữa 3 file

**Nhìn chung nhất quán.** Cấu trúc nhiều hạng mục [D1], lọc SKU [D2], min–max [D3], vòng đời giá [D4], quy tắc mock-up 3 tầng [D6], 2 vai [D9], VAT chung [D7] được kể giống nhau ở cả 3 file. Các điểm đá nhau / lỗi thời tìm được:

| # | Vị trí | Vấn đề |
|---|--------|--------|
| a1 | `05_SoQuyetDinh_DecisionLog.md` **dòng 31 (F4)**: "*(Có thể vô hiệu nếu O6 chọn phương án Excel-engine.)*" và **dòng 50 (mục D)**: "*(sẽ vô hiệu nếu O6 chọn Excel-engine...)*" | O6 đã ĐÓNG (dòng 43) → hai ghi chú này lỗi thời, gây cảm giác phương án Excel-engine còn sống. Xoá. |
| a2 | `05` **dòng 22 (D10)** yêu cầu ngôn ngữ công thức có "rẽ nhánh (if/else, **bảng tra theo lựa chọn**)" — nhưng đặc tả chuẩn trong `04_ERD.dbml` Project Note (dòng 31–40) **không có construct bảng tra nào**, chỉ có `if()`. | D10 hứa một năng lực mà đặc tả chưa hiện thực → mâu thuẫn giữa quyết định và đặc tả. (Trùng với finding c1.) |
| a3 | `02` **dòng 4** trỏ decision log về `_bmad-output/planning-artifacts/ADG-CPQ-decision-log.md`, trong khi V2 có bản riêng `ADG_CPQ_V2\05_SoQuyetDinh_DecisionLog.md` — **tồn tại 2 bản decision log song song**. `02` **dòng 8, 10** gọi tên file `ADG_CPQ_ERD.dbml` / `ADG_CPQ_UseCase_ChiTiet.md` nhưng tên thật trong thư mục V2 là `04_ERD.dbml` / `03_UseCase_ChiTiet.md`. | Rủi ro trôi dị bản khi cập nhật hướng mới (chỉ sửa 1 trong 2 bản log). Chốt 1 bản là "sống", bản kia ghi rõ deprecated/redirect; sửa tên file tham chiếu. |
| a4 | `02` **dòng 57**: markdown vỡ "range min–max theo sản phẩm **[D3** — ... **OPEN O1]**" | Lỗi trình bày nhỏ, sửa cho khỏi hiểu nhầm phạm vi ngoặc. |
| a5 | `01` **dòng 56 (O3)**: "hiện: **chưa in** hạn lên PDF" vs `02` dòng 63: "cơ chế **đã sẵn**, số ngày OPEN" | Không mâu thuẫn thật (nullable, trống = không in) nhưng cách diễn đạt ngược chiều nhau — nên đồng nhất: "cơ chế sẵn, mặc định trống = không in". |

## (b) Khớp hướng đã chốt 03/07? — Danh sách đích danh các chỗ còn "AI-import"

Hướng đã chốt: **người (IT/DA) đọc Excel và nhập công thức vào hệ thống**. Phần "toàn bộ logic BOM nằm trong hệ thống, không Excel-as-engine" của D10 **vẫn đúng hướng** — chỉ sai ở **cơ chế đưa công thức vào**. Các chỗ phải sửa:

| # | File / vị trí | Nội dung hiện tại | Sửa thành |
|---|---------------|-------------------|-----------|
| b1 | `02_NghiepVu_TrietLyThietKe.md` **dòng 22** (A0, [D10]) | "sẽ được **AI-import một lần** và nghiệm thu bằng golden tests (UC-A8)" | "được **IT/Data Analyst đọc và nhập công thức vào hệ thống**, nghiệm thu bằng golden tests (UC-A8)" |
| b2 | `01_TomTat_ThayDoi.md` **dòng 18** (mục ③) | "logic sẽ được **import (có AI hỗ trợ)** và nghiệm thu..." | "logic được **nhân sự IT/DA nhập vào hệ thống** và nghiệm thu..." |
| b3 | `05_SoQuyetDinh_DecisionLog.md` **dòng 22** (D10) | "nguồn để **AI-assisted import** một lần" + "UC **công cụ import** & golden tests" | "nguồn để IT/DA **đọc và nhập công thức**" + "UC **quy trình nhập công thức** & golden tests". Ghi bổ sung ngày 03/07 vào D10 (log là tài liệu "sống"). |
| b4 | `04_ERD.dbml` **dòng 21–22** (header) và **dòng 39–40** (Project Note) | "công thức cụ thể sẽ **AI-import** từ bảng tính công ty" | Cùng nội dung như b1. File này là đặc tả chuẩn mà 02 trỏ tới — bắt buộc sửa đồng bộ. |
| b5 | `03_UseCase_ChiTiet.md` **UC-A8** (dòng 303–333, đặc biệt dòng 308 "Admin (phối hợp đội phát triển/**AI**)" và dòng 320 "Logic được dịch (**AI-assisted**)") | UC đang mô tả là công cụ import AI | Viết lại UC-A8 = "Nhập logic BOM từ bảng tính (thủ công bởi IT/DA) & nghiệm thu golden tests". Luồng: khai thác logic ngầm với chủ file (giữ nguyên bước 1 — rất tốt) → IT/DA nhập công thức qua UC-A6/A4 → xây golden tests → chạy đối chiếu → go-live. |
| b6 | `02` **dòng 51** (A5), **dòng 88** (bảng Phần B), **dòng 93**; `05` **dòng 43** (O6 đã đóng) | Các câu "nhập qua UC-A8 (**import** + golden tests)" | Đổi chữ "import" → "nhập công thức" cho khớp UC-A8 mới. Lưu ý `03` dòng 264 (UC-A6) đã viết "**nhập tay** hoặc import qua UC-A8" — giữ vế nhập tay, bỏ vế import. |

**Điểm cần GIỮ NGUYÊN (và nhấn mạnh hơn):** golden tests (chạy song song Excel ↔ hệ thống, khớp 100% mới go-live, giữ lại chạy hồi quy — `03` dòng 321–333). Khi chuyển sang nhập tay, xác suất lỗi gõ nhầm/hiểu nhầm công thức **tăng**, nên golden tests từ "nghiệm thu import" trở thành **chốt chặn chất lượng chính** của toàn bộ phương án. Tương tự, các tiện ích đã có trong UC-A6 (kiểm tra biến không tồn tại E-1, chạy thử với bộ thông số mẫu 4a) là điều kiện sống còn cho người nhập tay — nên nâng thành yêu cầu bắt buộc, không phải nice-to-have.

## (c) Ngôn ngữ công thức có đủ cho "MỌI công thức" không? — **CHƯA ĐỦ**

Đặc tả hiện tại (`04_ERD.dbml` Project Note dòng 31–40): biến `proj.*`/`comp.*`; toán tử `+ - * / ( )`; so sánh `= != < <= > >=`; logic `and or not`; hàm `if, round(x[,bậc]), ceil, floor, min, max`; select so sánh theo `value`. Đối chiếu với những gì file Excel tính giá thật gần như chắc chắn có, **thiếu các năng lực sau** (xếp theo độ đau):

| # | Năng lực thiếu | Vì sao chắc chắn cần | Đề xuất |
|---|----------------|----------------------|---------|
| c1 | **Bảng tra (lookup) theo khoảng / theo khoá** — tương đương VLOOKUP/INDEX-MATCH: định mức theo ma trận khoảng rộng × khoảng cao, đơn giá bậc thang theo diện tích, hệ số theo dòng sản phẩm. | Excel giá cửa cuốn kiểu gì cũng có ≥1 bảng tra. Mô phỏng ma trận 20×15 bằng `if()` lồng = hàng trăm tầng if — IT/DA nhập tay không thể không sai, không thể review. Chính D10 (`05` dòng 22) cũng đã hứa "bảng tra theo lựa chọn" nhưng đặc tả chưa có. | Thêm thực thể **bảng tra do admin quản lý** (`lookup_table` + `lookup_table_row`, khoá theo giá trị rời rạc hoặc khoảng min–max) + hàm `lookup(bảng, khoá1[, khoá2])`, định nghĩa rõ hành vi out-of-range (lỗi hay lấy biên). |
| c2 | **Biến trung gian / tham chiếu kết quả dòng khác** — kiểu ô Excel tham chiếu ô: "số nan cửa = ceil(cao/0.077)" rồi "số vít = số_nan × 4"; giá trị tính một lần dùng ở nhiều dòng BOM. | Không có nó, mỗi dòng BOM phải lặp lại biểu thức con → nhập tay trùng lặp, sửa 1 chỗ sót N chỗ, sai lệch âm thầm. | Thêm tầng **biến dẫn xuất** (vd `calc.<code>` khai theo sản phẩm, có công thức riêng) hoặc cho dòng BOM tham chiếu `line.<code>.qty`; kèm quy tắc thứ tự tính (DAG) + **chặn tham chiếu vòng** lúc lưu. |
| c3 | **Hằng số dùng chung** (hao hụt %, khối lượng riêng, hệ số an toàn) đang buộc phải hard-code trong từng biểu thức. | Excel để các hệ số này ở 1 ô tham số; đổi hao hụt 5%→6% mà phải sửa 40 công thức là thảm hoạ vận hành. | Bảng hằng số cấu hình (`const.<code>`), ghi audit log như giá. |
| c4 | **Ngữ nghĩa lỗi & NULL chưa đặc tả:** chia cho 0, thông số chưa nhập (nháp dở), option select không khớp nhánh nào, kiểu dữ liệu select-value dùng trong số học (value là varchar — ép kiểu thế nào?). | Người nhập tay cần biết chính xác công thức fail thì hệ thống làm gì (chặn tính? báo dòng nào?) — nếu không, BOM sai lặng lẽ về nhà máy, đúng rủi ro mà D2 sinh ra để chống. | Đặc tả: mọi lỗi runtime → hạng mục không tính được giá, báo đích danh công thức + biến lỗi; quy tắc ép kiểu tường minh. |
| c5 | **Làm tròn theo quy cách vật tư & min bậc thang:** dạng đơn giản biểu diễn được (`ceil(x/6)*6` cho cây 6m, `max(x, 5)` cho tối thiểu 5m²) — chấp nhận được, nhưng nên bổ sung dạng tiện dụng `ceil(x, step)` / `round(x, step)` và ví dụ chuẩn trong đặc tả để N người nhập viết cùng một kiểu. | Giảm biến thể cách viết → dễ review chéo, dễ so golden tests. | Bổ sung hàm bước nhảy + thư viện ví dụ mẫu trong tài liệu admin. |
| c6 | **Dòng BOM có điều kiện tồn tại** (vật tư chỉ xuất hiện khi có điều kiện — vd có ray dẫn hướng khi cao > X): hiện chỉ lách bằng `if(..., qty, 0)`. | Cần chốt: qty = 0 thì dòng có bị loại khỏi phiếu BOM gửi nhà máy không? Chưa đặc tả. | Quy ước: qty ≤ 0 → loại khỏi phiếu; hoặc thêm `applicability_expression` cho dòng BOM. |

Kết luận (c): với đặc tả hiện tại, tuyên bố "hỗ trợ được MỌI công thức" (`02` dòng 22, `01` dòng 18–19) là **quá lời**. Tối thiểu phải có c1 + c2 + c4 trước khi IT/DA bắt đầu nhập bảng tính thật; c3, c5, c6 nên chốt cùng đợt vì đều rẻ.

## (d) Soát lại D1–D10 và O1–O5

| Mã | Đánh giá 03/07 |
|----|----------------|
| D1 | **Vững.** Nhất quán cả 3 file + ERD. |
| D2 | **Vững.** Lưu ý: giá trị của D2 phụ thuộc c4 — điều kiện tương thích lỗi runtime mà "nuốt" lặng lẽ thì SKU sai vẫn lọt. |
| D3, D5 | **Vững.** Phụ thuộc O1 (con số) — đúng như đã khoanh. |
| D4 | **Vững.** Tính lại lúc phát hành + snapshot vĩnh viễn là đúng kỷ luật. |
| D6 | **Vững.** Quy tắc 3 tầng nhất quán ở cả 3 file. |
| D7, D8, D9 | **Vững.** Không có gì mới làm lung lay. |
| D10 | **Đúng một nửa, phải viết lại:** vế "toàn bộ logic trong hệ thống, không Excel-as-engine" giữ nguyên; vế cơ chế "AI-assisted import" thay bằng "IT/DA nhập tay" (danh sách b1–b6). Golden tests giữ và nâng vai trò. Đồng thời D10 phải kéo theo việc nâng cấp ngôn ngữ công thức (c1–c6) thì lời hứa "mọi công thức" mới đứng được. |
| O1–O5 | **Còn đúng và đủ về danh mục.** O1/O2/O3 thuần con số/chính sách — thiết kế đã chừa chỗ đúng. O4/O5 chưa có tiến triển, giữ. |
| Đề xuất **mở OPEN mới (O9?)** | "Chốt phạm vi ngôn ngữ công thức mở rộng (bảng tra, biến dẫn xuất, hằng số, ngữ nghĩa lỗi)" — chờ đối chiếu với bảng tính thật khi công ty bàn giao; đây giờ là rủi ro lớn nhất của toàn phương án, xứng đáng một dòng trong bảng Open. |

**Điểm nghiệp vụ lung lay đáng theo dõi (chưa phải lỗi):**
1. **"Linh/phụ kiện không tác động BOM"** (`02` dòng 51 A5; `04_ERD.dbml` dòng 37): giả định mạnh — thực tế chọn bộ tời/motor khác nhau thường kéo theo vật tư gá lắp/ray khác nhau. Nếu bảng tính thật cho thấy điều ngược lại, phải mở biến `comp.*` (và lựa chọn SKU) vào công thức BOM — nên xác minh **ngay buổi khai thác file Excel đầu tiên**, đừng đợi golden tests fail mới biết.
2. **Nguồn của `quantity` linh kiện** trong công thức giá (`04_ERD.dbml` dòng 44: `Σ(item_component.unit_price × quantity)`): chưa nói rõ số lượng linh kiện do đại lý nhập hay công thức tính — cần một câu chốt trong A3/A6.
3. **Không lưu lịch sử giá vật tư** (MVP): vẫn ổn nhờ snapshot lúc phát hành, nhưng khi IT/DA nhập/sửa công thức thường xuyên ở giai đoạn đầu, `config_audit_log` + golden tests hồi quy là lưới an toàn duy nhất — cần ghi rõ "chạy hồi quy golden tests là bước bắt buộc sau mọi lần sửa công thức/giá" (hiện mới có ở UC-A8 business rule, nên nhắc trong 02 A5).

## Điều kiện để chuyển ỔN

1. Sửa hết b1–b6 (quét sạch "AI-import"/"AI-assisted" theo hướng nhập tay 03/07), cập nhật D10 trong decision log với ngày 03/07.
2. Bổ sung đặc tả ngôn ngữ công thức tối thiểu c1 (bảng tra), c2 (biến dẫn xuất/tham chiếu dòng), c4 (ngữ nghĩa lỗi) vào `04_ERD.dbml` Project Note + phản ánh vào `02` A0 và UC-A6/A8; hoặc hạ tuyên bố "mọi công thức" xuống phạm vi đã đặc tả và mở OPEN mới chờ bảng tính thật.
3. Dọn 2 ghi chú O6 lỗi thời trong `05` (dòng 31, 50) và chốt một bản decision log duy nhất (a3).
