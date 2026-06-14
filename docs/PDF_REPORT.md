# PDF Report — DB Schema Analyzer v1.0

The exported PDF is a structured A4 report generated client-side using **jsPDF 2.5.1**. It contains a cover page, 9 numbered sections, and an appendix.

---

## Report Structure

### Cover Page

| Element | Content |
|---|---|
| Title | "Schema Design Review & Best Practice Analysis" |
| Subtitle | Schema names submitted |
| Generated date | Current date |
| KPI tiles | Health Score, Tables, Issues, Critical count, Domain |
| Privacy notice | "Privacy First — No data stored or transmitted" |

---

### S1 — Business Context

- Business overview text (as entered)
- Auto-detected domain with weighting note
- Schema inventory table (name, platform, tables, columns, findings, score)
- Analysis scope summary (v1: Schema Only)
- v2 exclusions notice (data profiling, DB connection)

---

### S2 — Executive Summary

- Finding counts by severity (Critical / High / Medium / Low)
- Dimensional score bars for all 6 sections
- Colour-coded progress bars per section

---

### S3 — Schema & Architecture

- Entity inventory table (table name, schema, columns, PK status, worst finding)
- All Architecture / Naming / Operational findings with descriptions and SQL fixes

---

### S4 — Keys & Integrity

- Integrity checklist matrix per table (PK ✓/✗, FK gaps, Email UNIQUE, Data Type issues)
- All Integrity and Relationship findings with SQL fixes

---

### S5 — Normalization & Constraints

- 1NF / 2NF / 3NF assessment table (PASS / VIOLATIONS per form)
- Affected table names per normal form
- All Normalization findings with SQL fixes

---

### S6 — Query Analysis

- **Skipped notice** if no queries were submitted
- Query finding count and schema breakdown
- All Query findings (R41–R43) with SQL fixes

---

### S7 — Schema Comparison

- **Skipped** if only 1 schema submitted
- Side-by-side comparison table: Health %, Tables, Cols, Critical, High, Medium, Low, Total

---

### S8 — Issue Register

- **Complete list of all findings** sorted by severity (Critical → High → Medium → Low)
- Each entry includes: severity badge, title, schema › table, description, SQL fix (in green code block)

---

### S9 — Remediation Roadmap

- **Phase 1 — Immediate Critical Fixes** (Week 1)
- **Phase 2 — High Priority Improvements** (Weeks 2–3)
- **Phase 3 — Medium Term Refactoring** (Month 1–2)
- **Phase 4 — Optimization & Hardening** (Month 2–3)
- v2 data profiling notice

---

### Appendix

- Full list of all 40+ rules applied (R01–R43)
- v2 preview: data profiling, DB connection, query workload, schema history, team features

---

## PDF Design Specs

| Property | Value |
|---|---|
| Format | A4 (210mm × 297mm) |
| Margins | 18mm left, 18mm right |
| Font | Helvetica (built-in) |
| Code font | Courier (built-in) |
| Colour scheme | Dark background (#0D1117) with branded accent colours |
| Page header | Sticky: "DB SCHEMA ANALYSIS REPORT — v1.0" + schema names + page number |
| Page footer | "Privacy First — No data stored or transmitted" + Health Score |
| File name | `DB_Schema_Analysis_{SchemaNames}_{YYYY-MM-DD}.pdf` |

---

## Generating the PDF

Click **Export PDF Report** in the results header after completing an analysis. The PDF is generated entirely in the browser — no server upload occurs.

**Generation time:** ~2–5 seconds depending on finding count.

**Note:** PDF generation requires the jsPDF CDN script to be loaded. Works offline only if the CDN was previously cached by the browser.
