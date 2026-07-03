---
name: 'ADG CPQ'
type: architecture-spine
purpose: build-substrate
altitude: feature
paradigm: 'Modular monolith (NestJS) + layered controllerâ†’serviceâ†’repository; frontend Next.js tÃ¡ch riÃªng; monorepo'
scope: 'ToÃ n há»‡ thá»‘ng ADG CPQ â€” MVP web-first theo PRD 2026-07-03'
status: final
created: '2026-07-03'
updated: '2026-07-03'
binds: [F1, F2, F3, F4, F5, F6, F7, F8, F9]
sources:
  - _bmad-output/planning-artifacts/prds/prd-ADG-CPQ-2026-07-03/prd.md
  - _bmad-output/planning-artifacts/prds/prd-ADG-CPQ-2026-07-03/addendum.md
  - docs/ADG_CPQ_ERD.dbml
  - _bmad-output/planning-artifacts/ADG-CPQ-decision-log.md
companions: []
---

# Architecture Spine â€” ADG CPQ

## Design Paradigm

**Modular monolith** trÃªn NestJS: má»™t API duy nháº¥t chia **module theo domain nghiá»‡p vá»¥**, trong má»—i module lÃ  3 lá»›p `controller â†’ service â†’ repository (Prisma)`. Frontend lÃ  app Next.js riÃªng, giao tiáº¿p vá»›i API qua REST â€” khÃ´ng cÃ³ logic nghiá»‡p vá»¥ á»Ÿ frontend. Viá»‡c cháº­m cháº¡y qua **job queue** (BullMQ) trong cÃ¹ng codebase API, báº­t báº±ng cháº¿ Ä‘á»™ worker.

Ãnh xáº¡ module â†” domain:

| Module | Domain | NhÃ³m FR |
|---|---|---|
| `auth` | ÄÄƒng nháº­p, phiÃªn, vai trÃ² | F8 |
| `dealer` | Äáº¡i lÃ½, gross Ä‘áº¡i lÃ½ | F8 |
| `customer` | Kho khÃ¡ch theo Ä‘áº¡i lÃ½ | F5 |
| `catalog` | Danh má»¥c, sáº£n pháº©m, thÃ´ng sá»‘ dá»± Ã¡n, phá»¥ kiá»‡n | F1 |
| `component` | Loáº¡i/SKU linh kiá»‡n, auto-fill, tÆ°Æ¡ng thÃ­ch | F2 |
| `material` | Váº­t tÆ°, BOM template | F3 |
| `pricing` | Gross, range, VAT, tÃ­nh giÃ¡ | F4 |
| `quote` | Pipeline bÃ¡o giÃ¡, háº¡ng má»¥c, snapshot, phÃ¡t hÃ nh | F5, F7 |
| `mockup` | áº¢nh AI (adapter + queue) | F6 |
| `delivery` | Gá»­i khÃ¡ch, gá»­i nhÃ  mÃ¡y, PDF | F7 |
| `audit` | config_audit_log, interceptor | F8 |
| `import-tools` | Import cÃ´ng thá»©c, golden tests | F9 |

## Invariants & Rules

Chiá»u phá»¥ thuá»™c cho phÃ©p (vi pháº¡m = sai kiáº¿n trÃºc):

```mermaid
graph TD
    WEB["apps/web (Next.js)"] -->|REST /api/v1| API["apps/api (NestJS modules)"]
    WEB --> SHARED["packages/shared (zod schemas, types)"]
    API --> SHARED
    API --> FE["packages/formula-engine"]
    API --> DB[("PostgreSQL qua Prisma")]
    API --> Q[("Redis / BullMQ")]
    WORKER["worker mode (cÃ¹ng codebase api)"] --> API
    API -->|"port/adapter"| EXT["AI mockup Â· Zalo OA Â· Cá»•ng nhÃ  mÃ¡y"]
```

### AD-1 â€” Modular monolith NestJS, layered trong module

