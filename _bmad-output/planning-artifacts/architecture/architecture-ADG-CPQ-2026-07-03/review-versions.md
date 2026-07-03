# Review kiểm chứng công nghệ — Bảng Stack ARCHITECTURE-SPINE

- **Ngày review:** 03/07/2026
- **Đối tượng:** `ARCHITECTURE-SPINE.md` (bảng Stack + AD-7, AD-12, Structural Seed)
- **Phương pháp:** WebSearch xác minh phiên bản/giấy phép hiện hành tại 07/2026
- **Verdict tổng:** Stack về cơ bản đúng và hiện đại; cần 6 chỉnh sửa — nghiêm trọng nhất là cảnh báo MinIO (Community Edition bị cắt tính năng) và pin cụ thể Node 24 LTS.

---

## 1. Next.js 16.2.x LTS — ✅ ĐÚNG, giữ nguyên

- Bản mới nhất: **16.2.10 LTS**, phát hành 01/07/2026, là bản được khuyến nghị ("latest supported version as of July 2026").
- Next.js 16 (Cache Components, Turbopack stable, React Compiler, React 19.2) là dòng LTS hiện hành; không có bản 17 GA tại 07/2026.
- Kết luận: pin `16.2.x` chính xác. Có thể ghi chú "hiện tại 16.2.10".

Nguồn:
- https://eosl.date/eol/product/nextjs/
- https://nextjs.org/blog/next-16
- https://versionlog.com/nextjs/16/

## 2. NestJS 11.1.x, pin v11 thay vì v12 — ✅ ĐÚNG, quyết định pin v11 hợp lý; cập nhật số minor

- Bản v11 mới nhất: **11.1.27** (npm `@nestjs/core`, ~giữa 06/2026); v11 vẫn được bảo trì tích cực (11.1.22, 11.1.23 phát hành 05/2026).
- **NestJS v12 chưa GA**: PR release v12.0.0 ghi rõ "approx. Q3 2026". v12 là bước nhảy lớn: full ESM, Standard Schema (zod native trong `@Body/@Query/@Param`), Vitest/oxlint/Rspack thay Jest/ESLint/Webpack.
- Đánh giá quyết định pin v11: **đúng cho dự án bắt đầu 07/2026** — không khởi động dự án trên major vừa/sắp ra với thay đổi module system (CJS→ESM) lan toàn toolchain. Ghi chú thêm: v12 có Standard Schema hỗ trợ zod trực tiếp, hợp với AD-10 — đáng đánh giá nâng cấp **sau MVP**, không phải giữa dự án.
- Kết luận: giữ pin v11; ghi "11.1.x (hiện tại 11.1.27)".

Nguồn:
- https://github.com/nestjs/nest/releases/tag/v11.1.23
- https://www.npmjs.com/package/@nestjs/core
- https://www.infoq.com/news/2026/04/nestjs-12-roadmap-esm/
- https://github.com/nestjs/nest/pull/16391

## 3. Prisma 7.x — ✅ ĐÚNG, giữ nguyên

- Prisma 7.0.0 ra 19/11/2025 (client Rust-free thuần TypeScript, query nhanh hơn ~3x, bundle nhỏ hơn 90%); minor đang tiến đến **7.7.0** — dòng 7.x là bản production được khuyến nghị, tiếp tục được hỗ trợ.
- "Prisma Next" (tương lai v8) mới ở giai đoạn định hướng, không ảnh hưởng lựa chọn hiện tại.
- Kết luận: pin `7.x` chính xác.

Nguồn:
- https://www.prisma.io/blog/announcing-prisma-orm-7-0-0
- https://www.infoq.com/news/2026/01/prisma-7-performance/
- https://www.prisma.io/blog/the-next-evolution-of-prisma-orm

## 4. PostgreSQL 16 — ⚠️ NÊN NÂNG LÊN 18 (greenfield)

- Các bản được hỗ trợ tại giữa 2026: **18.4, 17.10, 16.14, 15.18, 14.23**. PostgreSQL 18 GA từ 09/2025, đã ~10 tháng tuổi và ở minor thứ 4 (18.4) → đủ chín cho dự án mới. PostgreSQL 19 mới ở Beta 1 (06/2026).
- PG 16 **không sai** (EOL ~11/2028, còn hơn 2 năm), nhưng với dự án greenfield chưa viết dòng code nào, chọn 16 nghĩa là tự nhận nợ nâng cấp sớm. PG 18 có I/O subsystem mới (đọc nhanh tới 3x) và tối ưu index — có lợi cho workload báo giá/BOM đọc nhiều.
- Kết luận: đổi `PostgreSQL 16 [ASSUMPTION]` → `PostgreSQL 18 (18.4)`. Prisma 7 hỗ trợ đầy đủ PG 18. Nếu hạ tầng cloud VN chỉ có PG 17 managed thì 17 cũng chấp nhận được (EOL 2030) — điều cần tránh là 16.

