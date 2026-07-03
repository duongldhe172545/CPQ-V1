---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-ADG-CPQ-2026-07-03/prd.md
  - _bmad-output/planning-artifacts/prds/prd-ADG-CPQ-2026-07-03/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-ADG-CPQ-2026-07-03/ARCHITECTURE-SPINE.md
  - docs/ADG_CPQ_UseCase_ChiTiet.md
  - docs/ADG_CPQ_ERD.dbml
  - _bmad-output/planning-artifacts/ADG-CPQ-decision-log.md
---

# ADG CPQ - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for ADG CPQ, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

**F1 — Catalog & cấu hình sản phẩm**

- FR-001: Quản lý cây danh mục nhiều cấp và hồ sơ sản phẩm (SKU duy nhất, vô hiệu hoá thay xoá cứng)
- FR-002: Khai báo bộ thông số dự án riêng theo sản phẩm (kiểu dữ liệu, freeform/dropdown, đơn vị, bắt buộc; option có value dùng trong công thức)
- FR-003: Thêm sản phẩm/thông số mới hoàn toàn bằng cấu hình, không đổi schema/code
- FR-004: Quản lý phụ kiện dùng chung + gán vào sản phẩm (giá có audit)

**F2 — Linh kiện & tương thích**

- FR-010: Quản lý loại linh kiện, bộ thông số của loại, SKU kèm đơn giá + thông số mặc định
- FR-011: Gán loại linh kiện vào sản phẩm; công thức auto-fill theo cặp sản phẩm↔loại
- FR-012: Điều kiện tương thích từng SKU; hệ thống lọc — đại lý chọn [D2]
- FR-013: Công cụ kiểm thử cho admin (bộ thông số mẫu → SKU lọt lọc + kết quả auto-fill; cảnh báo 0 SKU)

**F3 — Vật tư & BOM**

- FR-020: Danh mục vật tư, một đơn giá hiện hành, ẩn với đại lý
- FR-021: Workbook BOM Excel theo sản phẩm [D14]: upload có phiên bản + checksum, kiểm tra hợp đồng INPUT/OUTPUT, một version active [F4]; logic nằm nguyên trong file
- FR-022: Ngôn ngữ công thức (số học, so sánh, logic, if, round/ceil/floor/min/max, proj.*/comp.*) — CHỈ cho auto-fill + tương thích SKU; BOM đi qua workbook [D14]
- FR-023: Lỗi khi tính (workbook lỗi, mã vật tư lạ, thiếu giá, công thức auto-fill lỗi) → dừng, báo đại lý + cảnh báo admin, không ra giá sai âm thầm

**F4 — Định giá**

- FR-030: Tính giá theo hạng mục: cost_material = (OUTPUT workbook × đơn giá hệ thống [OPEN O9]) + linh kiện + phụ kiện → import_price = cost × (1 + gross%); ghi lại version workbook đã dùng
- FR-031: Gross 2 tầng; mức đại lý-theo-sản-phẩm thay thế mức chung
- FR-032: Range min–max theo sản phẩm [D3][O1]; validate min ≤ max
- FR-033: Sàn giá bán ≥ giá nhập mặc định; cờ allow_below_import [O2]
- FR-034: VAT một mức chung [D7]; tổng = Σ thành tiền + VAT
- FR-035: Làm tròn về đồng từng dòng; tổng = cộng dòng đã làm tròn
- FR-036: Dữ liệu mật ẩn ở tầng API với vai dealer

**F5 — Pipeline báo giá**

- FR-040: Tạo báo giá + kho khách riêng đại lý (SĐT unique trong đại lý)
- FR-041: Nhiều hạng mục/báo giá; thêm/xoá/sao chép; số lượng theo hạng mục [D1]
- FR-042: Một ảnh hiện trường/hạng mục (≤8MB, kiểm tra định dạng + độ phân giải, hướng dẫn chụp); trống khi nháp, bắt buộc trước phát hành [F1][D6]
- FR-043: Form thông số sinh từ cấu hình; validate kiểu + bắt buộc
- FR-044: Chọn SKU trong danh sách đã lọc; auto-fill; đại lý chỉnh được (overridden)
- FR-045: Tick phụ kiện (SL luôn 1/hạng mục)
- FR-046: Chọn giá bán từng hạng mục trong range; xem giá nhập/thành tiền/tổng/VAT realtime
- FR-047: Lưu nháp mọi bước, khôi phục đúng chỗ; sửa ngược → tính lại phần phụ thuộc, SKU mất tương thích buộc chọn lại
- FR-048: Tìm kiếm báo giá + sao chép làm nháp mới

**F6 — Ảnh mock-up AI (trong MVP)**

- FR-050: Sinh mock-up theo hạng mục qua dịch vụ AI; tối đa N lần (cấu hình); lỗi kỹ thuật không tính lượt [AD-14]
- FR-051: Quy tắc 3 tầng [D6]: mockup_required; hết lượt → thay ảnh (reset) hoặc phát hành không ảnh + lý do bắt buộc có log
- FR-052: Duyệt tối đa 1 ảnh/hạng mục đính PDF

**F7 — Phát hành & gửi**

- FR-060: Phát hành = tính lại toàn bộ [D4]; giá lệch → đại lý xác nhận lại
- FR-061: Snapshot đóng băng vĩnh viễn (khách [F2], sản phẩm [F3], thông số, BOM, giá)
- FR-062: PDF nhiều hạng mục + mock-up + tổng/VAT + hạn hiệu lực [O3]
- FR-063: Gửi khách: tải PDF thủ công (bắt buộc); Zalo OA tự động (should-have); retry
- FR-064: Phiếu BOM sản xuất gộp hạng mục × SL; MVP xuất file chuẩn; kênh tự động [O4]; retry
- FR-065: Báo giá đã phát hành chỉ đọc; sửa = sao chép

**F8 — Quản trị, phân quyền & audit**

- FR-070: Tài khoản 2 vai (admin/dealer); nhiều tài khoản/đại lý dùng chung dữ liệu
- FR-071: Hồ sơ đại lý (onboard, gross mặc định + theo sản phẩm, vô hiệu hoá)
- FR-072: Đơn vị tính + system_setting (VAT, lượt gen, mockup_required, allow_below_import, validity_days)
- FR-073: Audit log mọi thay đổi cấu hình giá/công thức; chỉ đọc
- FR-074: Số liệu vận hành tối thiểu cho G1–G4

**F9 — Nghiệm thu workbook BOM (trong MVP)**