- **Binds:** all
- **Prevents:** hai dev/agent tá»± chá»n kiáº¿n trÃºc khÃ¡c nhau cho hai module (microservice láº», code nghiá»‡p vá»¥ trong controller, gá»i tháº³ng Prisma tá»« controller).
- **Rule:** má»—i domain má»™t NestJS module; trong module Ä‘i Ä‘Ãºng `controller â†’ service â†’ repository`; module khÃ¡c chá»‰ Ä‘Æ°á»£c gá»i qua service export, khÃ´ng import repository cá»§a nhau. **Má»—i báº£ng DB cÃ³ Ä‘Ãºng má»™t module chá»§ sá»Ÿ há»¯u** (toÃ n bá»™ nhÃ³m `quote_*` thuá»™c `quote`) â€” module khÃ¡c muá»‘n Ä‘á»c/ghi báº£ng Ä‘Ã³ pháº£i Ä‘i qua service cá»§a module chá»§, cáº¥m má»Ÿ repository thá»© hai vÃ o cÃ¹ng báº£ng (Ä‘Æ°á»ng ghi thá»© hai sáº½ nÃ© guard submitted/dealer-scope).

### AD-2 â€” TÃ­nh toÃ¡n giÃ¡/BOM chá»‰ á»Ÿ server

- **Binds:** F3, F4, F5 (FR-030..036, FR-046)
- **Prevents:** frontend tá»± tÃ­nh giÃ¡ â†’ lá»‡ch káº¿t quáº£ vá»›i server vÃ  lá»™ cÃ´ng thá»©c/gross ra client.
- **Rule:** má»i phÃ©p tÃ­nh BOM, cost, gross, giÃ¡ nháº­p, range, VAT, tá»•ng cháº¡y trong module `pricing`/`quote` phÃ­a API. Frontend chá»‰ hiá»ƒn thá»‹ sá»‘ server tráº£ vá»; khÃ´ng cÃ³ báº£n sao cÃ´ng thá»©c nÃ o á»Ÿ client.

### AD-3 â€” Che dá»¯ liá»‡u máº­t á»Ÿ táº§ng API báº±ng DTO whitelist

- **Binds:** F4, F5, F7 (NFR-01, FR-036)
- **Prevents:** field máº­t (cost_material, unit_price váº­t tÆ°, gross, BOM chi tiáº¿t) lá»t vÃ o payload cho vai dealer rá»“i "giáº¥u báº±ng UI".
- **Rule:** response DTO cho vai dealer lÃ  **whitelist** khai bÃ¡o tÆ°á»ng minh; field máº­t khÃ´ng xuáº¥t hiá»‡n trong DTO Ä‘Ã³ (ká»ƒ cáº£ giÃ¡ trá»‹ null). Bá»™ e2e test cÃ³ ca cá»‘ Ä‘á»‹nh: Ä‘Äƒng nháº­p dealer, gá»i má»i endpoint quote/pricing, assert khÃ´ng cÃ³ field máº­t trong JSON.

### AD-4 â€” Má»™t formula engine duy nháº¥t, package riÃªng

- **Binds:** F2, F3, F9 (FR-011, FR-012, FR-021, FR-022, FR-090)
- **Prevents:** auto-fill, Ä‘iá»u kiá»‡n tÆ°Æ¡ng thÃ­ch, BOM vÃ  golden tests dÃ¹ng hai bá»™ semantics khÃ¡c nhau; hoáº·c dÃ¹ng `eval` JS gÃ¢y lá»— há»•ng.
- **Rule:** `packages/formula-engine` lÃ  parser AST tá»± viáº¿t theo Ä‘áº·c táº£ trong ERD Project Note (sá»‘ há»c, so sÃ¡nh, logic, `if`, `round/ceil/floor/min/max`, biáº¿n `proj.*`/`comp.*`). Pure function `(expression, variables) â†’ value | typedError`, khÃ´ng truy cáº­p I/O, khÃ´ng eval. Má»i nÆ¡i cáº§n cháº¡y cÃ´ng thá»©c Ä‘á»u import package nÃ y.

### AD-5 â€” PhÃ¡t hÃ nh lÃ  transaction snapshot; sau submitted lÃ  báº¥t biáº¿n