Nguồn:
- https://www.postgresql.org/about/news/postgresql-184-1710-1614-1518-and-1423-released-3297/
- https://www.postgresql.org/support/versioning/
- https://www.postgresql.org/about/news/postgresql-18-released-3142/

## 5. Node.js "LTS hiện hành" — ⚠️ GHI RÕ SỐ: Node 24 LTS

- **Node.js 24 "Krypton" là Active LTS hiện hành**, hỗ trợ đến **30/04/2028**. Node 22 chỉ còn Maintenance LTS. Node 26 đang là Current (non-LTS), chỉ vào LTS từ **10/2026**.
- Spine đang ghi "LTS hiện hành (pin trong .nvmrc lúc khởi tạo) [ASSUMPTION]" — mơ hồ, hai dev có thể pin khác nhau (22 vs 24). Ghi cứng **Node 24** và gỡ tag ASSUMPTION.
- Lưu ý: từ Node 27 lịch phát hành đổi sang mỗi năm một major đều vào LTS — không ảnh hưởng MVP.

Nguồn:
- https://nodejs.org/en/about/previous-releases
- https://nodesource.com/blog/nodejs-24-becomes-lts
- https://endoflife.date/nodejs

## 6. Redis 7.x + BullMQ — ⚠️ CẬP NHẬT: Redis 8 (AGPLv3) hoặc Valkey

- Redis 8 phát hành dưới **tri-license RSALv2 / SSPLv1 / AGPLv3** — chọn AGPLv3 là dùng open source hợp lệ; self-host nội bộ (không phân phối, không bán Redis-as-a-service) hoàn toàn ổn về pháp lý.
- Dòng **7.x là dòng cũ trước thời đổi giấy phép**; pin 7.x năm 2026 là pin bản đang lùi dần về maintenance.
- Phương án thay thế sạch giấy phép: **Valkey** (fork BSD-3-Clause, tương thích API Redis 7.2, đã ra v9). **BullMQ chính thức hỗ trợ Valkey — toàn bộ test suite chạy trên Valkey mỗi release**, và BullMQ chỉ yêu cầu Redis ≥ 6.2. → đổi engine sau này là drop-in, không đụng code.
- Kết luận: đổi `Redis 7.x` → `Redis 8.x (AGPLv3) hoặc Valkey 8+`; BullMQ bản hiện hành giữ nguyên. AD-7 không cần sửa.

Nguồn:
- https://redis.io/blog/agplv3/
- https://github.com/taskforcesh/bullmq/discussions/2642
- https://www.infoq.com/news/2025/05/redis-agpl-license/

## 7. Playwright render PDF — ✅ VẪN LÀ CÁCH PHỔ BIẾN/TỐT NHẤT, giữ nguyên

- Benchmark 2026: Playwright là công cụ HTML→PDF nhanh nhất (13ms/tài liệu phức tạp trên browser warm, so với 58ms Puppeteer, 629ms WeasyPrint); chất lượng render Chromium, CSS print (`@page`, `break-inside`) hoạt động đầy đủ.
- Lưu ý vận hành (đã khớp AD-7): mỗi Chromium instance ăn 150–300MB RAM → **phải render qua queue với concurrency giới hạn và giữ browser instance dùng lại** — đúng mô hình BullMQ worker của spine. Nên thêm ghi chú này vào hàng Playwright.
- Managed PDF API là lựa chọn thay thế được quảng bá 2026, nhưng với quy mô nhỏ + self-host + dữ liệu giá nhạy cảm (AD-3), Playwright self-host hợp lý hơn.

Nguồn:
- https://pdf4.dev/blog/html-to-pdf-benchmark-2026
- https://www.browserstack.com/guide/playwright-pdf-html-generation

## 8. MinIO — 🚩 CẢNH BÁO CHÍNH CỦA REVIEW