- FR-090: Nghiệm thu workbook bằng bộ ca mẫu [D14]: input + kết quả kỳ vọng của người làm giá, khớp 100% mới active/go-live; file Excel là nguồn chân lý logic BOM, hệ thống giữ phiên bản + checksum + audit
- FR-091: Bộ ca mẫu lưu theo sản phẩm, tự chạy lại khi upload version mới — chặn active nếu lệch

### NonFunctional Requirements

- NFR-01: Ẩn dữ liệu mật thực thi tầng API (field-level theo vai); e2e test ca "dealer cố đọc giá vốn"
- NFR-02: Phiên an toàn, mật khẩu băm, HTTPS, tuân thủ Nghị định 13/2023 (PII khách)
- NFR-03: Mobile-first cho pipeline đại lý; admin desktop-first [D8]
- NFR-04: Web responsive [D11]; frontend bọc được vào Zalo Mini App Phase 2 không viết lại
- NFR-05: ≥300 tài khoản, ~50 đồng thời, ~5.000 báo giá/tháng
- NFR-06: Tính BOM+giá ≤3s; mock-up AI ≤60s có tiến trình
- NFR-07: Gửi thất bại không mất dữ liệu, không chặn phát hành, luôn gửi lại được
- NFR-08: Toàn bộ UI/PDF/thông báo tiếng Việt; VND
- NFR-09: Báo giá phát hành bất biến; thay đổi cấu hình có audit trail
- NFR-10: Đồng thời sửa nháp: khoá lạc quan + cảnh báo (DRAFT_CONFLICT)

### Additional Requirements

Từ Architecture Spine (16 AD — ràng buộc mọi story, xem ARCHITECTURE-SPINE.md):

- **Không có starter template bên ngoài** — Epic đầu tiên phải scaffold monorepo pnpm theo Structural Seed: `apps/api` (NestJS 11), `apps/web` (Next.js 16), `packages/shared` (zod), `packages/formula-engine`, docker-compose (PostgreSQL 18, Valkey 8, object storage), pin Node 24 LTS + pnpm 11
- AD-1: modular monolith, mỗi bảng một module chủ (nhóm quote_* thuộc module quote)
- AD-2/AD-3: tính giá chỉ ở server; DTO whitelist theo vai + e2e test bảo mật
- AD-4: formula-engine là package riêng, parser AST, không eval — phạm vi CHỈ auto-fill + tương thích (thu hẹp theo D14)
- AD-17/D14: BOM tính bằng workbook Excel qua adapter `BomWorkbookEngine` duy nhất (INPUT named ranges → OUTPUT bảng chuẩn); engine cụ thể chọn bằng story spike; macro không hỗ trợ
- AD-5: INPUT vs DERIVED; publish = 1 transaction; closed-list field sau submitted
- AD-6: tiền integer VND, BIGINT physical, roundVnd() duy nhất
- AD-7: việc chậm qua BullMQ, job idempotent, polling status
- AD-8: dealer scoping từ token qua guard chung
- AD-9: audit qua interceptor chung
- AD-10: REST /api/v1, error envelope chuẩn, zod shared
- AD-11: không SDK Zalo ở lõi; auth cookie httpOnly
- AD-12: Prisma schema sinh từ ERD dbml; migrations là nguồn physical
- AD-13: AI/Zalo/nhà máy qua port/adapter (mock/manual trước)
- AD-14: lượt gen mock-up đếm bằng dòng có status, giữ chỗ khi enqueue
- AD-15: PII có ranh giới; Argon2id; không log PII/giá mật
- AD-16: khoá lạc quan updated_at cho nháp
- Seed dữ liệu ban đầu: import CSV/Excel (vật tư+giá, SKU, ~300 đại lý+gross, admin bootstrap) — module import-tools, trước pilot
- Thứ tự build (addendum PRD): F8 → F1–F3 → F4 → F5 → F7 → F6 → F9; golden-test harness dựng sớm

### UX Design Requirements

Chưa có tài liệu UX riêng (bmad-ux chưa chạy). Yêu cầu UX rút từ PRD/addendum, áp vào story liên quan:

- UX-DR1: Pipeline đại lý thao tác một tay trên điện thoại ngoài trời; bước chụp ảnh có overlay hướng dẫn khung hình
- UX-DR2: "Sao chép hạng mục" là thao tác tần suất cao — đặt nổi
- UX-DR3: Màn chọn giá bán hiển thị giá nhập + range có chặn; không render field mật rồi giấu bằng CSS
- UX-DR4: Màn công thức admin có bộ kiểm thử nhúng, báo lỗi bằng ngôn ngữ nghiệp vụ
- UX-DR5: Mock-up AI có hiển thị tiến trình (≤60s), trạng thái lượt gen còn lại

### FR Coverage Map

| FR | Epic | Ghi chú |
|---|---|---|
| FR-070, FR-071, FR-072, FR-073 | Epic 1 | Nền tảng, tài khoản, đại lý, cấu hình, audit |
| FR-001, FR-002, FR-003, FR-004 | Epic 2 | Catalog & thông số & phụ kiện |
| FR-010, FR-011, FR-012, FR-013 | Epic 3 | Linh kiện + tương thích + formula engine |
| FR-020, FR-021, FR-022, FR-023 | Epic 3 | Vật tư + BOM template |
| FR-030..FR-036 | Epic 4 | Định giá + ẩn dữ liệu mật |
| FR-040..FR-048 | Epic 5 | Pipeline báo giá đại lý |
| FR-060..FR-065 | Epic 6 | Phát hành, PDF, gửi khách/nhà máy |
| FR-050, FR-051, FR-052 | Epic 7 | Mock-up AI |
| FR-090, FR-091, FR-074 | Epic 8 | Nghiệm thu workbook, seed, số liệu vận hành |

Mọi FR của PRD đều có epic; không FR nào bị bỏ rơi.

## Epic List

### Epic 1: Nền tảng hệ thống & quản trị truy cập
Admin đăng nhập được vào hệ thống chạy bằng Docker, tạo đại lý và tài khoản (2 vai), đặt cấu hình hệ thống (VAT, lượt gen, cờ chính sách), và mọi thay đổi cấu hình từ đây về sau đều có vết audit. Bao gồm scaffold monorepo theo Structural Seed của spine (không dùng starter ngoài).
**FRs covered:** FR-070, FR-071, FR-072, FR-073

### Epic 2: Catalog sản phẩm cấu hình được
Admin tự dựng cây danh mục, sản phẩm, bộ thông số dự án (kèm dropdown option) và phụ kiện — thêm sản phẩm mới hoàn toàn bằng dữ liệu, không cần dev.
**FRs covered:** FR-001, FR-002, FR-003, FR-004