- **Binds:** F5, F7 (FR-060, FR-061, FR-065; D4)
- **Prevents:** hai luá»“ng ghi giÃ¡ theo hai Ä‘Æ°á»ng (má»™t Ä‘Æ°á»ng tin giÃ¡ nhÃ¡p, má»™t Ä‘Æ°á»ng tÃ­nh láº¡i) â†’ bÃ¡o giÃ¡ phÃ¡t hÃ nh vá»›i sá»‘ liá»‡u khÃ´ng nháº¥t quÃ¡n; hoáº·c code khÃ¡c sá»­a quote Ä‘Ã£ gá»­i.
- **Rule:** phÃ¢n biá»‡t hai loáº¡i cá»™t tiá»n: **INPUT** (`selling_price` â€” Ä‘áº¡i lÃ½ chá»n, persist bÃ¬nh thÆ°á»ng á»Ÿ nhÃ¡p) vÃ  **DERIVED** (`cost_material`, `import_price`, `line_total`, tá»•ng â€” chá»‰ Ä‘Æ°á»£c ghi bá»Ÿi **má»™t hÃ m Ä‘á»‹nh giÃ¡ duy nháº¥t** trong module `pricing`, kÃ¨m má»‘c `priced_at`; á»Ÿ nhÃ¡p lÃ  cache tham kháº£o). PhÃ¡t hÃ nh = má»™t transaction DB duy nháº¥t: recompute qua Ä‘Ãºng hÃ m Ä‘Ã³ â†’ ghi snapshot (khÃ¡ch, sáº£n pháº©m, thÃ´ng sá»‘, BOM, giÃ¡) â†’ set `submitted`; chá»‰ sá»‘ liá»‡u táº¡i publish lÃ  chÃ¢n lÃ½. Sau `submitted`, guard chung tá»« chá»‘i má»i mutation **trá»« closed-list cá»™t há»‡ thá»‘ng**: `quote_pdf_url`, `valid_until` (ghi trong chÃ­nh transaction publish) vÃ  cÃ¡c báº£ng tráº¡ng thÃ¡i gá»­i (`customer_delivery`, `factory_bom_dispatch`) â€” khÃ´ng cá»™t sá»‘ liá»‡u nÃ o Ä‘Æ°á»£c ghi sau publish.

### AD-6 â€” Tiá»n lÃ  sá»‘ nguyÃªn VND, má»™t hÃ m lÃ m trÃ²n

- **Binds:** F4 (FR-035)
- **Prevents:** má»—i chá»— lÃ m trÃ²n má»™t kiá»ƒu â†’ PDF, DB vÃ  tá»•ng khÃ´ng khá»›p nhau.
- **Rule:** má»i giÃ¡ trá»‹ tiá»n trong há»‡ thá»‘ng lÃ  **sá»‘ nguyÃªn Ä‘á»“ng**; lÃ m trÃ²n duy nháº¥t qua `roundVnd()` trong `packages/formula-engine` (module money), Ã¡p táº¡i tá»«ng dÃ²ng; tá»•ng = cá»™ng cÃ¡c dÃ²ng Ä‘Ã£ lÃ m trÃ²n, khÃ´ng lÃ m trÃ²n láº¡i. Sá»‘ lÆ°á»£ng giá»¯ 4 sá»‘ láº». **Ãnh xáº¡ váº­t lÃ½:** cá»™t tiá»n map `BIGINT` trong Prisma schema (ERD ghi `decimal(18,2)` lÃ  logical â€” physical dÃ¹ng bigint, ghi chÃº `@map`); JSON serialize **tiá»n lÃ  number** (an toÃ n trong pháº¡m vi VND thá»±c táº¿) â€” ngoáº¡i lá»‡ cÃ³ chá»§ Ä‘Ã­ch cá»§a convention "bigint â†’ string" vá»‘n chá»‰ Ã¡p cho ID.

### AD-7 â€” Viá»‡c cháº­m Ä‘i qua queue, job idempotent

- **Binds:** F6, F7 (FR-050, FR-063, FR-064; NFR-06, NFR-07)
- **Prevents:** gá»i AI/gá»­i OA/render PDF ngay trong request HTTP â†’ timeout, máº¥t job, double-send khi retry.
- **Rule:** AI mockup, render PDF, gá»­i khÃ¡ch, gá»­i nhÃ  mÃ¡y Ä‘á»u lÃ  BullMQ job. Job nháº­n id báº£n ghi (khÃ´ng nháº­n payload dá»¯ liá»‡u), tá»± Ä‘á»c tráº¡ng thÃ¡i má»›i nháº¥t, **idempotent** (cháº¡y láº¡i khÃ´ng gá»­i Ä‘Ã´i), cáº­p nháº­t status vÃ o báº£ng tÆ°Æ¡ng á»©ng; client theo dÃµi báº±ng polling status.

