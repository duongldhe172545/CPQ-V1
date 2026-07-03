# Review Rubric Walker — ARCHITECTURE-SPINE ADG CPQ

- **Ngày review:** 2026-07-03
- **Reviewer:** rubric walker (checklist good-spine trong `reviewer-gate.md`)
- **Đối tượng:** `_bmad-output/planning-artifacts/architecture/architecture-ADG-CPQ-2026-07-03/ARCHITECTURE-SPINE.md`
- **Nguồn đối chiếu:** `prd.md` mục 6–8 (FR/NFR/phụ thuộc)

## Verdict

**ĐẠT CÓ ĐIỀU KIỆN.** Bộ 13 AD có chất invariant thật, Rule phần lớn enforce được, seed tối giản, Mermaid hợp lệ. Nhưng có 3 dimension bị bỏ im lặng (không decided/deferred/open): đồng thời NFR-10, bao thư bảo mật NFR-02 (hash mật khẩu, HTTPS, Nghị định 13/2023), và migration/seed dữ liệu ban đầu ngoài công thức. Cộng thêm 4 mục Stack chưa pin phiên bản.

---

## 1. Từng AD có đúng chất "invariant" không?

| AD | Chất invariant? | Nhận xét |
|---|---|---|
| AD-1 Modular monolith, layered | ✅ | Không chỉ tuyên bố cấu trúc — Rule cấm import repository chéo module và cấm tách service khi chưa có AD mới → chặn đúng divergence "hai dev tự chọn kiến trúc khác nhau". Enforce được bằng lint dependency (eslint boundaries / nx). |
| AD-2 Tính giá chỉ ở server | ✅ | Invariant mẫu mực: chặn lệch kết quả + lộ công thức. Kiểm được bằng review + kiểm frontend không có logic tính. |
| AD-3 DTO whitelist cho dealer | ✅ | Rule kèm cơ chế kiểm chứng cụ thể (e2e cố định: đăng nhập dealer, quét mọi endpoint, assert không có field mật). Đây là AD mạnh nhất trong spine. |
| AD-4 Một formula engine | ✅ | Chặn hai bộ semantics + cấm eval. Pure function, ràng đúng vào đặc tả ERD Project Note (FR-022). |
| AD-5 Publish = transaction snapshot, submitted bất biến | ✅ | Khớp FR-060/061/065 và NFR-09. "Nháp chỉ giữ số tham khảo, không phải chân lý" là câu chốt đúng — loại bỏ divergence hai đường ghi giá. Guard chung từ chối mutation khi submitted là enforce được. |
| AD-6 Tiền integer VND, một hàm làm tròn | ✅ | Khớp FR-035 (làm tròn từng dòng, tổng = cộng số đã tròn). Lưu ý: FR-035 còn treo `[ASSUMPTION — chờ kế toán]` — spine nên nhắc lại điều kiện này (nếu kế toán đổi quy tắc, AD-6 phải sửa theo). |
| AD-7 Việc chậm qua queue, job idempotent | ✅ | "Job nhận id, không nhận payload, tự đọc trạng thái mới nhất" là quy tắc idempotency cụ thể, enforce được qua review + test retry. Khớp NFR-06/07. |
| AD-8 Dealer scoping từ token | ✅ | Chặn đúng lỗ hổng tenant-leak. "Endpoint nhận dealer_id từ body/query cho vai dealer là sai kiến trúc" — phát biểu kiểu falsifiable, tốt. |
| AD-9 Audit qua interceptor chung | ✅ | Khớp FR-073/NFR-09. Liệt kê tường minh các bảng nhạy cảm → kiểm được. |
| AD-10 REST /api/v1 + zod shared + error envelope | ✅ | Là contract-invariant chứ không phải structural: FE/BE cùng import schema, envelope lỗi duy nhất. Enforce được. |
| AD-11 Web-first, chừa đường ZMP | ✅ | Khớp NFR-03/04. Rule cấm SDK Zalo ở lõi + mô hình phiên mở được provider — đúng dạng "giữ đường Phase 2 mà không build Phase 2". |
| AD-12 PostgreSQL + Prisma đường dữ liệu duy nhất | ✅ (biên) | Nửa đầu là stack pick, nhưng phần "Prisma schema + migrations là nguồn physical duy nhất, đổi logical phải đối chiếu decision log, raw SQL chỉ trong repository kèm lý do" là invariant thật. Chấp nhận. |
| AD-13 Tích hợp ngoài qua port/adapter | ✅ | Ba interface đặt tên cụ thể; cô lập đúng 2 phụ thuộc chưa chốt trong PRD mục 8 (AI provider, kênh nhà máy). Deferred tương ứng an toàn nhờ AD này. |