### Epic 3: Linh kiện, vật tư & workbook BOM
Admin khai loại/SKU linh kiện, công thức auto-fill, điều kiện tương thích, vật tư, và **upload workbook BOM Excel** (Excel-as-engine [D14]) có phiên bản + chạy thử tại chỗ. Hai nền tảng: package `formula-engine` (auto-fill/tương thích — AD-4) và adapter `BomWorkbookEngine` (AD-17, chọn engine bằng spike).
**FRs covered:** FR-010, FR-011, FR-012, FR-013, FR-020, FR-021, FR-022, FR-023

### Epic 4: Bộ máy định giá & che dữ liệu mật
Admin đặt gross (2 tầng) và range min–max; hệ thống tính cost → giá nhập cho một bộ cấu hình bất kỳ qua đúng một hàm định giá (AD-5), tiền nguyên đồng (AD-6); dữ liệu mật bị chặn ở tầng API có e2e test chứng minh (AD-3).
**FRs covered:** FR-030, FR-031, FR-032, FR-033, FR-034, FR-035, FR-036

### Epic 5: Pipeline báo giá của đại lý
Đại lý (trên điện thoại) tạo báo giá nhiều hạng mục: khách hàng → sản phẩm → ảnh hiện trường → thông số → linh kiện đã lọc tương thích → phụ kiện → xem giá nhập & chọn giá bán trong range; lưu nháp mọi bước, sao chép hạng mục/báo giá, khoá lạc quan khi sửa đồng thời.
**FRs covered:** FR-040, FR-041, FR-042, FR-043, FR-044, FR-045, FR-046, FR-047, FR-048

### Epic 6: Phát hành báo giá & gửi đi
Đại lý bấm phát hành: hệ thống tính lại, chốt snapshot bất biến, sinh PDF nhiều hạng mục có hạn hiệu lực, cho tải PDF/gửi Zalo OA, xuất phiếu BOM sản xuất; kênh nào lỗi cũng gửi lại được.
**FRs covered:** FR-060, FR-061, FR-062, FR-063, FR-064, FR-065

### Epic 7: Ảnh mock-up AI
Đại lý sinh ảnh "nhà khách sau khi lắp cửa" cho từng hạng mục qua hàng đợi (adapter provider — AD-13/14), duyệt ảnh đính vào PDF; quy tắc 3 tầng không gây tắc phát hành.
**FRs covered:** FR-050, FR-051, FR-052

### Epic 8: Sẵn sàng vận hành: nghiệm thu workbook, seed & số liệu
Admin nghiệm thu workbook BOM thật bằng bộ ca mẫu (khớp 100% mới go-live, tự chạy hồi quy khi upload version mới); import seed dữ liệu ban đầu (vật tư, SKU, đại lý); hệ thống đếm được các chỉ số G1–G4.
**FRs covered:** FR-090, FR-091, FR-074

**Chuỗi phụ thuộc:** 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 (mỗi epic đứng được một mình trên nền các epic trước; Epic 7 sau Epic 6 vì phát hành có đường không-ảnh theo D6).

## Epic 1: Nền tảng hệ thống & quản trị truy cập

Admin đăng nhập được vào hệ thống chạy bằng Docker, tạo đại lý và tài khoản, đặt cấu hình hệ thống; mọi thay đổi cấu hình có vết audit. Mọi story tuân thủ ARCHITECTURE-SPINE (AD-1..16).

### Story 1.1: Scaffold monorepo & môi trường chạy được

As a dev/agent,
I want khung dự án dựng đúng Structural Seed của spine và chạy được bằng một lệnh,
So that mọi story sau có nền nhất quán để build.

**Acceptance Criteria:**

**Given** máy có Docker + Node 24 + pnpm 11
**When** chạy `pnpm install` và `docker compose up`
**Then** PostgreSQL 18, Valkey, object storage khởi động; `apps/api` (NestJS 11) trả `GET /api/v1/health` = 200; `apps/web` (Next.js 16) hiển thị trang chờ tiếng Việt
**And** monorepo có `packages/shared` (zod + error envelope AD-10) và `packages/formula-engine` (stub + 1 test chạy qua), Prisma kết nối DB, env validate bằng zod lúc boot, `.nvmrc` + `packageManager` được pin.

**Given** một request gây lỗi bất kỳ
**When** API trả lỗi
**Then** body đúng envelope `{ error: { code, message, details? } }` với message tiếng Việt.

### Story 1.2: Đăng nhập & phiên làm việc

As a người dùng (admin hoặc đại lý),
I want đăng nhập bằng email/mật khẩu và giữ phiên an toàn,
So that chỉ người có quyền mới vào được hệ thống.

**Acceptance Criteria:**

**Given** bảng `user_account` (Prisma migration đầu tiên) và một admin bootstrap tạo từ script seed
**When** đăng nhập đúng thông tin
**Then** nhận cookie phiên httpOnly (AD-11), response không chứa `password_hash`; đăng nhập sai trả `AUTH_INVALID_CREDENTIALS`
**And** mật khẩu băm Argon2id (AD-15), không xuất hiện trong log.

**Given** người dùng đã đăng nhập
**When** gọi API có gắn role guard (admin-only hoặc dealer-only)
**Then** sai vai bị chặn 403 với code `FORBIDDEN_ROLE`.

### Story 1.3: Quản lý hồ sơ đại lý

As a admin,
I want tạo/sửa/vô hiệu hoá đại lý và đặt gross mặc định,
So that mạng lưới đại lý được onboard vào hệ thống.

**Acceptance Criteria:**

**Given** màn hình admin "Đại lý"
**When** tạo đại lý với mã, tên, SĐT, `default_gross_percent`
**Then** đại lý được lưu (mã unique — trùng trả `DEALER_CODE_EXISTS`); danh sách lọc theo trạng thái
**And** vô hiệu hoá đại lý → tài khoản thuộc đại lý không đăng nhập được, dữ liệu cũ giữ nguyên.

### Story 1.4: Quản lý tài khoản & dealer scoping

As a admin,
I want tạo tài khoản 2 vai, gắn tài khoản dealer vào đại lý, khoá/mở,
So that đúng người vào đúng phạm vi dữ liệu.

**Acceptance Criteria:**

**Given** đại lý đã tồn tại
**When** tạo tài khoản vai `dealer` mà không chọn đại lý
**Then** bị chặn validate; email trùng trả `EMAIL_EXISTS`
**And** guard dealer-scope chung (AD-8) được cài: mọi endpoint dữ liệu-theo-đại-lý lấy `dealer_id` từ phiên, e2e chứng minh tài khoản đại lý A không đọc được dữ liệu đại lý B (fixture 2 đại lý).