### AD-8 â€” Dealer scoping báº¯t buá»™c tá»« token

- **Binds:** F5, F8 (FR-040, FR-070; NFR-01)
- **Prevents:** Ä‘áº¡i lÃ½ A Ä‘á»c/ghi dá»¯ liá»‡u Ä‘áº¡i lÃ½ B do quÃªn filter hoáº·c tin `dealer_id` client gá»­i lÃªn.
- **Rule:** `dealer_id` láº¥y **duy nháº¥t tá»« phiÃªn Ä‘Äƒng nháº­p**; guard/decorator chung Ã¡p filter nÃ y cho má»i truy váº¥n dá»¯ liá»‡u thuá»™c Ä‘áº¡i lÃ½ (customer, quote). Endpoint nháº­n `dealer_id` tá»« body/query cho vai dealer lÃ  sai kiáº¿n trÃºc.

### AD-9 â€” Audit qua interceptor chung

- **Binds:** F1â€“F4, F8 (FR-073)
- **Prevents:** má»—i mÃ n hÃ¬nh admin tá»± viáº¿t log tay â†’ chá»— cÃ³ chá»— khÃ´ng, khiáº¿u náº¡i giÃ¡ khÃ´ng truy Ä‘Æ°á»£c.
- **Rule:** má»i mutation trÃªn báº£ng cáº¥u hÃ¬nh nháº¡y cáº£m (material, component_sku, accessory, pricing_config, dealer_product_gross, formula, system_setting) Ä‘i qua má»™t interceptor/audit-service chung ghi `config_audit_log` (báº£ng, id, action, diff tÃ³m táº¯t, user, thá»i Ä‘iá»ƒm). KhÃ´ng viáº¿t insert audit ráº£i rÃ¡c.

### AD-10 â€” Má»™t há»£p Ä‘á»“ng API: REST /api/v1 + zod shared

- **Binds:** all
- **Prevents:** FE vÃ  BE lá»‡ch shape dá»¯ liá»‡u; má»—i endpoint má»™t kiá»ƒu lá»—i.
- **Rule:** REST dÆ°á»›i `/api/v1`; schema request/response khai báº±ng zod trong `packages/shared` â€” FE vÃ  BE cÃ¹ng import, khÃ´ng Ä‘á»‹nh nghÄ©a type trÃ¹ng. Lá»—i tráº£ vá» má»™t envelope duy nháº¥t `{ error: { code, message, details? } }` vá»›i `code` á»•n Ä‘á»‹nh; thÃ´ng Ä‘iá»‡p ngÆ°á»i dÃ¹ng báº±ng tiáº¿ng Viá»‡t.

### AD-11 â€” Web-first, chá»«a Ä‘Æ°á»ng Zalo Mini App `[ADOPTED â€” D11]`

- **Binds:** all frontend (NFR-03, NFR-04)
- **Prevents:** lÃµi frontend phá»¥ thuá»™c API riÃªng cá»§a ZMP â†’ Phase 2 khÃ´ng bá»c Ä‘Æ°á»£c; hoáº·c auth khÃ´ng má»Ÿ Ä‘Æ°á»£c Ä‘Æ°á»ng Zalo ID.
- **Rule:** frontend lÃ  Next.js responsive mobile-first cháº¡y trÃ¬nh duyá»‡t thÆ°á»ng; khÃ´ng gá»i SDK/API riÃªng cá»§a Zalo á»Ÿ lÃµi. Auth dÃ¹ng phiÃªn (cookie httpOnly) qua endpoint `auth` â€” thÃªm OAuth Zalo sau nÃ y lÃ  thÃªm provider, khÃ´ng Ä‘á»•i mÃ´ hÃ¬nh phiÃªn.

### AD-12 â€” PostgreSQL + Prisma lÃ  Ä‘Æ°á»ng dá»¯ liá»‡u duy nháº¥t `[ADOPTED â€” ERD]`

- **Binds:** all
- **Prevents:** truy cáº­p DB báº±ng SQL tay ráº£i rÃ¡c / ORM thá»© hai; schema trÃ´i khá»i ERD Ä‘Ã£ review.
- **Rule:** Prisma schema sinh khá»Ÿi Ä‘áº§u tá»« `docs/ADG_CPQ_ERD.dbml` (logical); khi code tá»“n táº¡i, Prisma schema + migrations lÃ  nguá»“n physical duy nháº¥t â€” Ä‘á»•i logical (thÃªm/bá» báº£ng, Ä‘á»•i quan há»‡) pháº£i Ä‘á»‘i chiáº¿u decision log. Raw SQL chá»‰ trong repository vÃ  pháº£i cÃ³ lÃ½ do ghi kÃ¨m.