**Kết luận mục 1:** không có AD nào là seed/structural giả dạng. AD-1 và AD-12 đứng sát ranh nhưng đều có Rule chặn divergence cụ thể nên qua.

## 2. Binds / Prevents / Rule có enforce được không?

- Mọi AD đủ 3 trường; Prevents đều nêu divergence cụ thể (không phải "để code sạch"); Rule đa số phát biểu dạng kiểm chứng được (cấm/bắt buộc + cơ chế: guard chung, interceptor chung, e2e cố định, package duy nhất).
- Binds trỏ đúng FR: đối chiếu chéo FR-011/012/021/022/030..036/040/046/050/060/061/063/064/065/070/073/090 — đều khớp nội dung PRD mục 6.
- Nhược điểm nhỏ: AD-11 ghi `Binds: all frontend` — không phải capability ID chuẩn, máy khó đối chiếu; nên đổi thành danh sách F có mặt frontend hoặc `all` kèm ghi chú. (LOW)

## 3. Dimension bị bỏ im lặng (finding chính)

Đối chiếu NFR (PRD mục 7) và phụ thuộc (mục 8) với spine (AD + Conventions + Deferred + Open):

1. **HIGH — NFR-10 (đồng thời / khoá lạc quan) hoàn toàn im lặng.** PRD yêu cầu: hai tài khoản cùng đại lý sửa cùng nháp → khoá lạc quan + cảnh báo, không ghi đè âm thầm. Spine không có AD, không convention (ví dụ cột `version`/`updated_at` check), không dòng Deferred nào nhắc tới. Đây đúng loại quyết định mà mỗi dev sẽ tự xử một kiểu (last-write-wins vs version check vs lock) → divergence chắc chắn. Phải decided hoặc ít nhất deferred có chủ đích.
2. **HIGH — Bao thư bảo mật NFR-02 chỉ được quyết một phần.** AD-11 quyết mô hình phiên (cookie httpOnly), AD-3/AD-8 quyết che dữ liệu giá. Nhưng: thuật toán băm mật khẩu, HTTPS/TLS termination, và tuân thủ **Nghị định 13/2023/NĐ-CP** về dữ liệu cá nhân khách hàng (SĐT, địa chỉ trong module `customer` + snapshot vĩnh viễn ở AD-5 — snapshot bất biến chứa PII có xung đột tiềm tàng với quyền xoá dữ liệu cá nhân) không xuất hiện ở bất kỳ đâu: không AD, không convention, không Deferred, không open question.
3. **HIGH — Migration/seed dữ liệu ban đầu im lặng ngoài công thức.** F9/`import-tools` chỉ lo import công thức + golden tests. Con đường đưa dữ liệu ban đầu vào hệ thống — danh mục vật tư + đơn giá, SKU linh kiện, phụ kiện, ~300 đại lý + gross, tài khoản admin bootstrap — không được quyết (import tool? seed script? nhập tay qua admin?) và cũng không nằm trong Deferred. Rubric gate nêu đích danh dimension này.
4. **MEDIUM — Vận hành/giám sát:** FR-023 đòi "cảnh báo admin" khi công thức lỗi; NFR-07 đòi theo dõi + gửi lại. Spine có logging convention và status-tracking qua queue, nhưng cơ chế alerting/monitoring (cái gì báo cho admin, ở đâu) không decided/deferred. Backup chỉ là một nhãn trong Mermaid ("backup hằng ngày") — chưa phải quyết định (retention? restore test?).
5. **MEDIUM — Nơi xử lý upload/lưu ảnh hiện trường (FR-042: ≤8MB, validate định dạng/độ phân giải):** stack có S3/MinIO nhưng đường upload (presigned URL vs qua API) không được quyết — hai dev sẽ chọn hai kiểu. Có thể defer, nhưng phải nói ra.

Các chiều còn lại đều có chủ: hạ tầng (open, chốt trước pilot), CI/CD (deferred), AI provider + kênh nhà máy (deferred, cô lập bởi AD-13), O1–O3 (deferred), physical DB (deferred), ZMP (deferred). Deferred hiện tại **không** chứa mục nào để hai unit diverge — các mục đều được cô lập bằng adapter/config/Docker. Đạt tiêu chí "nothing under Deferred could let two units diverge".

## 4. Mermaid