### Story 1.5: Đơn vị tính & cấu hình hệ thống

As a admin,
I want quản lý đơn vị tính và các cấu hình hệ thống theo key chuẩn,
So that VAT, lượt gen AI, cờ chính sách điều chỉnh được không cần dev.

**Acceptance Criteria:**

**Given** màn hình admin "Cấu hình"
**When** sửa các key `vat_percent`, `ai_max_regenerate`, `mockup_required`, `allow_below_import`, `quote_validity_days`
**Then** giá trị được validate đúng kiểu từng key (sai kiểu trả `SETTING_INVALID_VALUE`); `quote_validity_days` cho phép để trống
**And** đơn vị tính CRUD với `code` unique; xoá đơn vị đang được tham chiếu bị chặn.

### Story 1.6: Nhật ký thay đổi cấu hình (audit)

As a admin,
I want mọi thay đổi cấu hình nhạy cảm tự ghi vết và xem lại được,
So that truy được "ai đổi gì lúc nào" khi có khiếu nại giá.

**Acceptance Criteria:**

**Given** interceptor audit chung (AD-9) đã đăng ký cho `system_setting` và `dealer` (gross)
**When** admin sửa một giá trị
**Then** một dòng `config_audit_log` được ghi (bảng, id, action, diff tóm tắt, user, thời điểm)
**And** màn hình "Nhật ký" lọc theo bảng/người/khoảng thời gian, chỉ đọc; các bảng cấu hình ở epic sau chỉ việc đăng ký vào interceptor này.

## Epic 2: Catalog sản phẩm cấu hình được

Admin tự dựng danh mục, sản phẩm, thông số dự án và phụ kiện — thêm sản phẩm mới hoàn toàn bằng dữ liệu (FR-003 là nguyên tắc xuyên suốt: các story dưới không hard-code gì theo sản phẩm cụ thể).

### Story 2.1: Cây danh mục

As a admin,
I want quản lý cây danh mục nhiều cấp,
So that sản phẩm được tổ chức đúng cấu trúc kinh doanh.

**Acceptance Criteria:**

**Given** màn hình "Danh mục"
**When** tạo danh mục con của một danh mục cha, sắp thứ tự
**Then** cây hiển thị đúng cấp; xoá danh mục còn con hoặc còn sản phẩm bị chặn với thông báo đề nghị chuyển/vô hiệu hoá.

### Story 2.2: Hồ sơ sản phẩm

As a admin,
I want tạo/sửa/vô hiệu hoá sản phẩm với SKU duy nhất và thumbnail,
So that đại lý có sản phẩm để báo giá.

**Acceptance Criteria:**

**Given** danh mục đã có
**When** tạo sản phẩm (tên, SKU, danh mục, mô tả, thumbnail upload lên object storage)
**Then** SKU trùng trả `PRODUCT_SKU_EXISTS`; ảnh validate định dạng/dung lượng
**And** vô hiệu hoá → sản phẩm biến mất khỏi mọi danh sách phía đại lý nhưng dữ liệu cũ giữ nguyên.

### Story 2.3: Thông số dự án theo sản phẩm

As a admin,
I want khai bộ thông số dự án riêng cho từng sản phẩm (kiểu dữ liệu, freeform/dropdown, đơn vị, bắt buộc),
So that phiếu nhập liệu của đại lý sinh tự động từ cấu hình.

**Acceptance Criteria:**

**Given** sản phẩm đã có
**When** thêm thông số với `code`, kiểu `number|text|boolean|date|select`, kiểu nhập, đơn vị, bắt buộc; thêm option (value, label) cho dropdown
**Then** `code` trùng trong cùng sản phẩm trả `PARAM_CODE_EXISTS`; option `value` là giá trị dùng trong công thức
**And** đổi/xoá `code` đang được công thức tham chiếu (khi Epic 3 tồn tại) hiện cảnh báo liệt kê công thức ảnh hưởng, đòi xác nhận.

### Story 2.4: Phụ kiện & gán vào sản phẩm

As a admin,
I want quản lý phụ kiện dùng chung và gán vào sản phẩm,
So that đại lý tick chọn được phụ kiện ở báo giá.

**Acceptance Criteria:**

**Given** màn hình "Phụ kiện"
**When** tạo phụ kiện (tên, thumbnail, đơn giá) và gán vào một hoặc nhiều sản phẩm
**Then** thay đổi đơn giá ghi audit (đăng ký interceptor 1.6); gán trùng bị bỏ qua êm
**And** gỡ gán không ảnh hưởng dữ liệu báo giá cũ (khi có).

## Epic 3: Linh kiện, vật tư & workbook BOM

Admin cấu hình đủ đầu vào để tính vật tư: formula engine cho auto-fill/tương thích (AD-4), linh kiện + tương thích [D2], vật tư + workbook BOM Excel-as-engine (AD-17, [D14]).

### Story 3.1: Formula engine — parser công thức

As a admin (người hưởng),
I want một engine công thức duy nhất diễn giải đúng đặc tả (số học, so sánh, logic, if, round/ceil/floor/min/max, biến proj.*/comp.*),
So that auto-fill thông số linh kiện và điều kiện tương thích SKU cùng một semantics (BOM không dùng engine này — đi qua workbook [D14]).

**Acceptance Criteria:**

**Given** package `packages/formula-engine`
**When** evaluate biểu thức hợp lệ với map biến
**Then** trả đúng giá trị theo đặc tả ERD Project Note (bộ unit test phủ: 4 phép tính, so sánh chuỗi select `if(proj.mau = "tinh_dien", 1.2, 1.0)`, chia 0 → typedError `DIV_BY_ZERO`, biến thiếu → `UNKNOWN_VAR`, hàm round/ceil/floor/min/max)
**And** engine là pure function, không I/O, không eval; kèm `roundVnd()` (AD-6) có test làm tròn về đồng.

### Story 3.2: Loại linh kiện & bộ thông số của loại

As a admin,
I want khai loại linh kiện dùng chung và bộ thông số của loại,
So that SKU linh kiện có khung dữ liệu chuẩn.

**Acceptance Criteria:**

**Given** màn hình "Linh kiện"
**When** tạo `component_type` (code unique) và các `component_param_def` (code unique trong loại, kiểu, đơn vị, bắt buộc, option cho dropdown)
**Then** lưu thành công; xoá loại đang gán vào sản phẩm (khi 3.4 tồn tại) bị chặn.