### AD-13 â€” TÃ­ch há»£p ngoÃ i qua port/adapter

- **Binds:** F6, F7 (FR-050, FR-063, FR-064)
- **Prevents:** code nghiá»‡p vá»¥ gá»i tháº³ng SDK cá»§a má»™t nhÃ  cung cáº¥p (AI, Zalo, nhÃ  mÃ¡y) â†’ Ä‘á»•i provider pháº£i má»• lÃµi.
- **Rule:** má»—i tÃ­ch há»£p ngoÃ i cÃ³ interface trong module cá»§a nÃ³ (`MockupProvider`, `CustomerDeliveryChannel`, `FactoryDispatchChannel`); nghiá»‡p vá»¥ chá»‰ gá»i interface; adapter cá»¥ thá»ƒ (ká»ƒ cáº£ adapter "manual/download") Ä‘Äƒng kÃ½ qua DI. Provider AI vÃ  kÃªnh nhÃ  mÃ¡y chÆ°a chá»n â€” adapter mock/manual dÃ¹ng trÆ°á»›c.

### AD-14 â€” Sá»Ÿ há»¯u lÆ°á»£t gen mock-up: Ä‘áº¿m báº±ng dÃ²ng, giá»¯ chá»— khi enqueue

- **Binds:** F6 (FR-050, FR-051; D6)
- **Prevents:** API vÃ  worker Ä‘áº¿m lÆ°á»£t hai kiá»ƒu (Ä‘á»‘t lÆ°á»£t khi fail ká»¹ thuáº­t â€” trÃ¡i D6; hoáº·c retry Ä‘áº¿m Ä‘Ã´i); race trÃªn `generation_no`; reset lÆ°á»£t khi thay áº£nh Ä‘Ã¢m vÃ o unique index cÅ©.
- **Rule:** khÃ´ng cÃ³ cá»™t Ä‘áº¿m tay. Enqueue táº¡o ngay má»™t dÃ²ng `quote_item_ai_image` tráº¡ng thÃ¡i `pending` (giá»¯ chá»—, chá»‘ng race); job cáº­p nháº­t thÃ nh `succeeded` hoáº·c `failed_technical`. **Sá»‘ lÆ°á»£t Ä‘Ã£ dÃ¹ng = sá»‘ dÃ²ng khÃ´ng pháº£i `failed_technical` táº¡o sau láº§n thay áº£nh hiá»‡n trÆ°á»ng gáº§n nháº¥t**; `generation_no` tÄƒng Ä‘Æ¡n Ä‘iá»‡u theo háº¡ng má»¥c, khÃ´ng bao giá» tÃ¡i sá»­ dá»¥ng (thay áº£nh khÃ´ng xoÃ¡ dÃ²ng cÅ©). Äiá»u chá»‰nh ERD tÆ°Æ¡ng á»©ng: thÃªm cá»™t `status`, bá» `ai_generation_count`.

### AD-15 â€” PII & thÃ´ng tin xÃ¡c thá»±c cÃ³ ranh giá»›i

- **Binds:** all (NFR-02; Nghá»‹ Ä‘á»‹nh 13/2023/NÄ-CP)
- **Prevents:** PII khÃ¡ch hÃ ng vÃ  bÃ­ máº­t xÃ¡c thá»±c rÃ² rá»‰ qua log/response/báº£ng phá»¥ do má»—i dev tá»± xá»­.
- **Rule:** máº­t kháº©u bÄƒm Argon2id (hoáº·c bcrypt) â€” khÃ´ng bao giá» log/serialize. PII khÃ¡ch (tÃªn, SÄT, Ä‘á»‹a chá»‰) chá»‰ tá»“n táº¡i á»Ÿ báº£ng `customer` vÃ  cÃ¡c cá»™t snapshot trong `quote` â€” khÃ´ng sao chÃ©p sang báº£ng khÃ¡c, khÃ´ng xuáº¥t hiá»‡n trong log á»Ÿ má»i má»©c. HTTPS báº¯t buá»™c má»i mÃ´i trÆ°á»ng trá»« dev local. YÃªu cáº§u xoÃ¡ dá»¯ liá»‡u cÃ¡ nhÃ¢n: xoÃ¡/áº©n á»Ÿ `customer`; snapshot trong bÃ¡o giÃ¡ Ä‘Ã£ phÃ¡t hÃ nh giá»¯ láº¡i phá»¥c vá»¥ nghÄ©a vá»¥ há»£p Ä‘á»“ng â€” chÃ­nh sÃ¡ch retention cá»¥ thá»ƒ lÃ  cÃ¢u há»i phÃ¡p lÃ½ (xem Deferred).