- **Giấy phép:** AGPLv3 — với self-host dùng nội bộ (ADG không phân phối MinIO, không bán object storage) thì **AGPL không tạo nghĩa vụ mở mã nguồn ứng dụng CPQ** (app chỉ nói chuyện qua S3 API, không link mã MinIO). Về pháp lý: ổn.
- **Rủi ro thật là hướng sản phẩm:** năm 2025 MinIO **gỡ toàn bộ Web UI quản trị khỏi Community Edition** (policies, monitoring, replication controls); các tính năng chỉ còn trong AIStor Enterprise (~$96k/năm); cộng đồng phản ứng mạnh, xuất hiện fork **OpenMaxIO**. Community Edition vẫn chạy được qua CLI (`mc`) nhưng tín hiệu là bản free sẽ tiếp tục bị siết.
- **Khuyến nghị:** giữ S3-compatible làm hợp đồng (đúng), nhưng đổi ghi chú hàng Object storage thành: ưu tiên **S3 cloud VN** hoặc **Garage / SeaweedFS** (giấy phép mở thật, nhẹ, hợp quy mô nhỏ); nếu vẫn dùng MinIO thì ghi rõ chấp nhận quản trị bằng CLI, không dựa Web UI. Vì app chỉ dùng S3 API chuẩn nên đổi backend sau này không đụng code.

Nguồn:
- https://blocksandfiles.com/2025/06/19/minio-removes-management-features-from-basic-community-edition-object-storage-code/
- https://www.futuriom.com/articles/news/minio-faces-fallout-for-stripping-features-from-web-gui/2025/06
- https://www.min.io/blog/from-open-source-to-free-and-open-source-minio-is-now-fully-licensed-under-gnu-agplv3

## 9. pnpm — ⚠️ PIN pnpm 11

- **pnpm 11** ra 28/04/2026, hiện ở ~11.7–11.9, là dòng được bảo trì duy nhất; **pnpm 10.x không còn được bảo trì tích cực**, pnpm 9 EOL 30/04/2026.
- Spine chỉ ghi "pnpm workspaces" không số — pin `pnpm 11.x` (qua trường `packageManager` trong package.json / corepack).

Nguồn:
- https://pnpm.io/blog/releases/11.0
- https://eosl.date/eol/product/pnpm/
- https://endoflife.date/pnpm

## 10. Các mục không cần đổi

| Mục | Kết luận |
|---|---|
| TypeScript 5.x | Giữ — vẫn là dòng hiện hành |
| zod "bản hiện hành" | Giữ — zod 4 là bản hiện hành; NestJS v12 sau này còn hỗ trợ native (Standard Schema) |
| BullMQ "bản hiện hành" | Giữ — yêu cầu tối thiểu Redis 6.2, chạy tốt trên Redis 8/Valkey |
| Nguồn đã dẫn trong spine (nextjs.org/blog, nestjs releases, prisma blog) | Còn đúng |

---

## Tổng hợp chỉnh sửa cần áp vào bảng Stack

| # | Hiện tại | Sửa thành | Lý do |
|---|---|---|---|
| 1 | Node.js "LTS hiện hành [ASSUMPTION]" | **Node.js 24 LTS (Krypton)** | Active LTS đến 04/2028; Node 26 chỉ vào LTS từ 10/2026 — gỡ mơ hồ |
| 2 | NestJS 11.1.x | NestJS **11.1.x (hiện tại 11.1.27)** — giữ pin v11 | v12 (full ESM) mới dự kiến Q3/2026; đánh giá nâng cấp sau MVP |
| 3 | PostgreSQL 16 [ASSUMPTION] | **PostgreSQL 18 (18.4)** | Greenfield nên vào major chín hiện hành (GA 09/2025); 16 EOL 11/2028 tạo nợ nâng cấp sớm |
| 4 | Redis 7.x | **Redis 8.x (AGPLv3) hoặc Valkey 8+** | 7.x là dòng cũ; Valkey BSD-3 được BullMQ test chính thức — drop-in |
| 5 | MinIO self-host [ASSUMPTION] | S3-compatible: **cloud VN / Garage / SeaweedFS ưu tiên; MinIO chấp nhận CLI-only** | MinIO CE bị gỡ Web UI quản trị 2025, tính năng dồn về Enterprise $96k/năm |
| 6 | pnpm workspaces (không số) | **pnpm 11.x** (pin qua `packageManager`) | pnpm 10 hết bảo trì; 11 là dòng hiện hành từ 04/2026 |

Xác nhận đúng, không đổi: Next.js 16.2.x LTS (16.2.10), Prisma 7.x (7.7.0), Playwright cho PDF (vẫn là lựa chọn nhanh/phổ biến nhất 2026 — thêm ghi chú browser pool + queue, đã khớp AD-7), TypeScript 5.x, zod, BullMQ.
