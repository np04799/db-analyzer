# Changelog — DB Schema Analyzer

All notable changes to this project are documented here.
Format: `## [version] — YYYY-MM-DD`

---

## [1.1.0] — 2026-06-23

### Added
- **10 new analysis rules** (N1–N10):
  - N1: `status`/`type`/`state` VARCHAR column without CHECK constraint (Medium)
  - N2: Nullable FK column without ON DELETE rule (Medium)
  - N3: Self-referencing FK not declared — `parent_id`, `manager_id` (Medium)
  - N4: Missing `tenant_id`/`org_id` in SaaS domain (High)
  - N5: Junction/pivot table missing composite PRIMARY KEY (Medium)
  - N6: DECIMAL/NUMERIC without explicit precision and scale (Medium)
  - N7: Comma-separated values stored in TEXT/VARCHAR column — 1NF (High)
  - N8: BOOLEAN column missing `is_`/`has_`/`can_` prefix (Low)
  - N9: Missing `created_by`/`updated_by` on transactional tables (Low)
  - N10: No audit table detected in Healthcare/Finance/Banking domain (High)

### Fixed
- Pre-existing bug: plaintext password rule (`R09`) failed to fire because
  the DDL parser strips `(n)` from `VARCHAR(n)`. Rule now uses `col.rawType`
  fallback — any password column without a 60+ char type is flagged.

---

## [1.0.5] — 2026-06-23

### Added
- Multi-Schema sample (🔀 purple button): Auth Service (92/A) vs Billing Service (62/D)
  — pre-loaded with 2 schemas to test Comparison tab and Schema Intelligence
- ER Diagram: explicit FK relationships shown as solid lines with arrowheads
- ER Diagram: inferred FK relationships (via `_id` naming) shown as dashed lines
- ER Diagram: FK column label shown at midpoint of each relationship line
- ER Diagram: legend in toolbar (solid = Explicit FK, dashed = Inferred FK)
- No duplicate edges — explicit FK always trumps inferred FK for same table pair

### Fixed
- SaaS sample DDL: added proper `REFERENCES` clauses (5 explicit FKs now visible in diagram)
- E-commerce sample DDL: added `REFERENCES` on orders/order_items (3 explicit FKs)

---

## [1.0.4] — 2026-06-22

### Added
- 📊 ER Diagram tab — pure SVG rendering, no external library, fully offline
  - Sugiyama layered layout for ≤50 tables
  - Cluster layout for 51+ tables (grouped by schema)
  - Table boxes with PK 🔑 (gold), FK 🔗 (blue), UNIQUE ◈ (green) column icons
  - Schema colour-coded (multi-schema support)
  - Zoom/pan: scroll to zoom, drag to pan
  - Download as SVG (vector) or PNG (2× retina via canvas)
  - Registered in TAB_RENDERERS — re-renders on every tab click
- 💀 Badly Designed sample schema (red button)
  - 4 intentionally broken tables, 49 findings, score 52/F, 6 Critical
  - Auto-sets business overview + platform to Generic SQL
  - Tooltip: "Intentionally bad schema — use for demo or testing"

---

## [1.0.3] — 2026-06-20

### Added
- Platform mismatch warning banner in Summary tab
  - `detectPlatformMismatch()` checks all columns against selected platform
  - Orange advisory banner — zero score impact
  - Covers all 6 platforms: PostgreSQL, MySQL, SQL Server, Oracle, SQLite, Generic SQL
  - Inferred FK detection: `_id` columns matched to table names
- "How this analysis works" collapsible section above results tabs
- "About This Analysis" page added to PDF report (before S1)
- 📖 Documentation link added to footer → `/docs.html`
- `docs.html` — full documentation page with:
  - 26 sidebar navigation links with scrollspy
  - 11 rule tables, 69 rule rows
  - All 6 platform sections (native vs incompatible types)
  - Health score formula, grade scale, domain multiplier table
  - v1 limitations + v2 roadmap column
  - FAQ (7 questions)

---

## [1.0.2] — 2026-06-19

### Added
- PPT presentations: 9-slide full deck + 1-page summary (zipped, downloadable via HIW heading)
- "How It Works" heading links to PPT zip download

### Fixed
- Critical blank-tab bug: `renderSummaryTab` was referencing `DC`, `dimRows`,
  `ovBlock`, `tblRows` that were orphaned in `renderAll` scope after registry refactor.
  All 4 variables moved into `renderSummaryTab` body.

---

## [1.0.1] — 2026-06-18

### Added
- TAB_RENDERERS registry — Summary and Comparison tabs re-render on every activation
- Schema Intelligence card (split/merge recommendations, domain mixing detection)
- Severity pill click-through — pills in Summary/Comparison tab filter Issues tab
- Platform lock on DDL entry — dropdown disables once schema is entered
- Multi-schema support (up to 5 schemas), Comparison tab side-by-side analysis
- Sample schemas: E-commerce, Healthcare, SaaS, Legacy ERP

### Changed
- Single-column layout (nav removed), font size 16px base
- Summary/Comparison cards made permanently expanded (non-collapsible)
- How It Works horizontal pipeline banner above footer
- Footer 3-column layout with Documentation link

---

## [1.0.0] — 2026-06-15

### Initial Release
- 40+ deterministic rules across 8 categories
- Health score (weighted composite, A–F grade) with domain multipliers
- Issues tab with severity filter and collapsible categories
- Solutions tab with effort labels and SQL fixes
- Roadmap tab (4-phase remediation plan)
- PDF export (9 sections)
- Platform selector (PostgreSQL, MySQL, SQL Server, Oracle, SQLite, Generic SQL)
- DDL file upload (.sql) and query input
- Zero backend — 100% client-side, no data transmission