### AD-16 â€” Äá»“ng thá»i trÃªn nhÃ¡p: khoÃ¡ láº¡c quan

- **Binds:** F5 (NFR-10)
- **Prevents:** hai tÃ i khoáº£n cÃ¹ng Ä‘áº¡i lÃ½ má»Ÿ cÃ¹ng má»™t nhÃ¡p vÃ  ghi Ä‘Ã¨ Ã¢m tháº§m láº«n nhau â€” má»—i dev tá»± xá»­ má»™t kiá»ƒu (last-write-wins chá»— nÃ y, lock chá»— kia).
- **Rule:** má»i mutation lÃªn `quote`/`quote_item` á»Ÿ tráº¡ng thÃ¡i nhÃ¡p mang theo `updated_at` client Ä‘Ã£ Ä‘á»c; server so khá»›p trÆ°á»›c khi ghi â€” lá»‡ch thÃ¬ tá»« chá»‘i vá»›i mÃ£ lá»—i chuáº©n `DRAFT_CONFLICT` (409), client táº£i láº¡i. KhÃ´ng dÃ¹ng khoÃ¡ bi quan/khoÃ¡ phiÃªn.

## Consistency Conventions

| Concern | Convention |
| --- | --- |
| Naming DB | Báº£ng/cá»™t `snake_case` Ä‘Ãºng theo ERD dbml; khÃ´ng Ä‘á»•i tÃªn khi sinh Prisma schema (`@@map`/`@map` náº¿u cáº§n) |
| Naming code | TypeScript `camelCase`; class `PascalCase`; DTO háº­u tá»‘ `Dto`; module thÆ° má»¥c `kebab-case` |
| ID | `bigint` autoincrement (theo ERD); serialize ra JSON dáº¡ng string |
| Thá»i gian | LÆ°u `timestamptz` UTC; hiá»ƒn thá»‹ mÃºi giá» `Asia/Ho_Chi_Minh`; format ngÃ y `dd/MM/yyyy` |
| Tiá»n & sá»‘ lÆ°á»£ng | Integer VND (AD-6); quantity `decimal(18,4)`; khÃ´ng dÃ¹ng float cho tiá»n á»Ÿ báº¥t ká»³ táº§ng nÃ o |
| Lá»—i | Envelope AD-10; `code` dáº¡ng `SCREAMING_SNAKE` á»•n Ä‘á»‹nh (vd `PRICE_OUT_OF_RANGE`, `SKU_INCOMPATIBLE`) |
| Tráº¡ng thÃ¡i | Enum trong DB theo ERD; TS dÃ¹ng union type sinh tá»« zod, khÃ´ng hardcode chuá»—i ráº£i rÃ¡c |
| Config | Biáº¿n mÃ´i trÆ°á»ng qua `@nestjs/config` + zod validate lÃºc boot; cáº¥u hÃ¬nh nghiá»‡p vá»¥ (VAT, lÆ°á»£t gen...) Ä‘á»c tá»« `system_setting`, khÃ´ng tá»« env |
| Logging | Log cÃ³ cáº¥u trÃºc (JSON) kÃ¨m `quoteId`/`dealerId` khi cÃ³; khÃ´ng log dá»¯ liá»‡u máº­t (giÃ¡ vá»‘n, gross) á»Ÿ má»©c info |
| i18n | MVP má»™t ngÃ´n ngá»¯ tiáº¿ng Viá»‡t; chuá»—i UI/thÃ´ng bÃ¡o táº­p trung constants, khÃ´ng ráº£i trong JSX/service |
| Test | Unit cho formula-engine + pricing (báº¯t buá»™c); e2e cho pipeline phÃ¡t hÃ nh vÃ  ca báº£o máº­t AD-3/AD-8 |