### Story 3.3: SKU linh kiện & thông số mặc định

As a admin,
I want tạo SKU linh kiện với đơn giá và giá trị thông số mặc định,
So that có nguồn biến comp.* và giá cho tính toán.

**Acceptance Criteria:**

**Given** loại linh kiện đã có
**When** tạo SKU (mã unique, tên, đơn giá, đơn vị) và nhập `component_sku_param_value` theo từng thông số
**Then** thay đổi đơn giá ghi audit; thông số không áp dụng được để trống
**And** đơn giá không xuất hiện trong bất kỳ response nào cho vai dealer (chuẩn bị cho AD-3; kiểm bằng e2e ở 4.3).

### Story 3.4: Gán linh kiện vào sản phẩm & công thức auto-fill

As a admin,
I want gán loại linh kiện vào sản phẩm và khai công thức auto-fill theo cặp sản phẩm↔loại,
So that thông số linh kiện tự điền theo thông số dự án.

**Acceptance Criteria:**

**Given** sản phẩm (Epic 2) + loại linh kiện (3.2) + formula engine (3.1)
**When** gán loại (bắt buộc/không, thứ tự) và nhập `formula_expression` cho thông số cần auto-fill
**Then** lưu chỉ khi validate cú pháp + mọi biến `proj.*` tồn tại trong thông số của sản phẩm và `comp.*` trong thông số của loại (sai trả `FORMULA_UNKNOWN_VAR` chỉ đích danh biến)
**And** thông số không có công thức được đánh dấu "nhập tay".

### Story 3.5: Điều kiện tương thích SKU

As a admin,
I want khai điều kiện tương thích cho từng SKU theo cặp sản phẩm↔loại,
So that đại lý chỉ thấy SKU hợp lệ với thông số đã nhập [D2].

**Acceptance Criteria:**

**Given** SKU thuộc loại đã gán vào sản phẩm
**When** khai `condition_expression` (trả boolean) hoặc để trống (= luôn khả dụng cho sản phẩm này)
**Then** SKU không được khai dòng nào = không khả dụng cho sản phẩm; validate biểu thức như 3.4
**And** service lọc `getCompatibleSkus(productId, componentTypeId, params)` trả đúng danh sách theo bộ test 3 kịch bản (tất cả lọt / một phần / không SKU nào).

### Story 3.6: Công cụ kiểm thử cấu hình cho admin

As a admin,
I want nhập bộ thông số mẫu và xem SKU lọt bộ lọc + kết quả auto-fill,
So that phát hiện lỗi cấu hình trước khi đại lý gặp.

**Acceptance Criteria:**

**Given** sản phẩm đã cấu hình 3.4 + 3.5
**When** nhập bộ giá trị thông số mẫu ở tab "Kiểm thử"
**Then** hiển thị mỗi loại linh kiện: danh sách SKU lọt lọc + giá trị auto-fill từng thông số; lỗi công thức hiển thị bằng ngôn ngữ nghiệp vụ (UX-DR4)
**And** tổ hợp cho ra 0 SKU ở loại bắt buộc hiển thị cảnh báo đỏ "đại lý sẽ kẹt ở tổ hợp này".

### Story 3.7: Danh mục vật tư

As a admin,
I want quản lý vật tư với một đơn giá hiện hành,
So that BOM có nguồn giá để tính giá vốn.

**Acceptance Criteria:**

**Given** màn hình "Vật tư"
**When** tạo/sửa vật tư (code unique, tên, đơn vị, đơn giá)
**Then** đổi đơn giá ghi audit; vô hiệu hoá thay xoá cứng
**And** mọi field giá vật tư nằm ngoài DTO vai dealer.

### Story 3.8: Spike + adapter engine tính workbook Excel

As a hệ thống,
I want một adapter `BomWorkbookEngine` duy nhất tính được file Excel server-side (đã chọn engine qua spike),
So that BOM tính bằng chính file của công ty, không tái hiện logic (AD-17, [D14]).

**Acceptance Criteria:**

**Given** ba ứng viên engine (LibreOffice headless, HyperFormula, sidecar Python `formulas`) và một workbook mẫu chuẩn hoá (INPUT named ranges, OUTPUT bảng chuẩn) có công thức đại diện (IF, VLOOKUP, ROUND, bảng tra)
**When** chạy spike so sánh: độ phủ hàm, thời gian tính, độ phức tạp vận hành
**Then** một engine được chọn kèm biên bản so sánh ghi vào addendum; adapter `BomWorkbookEngine(workbookFile, inputs) → [{material_code, quantity}] | typedError` hoạt động với workbook mẫu
**And** engine chạy được trong worker; macro/VBA bị từ chối rõ ràng với mã lỗi `WORKBOOK_UNSUPPORTED`; kết quả tính ≤ 3s cho workbook mẫu (NFR-06).

### Story 3.9: Quản lý workbook BOM — upload, hợp đồng & phiên bản

As a admin,
I want upload workbook BOM cho sản phẩm, hệ thống kiểm tra hợp đồng và quản lý phiên bản,
So that logic BOM của công ty vào hệ thống có kiểm soát (FR-021).

**Acceptance Criteria:**

**Given** sản phẩm + thông số (2.3) + vật tư (3.7) + adapter engine (3.8)
**When** upload file `.xlsx`
**Then** hệ thống tạo `product_bom_workbook` version mới (checksum, người upload, audit) và **kiểm tra hợp đồng**: INPUT có đủ named range khớp `code` thông số của sản phẩm; OUTPUT đúng bảng chuẩn; mọi `material_code` tồn tại trong danh mục vật tư — lỗi nào chỉ đích danh lỗi đó
**And** màn "chạy thử": nhập bộ thông số mẫu → hiển thị OUTPUT (vật tư + số lượng) + giá vốn NVL ước tính; version chỉ active được sau nghiệm thu (8.1), active mới tự bỏ active cũ (F4).

## Epic 4: Bộ máy định giá & che dữ liệu mật

Từ cấu hình ra giá nhập chuẩn qua một hàm duy nhất; dữ liệu mật bị chặn ở tầng API có bằng chứng test.

### Story 4.1: Cấu hình giá — gross & range min–max

As a admin,
I want đặt global gross + range min–max theo sản phẩm và gross riêng theo cặp đại lý–sản phẩm,
So that chính sách giá của công ty nằm trong hệ thống [D3].

**Acceptance Criteria:**