- **Diagram 1 (dependency):** cú pháp hợp lệ (node label có ngoặc/dấu tiếng Việt đều nằm trong quotes; edge label `|REST /api/v1|` và `|"port/adapter"|` hợp lệ; cylinder `[( )]` đúng). Semantics đúng chiều phụ thuộc. Nhánh `WORKER --> API` hơi mơ hồ (worker là chế độ chạy của chính api, không phải thành phần phụ thuộc qua network) — chú thích trong node đã cứu, chấp nhận. (LOW)
- **Diagram 2 (deploy):** hợp lệ; subgraph title quoted, `<br/>` trong quotes OK. Đã dán `[ASSUMPTION — hạ tầng chưa chốt]` đúng chỗ.

## 5. Seed có tối giản không?

Có. ~15 dòng, chỉ nêu xương monorepo (apps/api modules + common, apps/web hai vùng dealer/admin, 2 packages, docker-compose) — không liệt kê file lẻ, không lock cấu trúc thư mục con ngoài mức cần cho AD-1/AD-4/AD-10. Đạt.

## 6. Stack

- Có dòng xác minh phiên bản kèm ngày (03/07/2026) và nguồn — đạt yêu cầu "verified-current" về hình thức.
- **MEDIUM — 4 mục chưa pin:** Node.js ("LTS hiện hành"), zod, BullMQ, Playwright đều ghi "bản hiện hành" — đây chính là loại miss mà lint spine bắt (unpinned Stack versions). Node có kế hoạch pin qua `.nvmrc` lúc khởi tạo (chấp nhận được nếu ghi rõ ai pin, khi nào); ba mục còn lại nên pin major.
- Ghi chú tốt: NestJS pin v11 kèm lý do (không nhảy v12 ESM giữa dự án) — đúng tinh thần invariant.

## 7. Coverage FR/NFR (PRD mục 6–8)

- F1–F9: đủ mặt trong Capability Map, mỗi capability có module + AD govern. Đối chiếu từng nhóm FR: không FR nào rơi ngoài module map.
- NFR-01 ✅ (AD-3, AD-8 + e2e bắt buộc trong Conventions). NFR-03/04 ✅ (AD-11 + seed hai vùng UI). NFR-05 ✅ (prod một máy, nói rõ không phân tán). NFR-06/07 ✅ (AD-7). NFR-08 ✅ (conventions i18n/tiền tệ). NFR-09 ✅ (AD-5, AD-9). **NFR-02 ⚠️ một phần, NFR-10 ✗ im lặng** — xem mục 3.
- Phụ thuộc mục 8: cả 5 dòng đều có chỗ đứng trong spine (AD-13 + Deferred + O1–O4). ✅

## 8. Danh sách finding

| # | Mức | Finding | Đề xuất |
|---|---|---|---|
| 1 | HIGH | NFR-10 khoá lạc quan hoàn toàn im lặng | Thêm AD (hoặc convention + Deferred có chủ đích): cơ chế version check chung cho quote nháp |
| 2 | HIGH | NFR-02 quyết một phần: hash mật khẩu, HTTPS, Nghị định 13/2023 (PII trong snapshot bất biến) không có chỗ đứng | Thêm convention bảo mật + open question về PII-trong-snapshot |
| 3 | HIGH | Migration/seed dữ liệu ban đầu (vật tư, SKU, đại lý, admin bootstrap) không decided/deferred | Quyết con đường nhập liệu ban đầu hoặc thêm dòng Deferred tường minh |
| 4 | MEDIUM | 4 mục Stack chưa pin (Node, zod, BullMQ, Playwright) | Pin major hoặc ghi rõ cơ chế pin lúc khởi tạo |
| 5 | MEDIUM | Alerting/giám sát (FR-023 "cảnh báo admin", NFR-07) và backup/restore chỉ là nhãn diagram | Thêm convention hoặc Deferred |
| 6 | MEDIUM | Đường upload ảnh hiện trường (FR-042) chưa quyết (presigned vs qua API) | Quyết hoặc defer tường minh |
| 7 | LOW | AD-11 `Binds: all frontend` không phải capability ID chuẩn | Đổi thành danh sách F |
| 8 | LOW | AD-6 chưa nhắc điều kiện `[ASSUMPTION — chờ kế toán]` của FR-035 | Thêm ghi chú điều kiện |
| 9 | LOW | Nhánh `WORKER --> API` trong diagram dependency mơ hồ về ngữ nghĩa | Ghi chú "cùng process image" |

**Tổng:** 3 HIGH · 3 MEDIUM · 3 LOW. Không có CRITICAL — không finding nào cho thấy AD sai hay mâu thuẫn PRD; tất cả là thiếu-chỗ-đứng (silence), sửa được bằng bổ sung, không phải đập lại.