## Stack

ÄÃ£ xÃ¡c minh phiÃªn báº£n hiá»‡n hÃ nh trÃªn web 03/07/2026 (nguá»“n: [nextjs.org/blog](https://nextjs.org/blog), [github.com/nestjs/nest/releases](https://github.com/nestjs/nest/releases), [prisma.io/blog](https://www.prisma.io/blog/announcing-prisma-orm-7-0-0)).

| Name | Version |
| --- | --- |
| Node.js | **24 LTS (Krypton)** â€” Active LTS Ä‘áº¿n 04/2028, pin `.nvmrc` |
| TypeScript | 5.x |
| Next.js (apps/web) | 16.2.x LTS |
| NestJS (apps/api) | 11.1.x â€” **pin v11**, khÃ´ng nháº£y v12 (ESM, GA Q3/2026) giá»¯a dá»± Ã¡n; Ä‘Ã¡nh giÃ¡ láº¡i sau MVP |
| Prisma ORM | 7.x |
| PostgreSQL | **18.x** â€” greenfield vÃ o major chÃ­n hiá»‡n hÃ nh (GA 09/2025) |
| Queue store | **Valkey 8+** (BSD-3, BullMQ cháº¡y full test suite chÃ­nh thá»©c) hoáº·c Redis 8 (AGPL) |
| BullMQ | báº£n hiá»‡n hÃ nh |
| zod | báº£n hiá»‡n hÃ nh (schemas shared) |
| Playwright (render PDF tá»« template HTML) | báº£n hiá»‡n hÃ nh |
| Object storage (áº£nh, PDF) | S3-compatible â€” Æ°u tiÃªn cloud VN / Garage / SeaweedFS; náº¿u MinIO CE thÃ¬ cháº¥p nháº­n quáº£n trá»‹ CLI-only (Web UI Ä‘Ã£ bá»‹ gá»¡ khá»i báº£n community) `[theo háº¡ táº§ng â€” Deferred]` |
| pnpm | **11.x**, pin qua trÆ°á»ng `packageManager` |

## Structural Seed

```text
adg-cpq/
  apps/
    api/                    # NestJS: cÃ¡c module theo báº£ng Ã¡nh xáº¡ á»Ÿ Design Paradigm
      src/modules/<domain>/ # controller / service / repository / dto per module
      src/common/           # guards (dealer-scope), interceptors (audit), error envelope
    web/                    # Next.js 16 App Router
      app/(dealer)/         # pipeline bÃ¡o giÃ¡ â€” mobile-first
      app/admin/            # quáº£n trá»‹ â€” desktop-first
  packages/
    shared/                 # zod schemas, types, error codes, constants tiáº¿ng Viá»‡t
    formula-engine/         # parser AST + money (roundVnd) + golden-test harness
  docs/                     # tÃ i liá»‡u nghiá»‡p vá»¥ v2 (Ä‘Ã£ cÃ³)
  docker-compose.yml        # api, web, worker(=api cháº¿ Ä‘á»™ worker), postgres, redis, minio
```

Triá»ƒn khai & mÃ´i trÆ°á»ng `[ASSUMPTION â€” háº¡ táº§ng chÆ°a chá»‘t]`:

```mermaid
graph LR
    subgraph "Má»—i mÃ´i trÆ°á»ng (dev / staging / prod)"
        WEB["web (Next.js)"] --> API["api (NestJS)"]
        API --> PG[("PostgreSQL 16<br/>backup háº±ng ngÃ y")]
        API --> RD[("Redis")]
        WK["worker (api, WORKER=1)"] --> RD
        WK --> PG
        API --> S3[("MinIO / S3<br/>áº£nh + PDF")]
        WK --> AI["AI mockup provider (adapter)"]
        WK --> ZA["Zalo OA (adapter)"]
        WK --> FT["Cá»•ng nhÃ  mÃ¡y (adapter/manual)"]
    end
```

- 3 mÃ´i trÆ°á»ng dev / staging / prod, Ä‘Ã³ng gÃ³i Docker Compose; prod má»™t mÃ¡y Ä‘á»§ cho NFR-05 (50 Ä‘á»“ng thá»i) â€” khÃ´ng cáº§n phÃ¢n tÃ¡n.
- NhÃ  cung cáº¥p háº¡ táº§ng (server cÃ´ng ty vs cloud VN) lÃ  **cÃ¢u há»i má»Ÿ**, pháº£i chá»‘t trÆ°á»›c pilot; kiáº¿n trÃºc khÃ´ng phá»¥ thuá»™c lá»±a chá»n nÃ y.

## Capability â†’ Architecture Map

| Capability | Lives in | Governed by |
| --- | --- | --- |
| F1 Catalog & sáº£n pháº©m | `catalog` | AD-1, AD-9, AD-12 |
| F2 Linh kiá»‡n & tÆ°Æ¡ng thÃ­ch | `component` + `formula-engine` | AD-1, AD-4 |
| F3 Váº­t tÆ° & BOM | `material` + `formula-engine` | AD-4, AD-6 |
| F4 Äá»‹nh giÃ¡ | `pricing` | AD-2, AD-3, AD-6 |
| F5 Pipeline bÃ¡o giÃ¡ | `quote`, `customer` | AD-2, AD-5, AD-8 |
| F6 Mock-up AI | `mockup` (+queue) | AD-7, AD-13 |
| F7 PhÃ¡t hÃ nh & gá»­i | `quote`, `delivery` (+queue) | AD-5, AD-7, AD-13 |
| F8 Quáº£n trá»‹ & audit | `auth`, `dealer`, `audit` | AD-8, AD-9 |
| F9 Import & golden tests | `import-tools` + `formula-engine` | AD-4 |
| Seed dá»¯ liá»‡u ban Ä‘áº§u (váº­t tÆ° + Ä‘Æ¡n giÃ¡, SKU linh kiá»‡n, ~300 Ä‘áº¡i lÃ½ + gross, admin bootstrap) | `import-tools` â€” cÃ´ng cá»¥ import CSV/Excel, build trong MVP trÆ°á»›c pilot | AD-9, AD-12 |

## Deferred

| Quyáº¿t Ä‘á»‹nh | VÃ¬ sao chá» Ä‘Æ°á»£c |
| --- | --- |
| NhÃ  cung cáº¥p AI mockup | Adapter interface (AD-13) cÃ¡ch ly; chá»n khi test cháº¥t lÆ°á»£ng áº£nh + chi phÃ­/láº§n â€” cáº§n trÆ°á»›c khi build F6 tháº­t |
| KÃªnh + format gá»­i nhÃ  mÃ¡y tá»± Ä‘á»™ng | `[OPEN O4]`; MVP xuáº¥t file chuáº©n qua adapter manual |
| Háº¡ táº§ng deploy (server cÃ´ng ty vs cloud VN) | Docker hoÃ¡ nÃªn chuyá»ƒn Ä‘Æ°á»£c; chá»‘t trÆ°á»›c pilot |
| CI/CD chi tiáº¿t | Sau khi repo cÃ³ code; tá»‘i thiá»ƒu: lint + test + build má»—i PR |
| Bá»c Zalo Mini App | Phase 2; AD-11 Ä‘Ã£ giá»¯ Ä‘Æ°á»ng |
| GiÃ¡ trá»‹ cáº¥u hÃ¬nh O1/O2/O3 | Nháº­p qua admin khi cÃ´ng ty ban hÃ nh |
| Chi tiáº¿t physical DB (index phá»¥, partition) | Code + dá»¯ liá»‡u tháº­t quyáº¿t Ä‘á»‹nh; ERD logical Ä‘Ã£ Ä‘á»§ cho MVP |
| ChÃ­nh sÃ¡ch retention/xoÃ¡ PII trong snapshot Ä‘Ã£ phÃ¡t hÃ nh (Nghá»‹ Ä‘á»‹nh 13) | Cáº§n Ã½ kiáº¿n phÃ¡p lÃ½; AD-15 Ä‘Ã£ khoanh ranh giá»›i dá»¯ liá»‡u â€” chá»‘t trÆ°á»›c pilot |
| ÄÆ°á»ng upload áº£nh (qua API vs presigned direct-to-storage) | Quyáº¿t khi build FR-042; khÃ´ng áº£nh hÆ°á»Ÿng AD nÃ o |
| Backup/alerting cá»¥ thá»ƒ (táº§n suáº¥t, cÃ´ng cá»¥ giÃ¡m sÃ¡t) | Chá»‘t cÃ¹ng lá»±a chá»n háº¡ táº§ng, trÆ°á»›c pilot |