**Given** màn hình "Cấu hình giá"
**When** đặt `global_gross_percent`, `min_selling_price`, `max_selling_price` (đều nullable) và `dealer_product_gross`
**Then** min > max trả `PRICE_RANGE_INVALID`; mọi thay đổi ghi audit
**And** resolve gross đúng quy tắc: gross đại lý-theo-sản-phẩm **thay thế** default của đại lý (unit test 3 tổ hợp).

### Story 4.2: Hàm định giá duy nhất

As a hệ thống,
I want một hàm định giá tính cost_material → import_price cho một cấu hình hạng mục bất kỳ,
So that mọi nơi cần giá đều gọi đúng một đường (AD-5).

**Acceptance Criteria:**

**Given** cấu hình đầy đủ (workbook BOM version active [D14], linh kiện + số lượng, phụ kiện, gross, thông số)
**When** gọi `priceItem(config)` trong module `pricing`
**Then** trả breakdown: BOM lines (số lượng từ OUTPUT workbook qua adapter 3.8 × đơn giá `material` [OPEN O9]), tiền linh kiện, tiền phụ kiện, `cost_material`, `applied_gross_percent`, `import_price` — mọi số tiền qua `roundVnd()` từng dòng, kèm `priced_at` + `bom_workbook_version` đã dùng
**And** công thức lỗi/vật tư thiếu giá → typedError có mã (`FORMULA_ERROR`, `MATERIAL_PRICE_MISSING`), không trả giá sai âm thầm (FR-023); admin có màn "chạy thử giá" dùng chính hàm này; unit test đối chiếu một ca tính tay đầy đủ.

### Story 4.3: Chặn dữ liệu mật tầng API — có bằng chứng

As a công ty,
I want vai dealer không bao giờ nhận được field mật qua API,
So that giá vốn/gross tuyệt mật kể cả với người biết kỹ thuật (NFR-01).

**Acceptance Criteria:**

**Given** DTO whitelist cho vai dealer (AD-3) áp lên mọi endpoint hiện có
**When** chạy bộ e2e "dealer soi API": đăng nhập dealer, gọi mọi endpoint trả về dữ liệu giá/cấu hình
**Then** JSON không chứa các khoá: `cost_material`, `applied_gross_percent`, `unit_price` (vật tư), `global_gross_percent`, `gross_percent`, BOM chi tiết — kể cả giá trị null; danh sách khoá cấm nằm ở một constant dùng chung cho test
**And** bộ test này chạy trong CI, fail = chặn merge.

## Epic 5: Pipeline báo giá của đại lý

Đại lý trên điện thoại tạo báo giá nhiều hạng mục end-to-end đến chọn giá bán; mobile-first (NFR-03, UX-DR1..3).

### Story 5.1: Khách hàng & khởi tạo báo giá

As a đại lý,
I want tạo báo giá và gắn khách (tra SĐT từ kho khách riêng của tôi),
So that bắt đầu pipeline ngay tại công trình.

**Acceptance Criteria:**

**Given** đại lý đã đăng nhập
**When** nhập SĐT khách
**Then** khách cũ tự nạp thông tin cho xác nhận/sửa; khách mới nhập tên + địa chỉ (số nhà, đường, phường/xã, tỉnh); `unique(dealer_id, phone)` — mọi truy vấn qua guard dealer-scope (AD-8)
**And** quote tạo ở `status=draft`, `current_phase=customer`, chưa đòi hỏi ảnh (F1).

### Story 5.2: Quản lý hạng mục

As a đại lý,
I want thêm/xoá/sao chép hạng mục và đặt số lượng,
So that một báo giá phủ cả đơn hàng nhiều cửa [D1].

**Acceptance Criteria:**

**Given** báo giá draft có khách
**When** thêm hạng mục, chọn sản phẩm (chỉ hiện sản phẩm active + cấu hình đủ: BOM active + pricing config), nhập số lượng
**Then** `quote_item` tạo với `line_no` tăng, `step=product→photo`; đổi sản phẩm khi đã có dữ liệu bước sau → cảnh báo xoá dữ liệu, đòi xác nhận
**And** "Sao chép hạng mục" nhân bản cấu hình (trừ ảnh hiện trường) và đặt nổi trên UI (UX-DR2); xoá hạng mục xoá dữ liệu con.

### Story 5.3: Ảnh hiện trường theo hạng mục

As a đại lý,
I want chụp/tải một ảnh hiện trường cho từng hạng mục theo hướng dẫn,
So that có đầu vào cho mock-up và hồ sơ báo giá.

**Acceptance Criteria:**

**Given** hạng mục ở bước `photo`
**When** tải ảnh
**Then** validate định dạng + ≤8MB + độ phân giải tối thiểu (lỗi chỉ rõ nguyên nhân); lưu object storage; set `field_photo_updated_at` [AD-14]
**And** màn chụp có overlay hướng dẫn khung hình (UX-DR1); cho "bỏ qua tạm" đi tiếp các bước khác nhưng hạng mục không thể phát hành khi thiếu ảnh (F1); thay ảnh khi đã có mock-up → cảnh báo reset lượt gen.

### Story 5.4: Nhập thông số dự án

As a đại lý,
I want điền thông số theo form sinh từ cấu hình sản phẩm,
So that hệ thống có biến proj.* để tính mọi thứ.

**Acceptance Criteria:**

**Given** hạng mục đã chọn sản phẩm
**When** điền form (freeform validate đúng kiểu; dropdown chọn từ option)
**Then** thiếu thông số bắt buộc chặn chuyển bước, chỉ rõ trường; giá trị lưu `quote_item_param` kèm snapshot code/tên/đơn vị
**And** sửa thông số sau khi đã có linh kiện → chạy lại lọc tương thích + auto-fill, SKU đã chọn không còn hợp lệ bị đánh dấu buộc chọn lại (FR-047).

### Story 5.5: Chọn linh kiện đã lọc tương thích

As a đại lý,
I want chọn SKU trong danh sách đã lọc theo thông số và chỉnh giá trị auto-fill khi cần,
So that không thể chọn sai linh kiện [D2].

**Acceptance Criteria:**

**Given** thông số đã đủ
**When** mở bước linh kiện
**Then** mỗi loại chỉ hiện SKU vượt bộ lọc (service 3.5, chạy server — AD-2); auto-fill điền giá trị từ công thức 3.4; đại lý chỉnh → `is_overridden=true`
**And** loại bắt buộc chưa chọn chặn chuyển bước; bộ lọc ra 0 SKU → thông báo đại lý + bắn cảnh báo admin (`SKU_INCOMPATIBLE` log); công thức lỗi runtime → để trống cho nhập tay + cảnh báo admin.

### Story 5.6: Phụ kiện

