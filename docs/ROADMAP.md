# Roadmap — DB Schema Analyzer

## Current Version: v1.1.0

---

## ✅ Completed

- 50+ deterministic rules across 8 categories
- Health score (A–F) with domain multipliers (Healthcare, Finance, E-commerce, SaaS)
- Platform mismatch detection (PostgreSQL, MySQL, SQL Server, Oracle, SQLite)
- ER Diagram tab — pure SVG, Sugiyama layout, FK relationships, zoom/pan, SVG/PNG download
- Multi-schema support (up to 5), Comparison tab
- Schema Intelligence card (split/merge recommendations)
- PDF export (9 sections + About This Analysis page)
- docs.html — full rule reference, platform guide, scoring formula
- Sample schemas: E-commerce, Healthcare, SaaS, Legacy ERP, Badly Designed, Multi-Schema
- Zero backend — 100% client-side, no data transmission

---

## 🔴 Phase 1 — Free Product Consolidation (Next)

### Task 2: ER Diagram Edit Mode B — Drag Tables
Drag table boxes to reposition them in the ER diagram. Visual only — does not change DDL.
Pan is already wired via mouse events; table-level drag is a small extension.

### Task 3: ER Diagram Edit Mode A — Inline Name Edit
Click a table or column name in the diagram → edit inline → syncs back to DDL → re-analysis triggered.
Requires "regenerate DDL from diagram state" function.

### Task 4: Migration Script Generator
One button that compiles all Critical + High findings into a single `migration.sql` file
with ALTER TABLE fixes in the correct order (PKs → FKs → indexes).

### Task 5: Schema Diff
Paste v1 + v2 of a schema → see what changed, what new issues were introduced,
what was fixed, and the score delta. Client-side diff, no backend required.

---

## 🟠 Phase 2 — Protect + Capture Demand

### Task 6: JS Obfuscation
Run the JS through `javascript-obfuscator` to protect the 50+ rule engine.
Reserve all onclick function names to prevent button breakage.

### Task 7: Email Waitlist Capture
Add a "Pro coming soon" banner with email capture to the live site.
Builds the waitlist before the paid tier launches.

### Task 8: Compliance Rule Sets (HIPAA / PCI-DSS / GDPR)
Domain-specific compliance checks beyond current domain multipliers:
- HIPAA: PHI column encryption, audit trail required, minimum necessary access
- PCI-DSS: No card storage, tokenisation required, network segmentation hints
- GDPR: Right to erasure (soft delete), consent tracking, data retention columns

### Task 9: Export Findings as CSV
One-click export of all findings as CSV for Jira/Linear bulk import.

### Task 10: Copy All Fixes Per Section
In Summary tab, a "Copy all fixes" button per section generates a bulk SQL block.

---

## 🟢 Phase 3 — Monetisation

### Task 11: Pro Tier ($19/mo)
Gate: score history, shareable results link, CSV export, compliance rules, schema diff.

### Task 12: Team Tier ($79/mo)
Gate: team workspace, custom rule builder, CI/CD GitHub Action snippet.

### Task 13: Shareable Results Link
Encode analysis result in URL hash (base64 compressed). No backend required.

### Task 14: Score History (localStorage)
Remember last 30 analysis runs. Show trend: `Last: 67 → Today: 78 ↑11`.

### Task 15: Dark/Light Theme Toggle
CSS variable swap. Broader appeal for light-mode users.

---

## 🟡 Lower Priority

| # | Task |
|---|------|
| 16 | GitHub Action CI/CD snippet (YAML for PR gates) |
| 17 | Print-friendly CSS (`@media print`) |
| 18 | Update docs.html with new rules and features |
| 19 | Regenerate PPT decks with v1.1 features |
| 20 | Redundant index detection rule |
| 21 | Nullable unique constraint rule |
| 22 | Column/table search/grep across schema |
| 23 | Orphan table dedicated report |
| 24 | Data classification tagging (PII/PHI/Financial/Public) |
| 25 | Per-table drill-down in Issues tab |

---

## 🔵 v2 — Backend Required

| # | Feature |
|---|---------|
| 26 | Direct DB connection (PostgreSQL/MySQL via connection string) |
| 27 | Data profiling (null rates, cardinality, dead column detection) |
| 28 | Team features + Jira/Linear integration |
| 29 | Schema history + versioning |
| 30 | React + TypeScript migration |
| 31 | SSO + Enterprise auth (SAML, Okta) |
| 32 | REST API Analyzer — separate product (OpenAPI YAML input) |

---

## Monetisation Strategy

**Free forever:** Core analysis (50+ rules), ER diagram, PDF export, all sample schemas

**Pro ($19/mo):** Score history, shareable link, CSV export, compliance rules, schema diff

**Team ($79/mo):** Team workspace, custom rules, CI/CD integration, priority support

**Enterprise ($2,000/yr):** Self-hosted Docker, custom rule sets, SSO, audit logs, SLA

Target market: Healthcare, Finance, SaaS engineering teams — highest WTP for compliance features.
