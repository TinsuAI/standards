# Glossary — TinsuAI cross-product

Customs-compliance domain terms used across all TinsuAI products.
Per-product glossaries (e.g. `data-hub/.ai/GLOSSARY.md`) MAY add
product-specific terms, but the entries below are canonical for
shared concepts.

## Customs-compliance domain (Vietnam)

- **HQ** — Hải quan (customs authority).
- **DNCX** — Doanh nghiệp chế xuất (export-processing enterprise).
  Manufacturing factories with bonded import-for-export status under
  Vietnam customs law. Customers of the agencies that use TinsuAI
  products.
- **BCQT** — Báo Cáo Quyết Toán (annual customs settlement report).
  Required of every DNCX. Aggregates a year of customs declarations
  + ERP material movement into Mẫu 15 / 15a / 16 forms.
- **CO** — Certificate of Origin / Giấy chứng nhận xuất xứ.
  Per-shipment customs document attesting goods origin for
  trade-agreement preferential tariffs.
- **BCCT** — Báo Cáo Cập Nhật Tờ khai (customs declaration registry).
  Excel listing of all import/export declarations for a DNCX in a
  year. Critical input for both BCQT settlement and CO origin
  verification.
- **NXT** — Nhập Xuất Tồn (input/output/balance). ERP-side inventory
  movement summary by material code. Used for cross-checking customs
  declarations against actual material flow.
- **Danh Mục** — Material registry / catalog. List of NVL/SP/BTP
  used by a DNCX, with HS codes + customs codes mapped.
  - **NVL** — Nguyên Vật Liệu (raw materials).
  - **SP** — Sản Phẩm (finished products).
  - **BTP** — Bán Thành Phẩm (semi-finished products / WIP).
    Subdivided: BTP_SX (self-produced), BTP_NM (purchased and
    declared as import).
- **BOM** — Bill of Materials. Product structure showing what raw
  materials / sub-assemblies go into each finished product.
  Required for Mẫu 16.
- **Mẫu 15 / 15a / 16** — Settlement report forms per Thông tư
  39/2018/TT-BTC. Mẫu 15 = raw materials. Mẫu 15a = finished
  products. Mẫu 16 = norms / consumption rates per product.
- **TT 39/2018** — Thông tư 39/2018/TT-BTC. Regulatory basis for
  BCQT format.
- **TT 121/2025** — Thông tư 121/2025. Newer regulation; introduces
  Mẫu 15a column split + Mẫu 16 separate norms for re-imported
  products.
- **Tờ khai** — Customs declaration. Each import/export shipment
  files one. Identified by `declaration_no`.

## Product-portfolio terms

- **Data Hub** — TinsuAI product owning shared HQ-data tier (BCCT,
  Danh Mục, BOM). Provisional name; may rename at product launch.
- **BCQT-System** — TinsuAI product producing annual settlement
  reports. Read-only consumer of Data Hub.
- **CO-System** — TinsuAI product producing per-shipment origin
  certificates. Read-only consumer of Data Hub for HQ-data; may
  write per-shipment data via Data Hub's API depending on the
  current MVP scope (see each product's `.ai/DECISIONS.md` for the
  current ownership boundary, which has been narrowed and broadened
  multiple times).
- **Agency** — A customs-compliance services firm; the direct user
  of TinsuAI products. Serves multiple DNCXs as customers.