As a đại lý,
I want tick chọn phụ kiện của sản phẩm,
So that báo giá đủ hạng mục đi kèm.

**Acceptance Criteria:**

**Given** hạng mục qua bước linh kiện
**When** tick/bỏ tick phụ kiện khả dụng của sản phẩm
**Then** mỗi phụ kiện được chọn là một dòng `quote_item_accessory` snapshot tên + đơn giá, số lượng luôn 1; không chọn gì vẫn đi tiếp; hạng mục sang `done`.

### Story 5.7: Xem giá & chọn giá bán

As a đại lý,
I want xem giá nhập từng hạng mục và chọn giá bán trong khoảng cho phép,
So that chốt được tổng báo giá gửi khách.

**Acceptance Criteria:**

**Given** mọi hạng mục `done`
**When** mở bước giá
**Then** từng hạng mục hiển thị `import_price` (tính qua hàm 4.2) + range min–max; nhập giá ngoài range trả `PRICE_OUT_OF_RANGE` kèm khoảng hợp lệ; dưới giá nhập bị chặn trừ khi `allow_below_import` bật [O2]
**And** `line_total`, `subtotal`, VAT, `total` cập nhật realtime từ server (AD-2); UI không render bất kỳ field mật nào (UX-DR3); range trống = chỉ áp sàn giá nhập.

### Story 5.8: Lưu nháp, khôi phục & khoá lạc quan

As a đại lý,
I want thoát ra bất kỳ lúc nào và quay lại đúng chỗ đang dở, không bị đồng nghiệp ghi đè,
So that làm báo giá linh hoạt ngoài công trình.

**Acceptance Criteria:**

**Given** báo giá draft đang dở
**When** mở lại từ danh sách
**Then** khôi phục đúng `current_phase` + `step` từng hạng mục
**And** hai phiên cùng sửa một nháp: phiên ghi sau với `updated_at` cũ nhận `DRAFT_CONFLICT` 409 và được nhắc tải lại (AD-16); sửa bước trước làm mất hiệu lực kết quả bước sau của hạng mục đó (đánh dấu cần tính lại).

### Story 5.9: Danh sách, tìm kiếm & sao chép báo giá

As a đại lý,
I want tìm báo giá theo SĐT khách/mã/trạng thái/ngày và sao chép báo giá cũ,
So that tái sử dụng thay vì nhập lại từ đầu.

**Acceptance Criteria:**

**Given** kho báo giá của đại lý (chỉ của đại lý mình — AD-8)
**When** tìm kiếm và bấm "Sao chép" trên một báo giá
**Then** tạo draft mới copy hạng mục + cấu hình, KHÔNG copy ảnh hiện trường/mock-up/số giá đã chốt; draft mới đi lại pipeline và tính giá theo cấu hình hiện hành [D4].

## Epic 6: Phát hành báo giá & gửi đi

Phát hành an toàn tuyệt đối về số liệu; PDF chuyên nghiệp; hai kênh gửi có retry.

### Story 6.1: Transaction phát hành & snapshot bất biến

As a đại lý,
I want bấm phát hành và hệ thống chốt số liệu đúng cấu hình mới nhất,
So that báo giá gửi khách không bao giờ sai giá [D4].

**Acceptance Criteria:**

**Given** báo giá draft đủ điều kiện (mọi hạng mục done + có ảnh; quy tắc mock-up theo Epic 7 — khi Epic 7 chưa build, `mockup_required` mặc định tắt)
**When** bấm "Phát hành"
**Then** trong MỘT transaction: recompute qua hàm 4.2 → nếu `import_price` lệch nháp, dừng và hiện so sánh buộc xác nhận lại giá bán → snapshot khách [F2] + sản phẩm [F3] + `valid_until` (nếu có cấu hình [O3]) → `status=submitted`, `submitted_at`
**And** sau submitted: mọi mutation bị guard từ chối trừ closed-list AD-5 (e2e chứng minh); hạng mục thiếu điều kiện → chặn, chỉ đích danh hạng mục + bước dở.

### Story 6.2: PDF báo giá & tải về

As a đại lý,
I want PDF báo giá nhiều hạng mục sinh tự động và tải được ngay,
So that gửi khách bằng bất kỳ kênh nào.

**Acceptance Criteria:**

**Given** báo giá vừa submitted
**When** job render PDF chạy (BullMQ, template HTML + Playwright — AD-7)
**Then** PDF tiếng Việt gồm: thông tin khách (từ snapshot), bảng hạng mục (sản phẩm — SL — đơn giá — thành tiền), ảnh mock-up nếu có, subtotal + VAT + tổng, hạn hiệu lực nếu có; lưu storage, `quote_pdf_url` cập nhật (closed-list)
**And** UI hiển thị trạng thái render (polling) và nút tải PDF; job idempotent — chạy lại không tạo bản ghi trùng.

### Story 6.3: Gửi khách qua kênh tự động (Zalo OA)

As a đại lý,
I want hệ thống tự gửi PDF cho khách và cho tôi gửi lại khi lỗi,
So that khách nhận báo giá nhanh mà tôi không phải thao tác tay.

**Acceptance Criteria:**

**Given** interface `CustomerDeliveryChannel` (AD-13) với adapter mặc định "manual" (đánh dấu đã gửi tay) và adapter Zalo OA (bật qua config khi có OA)
**When** phát hành xong
**Then** bản ghi `customer_delivery` tạo với status pending→sent/failed; lỗi hiển thị lý do, nút "Gửi lại" tạo lần gửi mới không đổi nội dung PDF
**And** kênh lỗi không rollback trạng thái submitted (NFR-07).

### Story 6.4: Phiếu BOM sản xuất

As a công ty,
I want phiếu BOM sản xuất gộp mọi hạng mục × số lượng xuất ra chuẩn,
So that nhà máy sản xuất đúng đơn.

**Acceptance Criteria:**

**Given** báo giá submitted với BOM snapshot từng hạng mục
**When** job dispatch chạy
**Then** phiếu BOM (file PDF/Excel chuẩn) gộp vật tư mọi hạng mục nhân số lượng, kèm mã báo giá + thông số từng hạng mục; adapter `FactoryDispatchChannel` mặc định "manual/download" (kênh tự động chờ [O4])
**And** `factory_bom_dispatch` theo dõi status + retry; admin và đại lý tải được phiếu; **phiếu BOM chỉ tải được bởi vai admin** trừ khi công ty quyết khác (BOM chứa dữ liệu mật — theo AD-3).

## Epic 7: Ảnh mock-up AI

Sinh ảnh "sau khi lắp" theo hạng mục qua hàng đợi, không bao giờ làm tắc phát hành.

### Story 7.1: Hạ tầng sinh ảnh — queue, adapter & đếm lượt

As a hệ thống,
I want luồng sinh mock-up chạy nền với cơ chế đếm lượt chống race,
So that lượt gen đúng quy tắc D6 kể cả khi lỗi/retry (AD-14).

**Acceptance Criteria:**

**Given** interface `MockupProvider` (AD-13) + adapter mock (trả ảnh giả sau delay, phục vụ dev/test; provider thật chọn sau — Deferred)
**When** đại lý bấm "Tạo ảnh" cho hạng mục
**Then** một dòng `quote_item_ai_image` `status=pending` tạo ngay (giữ chỗ), `generation_no` tăng đơn điệu; job gọi provider → `succeeded` (có ảnh) hoặc `failed_technical`
**And** số lượt đã dùng = số dòng ≠ `failed_technical` sau `field_photo_updated_at`; đủ `ai_max_regenerate` → từ chối enqueue với `MOCKUP_QUOTA_EXCEEDED`; unit test 4 kịch bản (thành công / lỗi kỹ thuật / retry / thay ảnh reset).

### Story 7.2: Duyệt ảnh mock-up

As a đại lý,
I want xem ảnh sinh ra, tạo lại khi chưa ưng, chọn một ảnh cho mỗi hạng mục,
So that PDF có ảnh thuyết phục khách.

**Acceptance Criteria:**

**Given** hạng mục có ảnh hiện trường
**When** tạo ảnh và chờ
**Then** UI hiển thị tiến trình (polling, kỳ vọng ≤60s — NFR-06) + số lượt còn lại (UX-DR5); duyệt đặt `is_selected=true` (tối đa 1/hạng mục — chọn ảnh khác tự bỏ chọn ảnh cũ)
**And** lỗi kỹ thuật hiển thị "thử lại — không tính lượt".

### Story 7.3: Quy tắc 3 tầng khi phát hành

As a công ty,
I want ảnh mock-up bắt buộc theo nguyên tắc nhưng không bao giờ chặn chết nghiệp vụ,
So that vừa giữ chất lượng vừa không tắc [D6].

**Acceptance Criteria:**

**Given** `mockup_required` bật
**When** phát hành báo giá có hạng mục chưa có ảnh được chọn
**Then** chặn với 2 lối thoát rõ trên UI: (a) thay ảnh hiện trường → lượt gen tính lại từ mốc mới; (b) "phát hành không kèm ảnh" đòi `no_mockup_reason` bắt buộc, lưu + log
**And** báo cáo đếm được tỷ lệ phát hành không kèm mock-up (nguồn cho counter-metric G1); `mockup_required` tắt thì bỏ qua toàn bộ kiểm tra.

## Epic 8: Sẵn sàng vận hành — nghiệm thu workbook, seed & số liệu

Đưa dữ liệu và workbook thật vào hệ thống một cách có nghiệm thu; đo được G1–G4.

### Story 8.1: Bộ ca nghiệm thu workbook

As a admin,
I want định nghĩa bộ ca mẫu cho từng sản phẩm và chạy nghiệm thu workbook tự động,
So that chỉ workbook cho kết quả đúng mới được dùng tính giá thật [D14].

**Acceptance Criteria:**

**Given** workbook version đã qua kiểm tra hợp đồng (3.9)
**When** nạp bộ ca mẫu (input: bộ thông số; expected: [mã vật tư + số lượng] do người làm giá xác nhận) và bấm "Nghiệm thu"
**Then** hệ thống chạy từng ca qua adapter engine, so OUTPUT với kỳ vọng từng dòng; báo cáo liệt kê ca lệch (input, expected, actual); khớp 100% mới gắn `validated_at` và cho phép active
**And** bộ ca lưu theo sản phẩm, **tự chạy lại khi upload version workbook mới** — lệch thì chặn active (FR-091), kết quả ghi log.

### Story 8.2: Import seed dữ liệu ban đầu

As a admin,
I want import CSV/Excel: vật tư + đơn giá, SKU linh kiện + thông số, danh sách đại lý + gross,
So that không phải nhập tay hàng trăm bản ghi trước pilot.

**Acceptance Criteria:**

**Given** file mẫu tải từ hệ thống (template cột chuẩn)
**When** upload file import
**Then** preview + validate từng dòng (lỗi chỉ rõ dòng/cột), import theo giao dịch — lỗi thì không ghi nửa vời; bản ghi tạo qua đúng service nghiệp vụ nên audit log đầy đủ
**And** import lại file đã sửa cập nhật theo `code` (idempotent theo mã).

### Story 8.3: Chuẩn hoá & đưa workbook thật vào go-live

As a admin,
I want đưa file Excel BOM thật của công ty vào hệ thống theo chuẩn workbook và nghiệm thu go-live,
So that sản phẩm đầu tiên tính giá bằng chính logic thật [D14].

**Acceptance Criteria:**

**Given** file Excel BOM thật được công ty cung cấp [OPEN — chờ công ty; kèm quyết định O9 về nội dung OUTPUT]
**When** chuẩn hoá file theo mẫu (thêm sheet INPUT named ranges + sheet OUTPUT bảng chuẩn — không sửa logic tính), upload qua 3.9 và nghiệm thu qua 8.1
**Then** sản phẩm go-live với workbook đã nghiệm thu; hướng dẫn chuẩn hoá workbook được tài liệu hoá để nhân bản cho các sản phẩm sau
**And** quy trình vận hành ghi rõ: sửa logic = sửa file → upload version mới → nghiệm thu lại — không có đường tắt; story này giao khung + quy trình, chạy thật khi có file.

### Story 8.4: Số liệu vận hành G1–G4

As a admin,
I want xem các con số cơ bản đo mục tiêu kinh doanh,
So that biết hệ thống có đạt G1–G4 không ngay từ pilot.

**Acceptance Criteria:**

**Given** dữ liệu báo giá phát sinh
**When** mở màn "Số liệu"
**Then** hiển thị theo khoảng thời gian + đại lý + sản phẩm: số báo giá theo trạng thái, thời gian trung bình tạo→phát hành (G3), tỷ lệ phát hành không kèm mock-up (counter-metric), 100% tuân thủ range (G4 — đếm vi phạm phải = 0)
**And** số liệu export CSV; tỷ lệ chốt đơn (G1) ghi nhận là "chưa đo được trong hệ thống" — cần nguồn đơn hàng ngoài, hiển thị placeholder có chú thích.
