# PDF Report — DB Schema Analyzer

## Overview

The PDF export generates a professional stakeholder report using **jsPDF 2.5.1**
loaded from CDN. All PDF generation is client-side — no data leaves the browser.

**Trigger:** "Export PDF Report" button in the results area.
**Function:** `buildPDF(result)` → downloads `schema_analysis_report.pdf`

---

## Report Structure

### Cover Page
- Tool name, version badge
- Health Score (large, colour-coded by grade)
- Grade (A–F)
- Schema name(s)
- Date generated
- Tagline: "Privacy-first schema analysis — no data transmitted"

### Page 2: About This Analysis
Explains the methodology to non-technical stakeholders:
- What the tool is (static linter, not live DB analysis)
- Two-column layout: ✅ What it reliably catches vs ✗ v1 Limitations
- "How to rely on it" highlighted box
- Methodology note (balanced-paren parser, zero data transmission)

### S1 — Business Context
- Business overview text (from Step 1)
- Domain detected (if any) with multiplier explanation
- Schema inventory (table names submitted)

### S2 — Executive Summary
- Section score bars (colour-coded per category)
- Finding count by severity (Critical/High/Medium/Low)
- Grade and total health score

### S3 — Schema Inventory
- Per-schema table list with column counts
- FK relationship summary

### S4 — Security Findings
- All Critical and High security findings
- Each finding: title, table, description, SQL fix

### S5 — Integrity & Normalization Findings
- Critical/High integrity and normalization findings

### S6 — Performance & Relationship Findings
- High performance and relationship findings

### S7 — Multi-Schema Comparison *(if 2+ schemas)*
- Side-by-side score comparison table
- Score delta and grade for each schema

### Appendix — Full Findings List
- Complete findings sorted by severity
- Every finding: severity, category, title, table, fix

---

## PDF Builder Helpers

```javascript
function sf(r, g, b)        // setFillColor shorthand
function st(r, g, b)        // setTextColor shorthand
function secBanner(n, title) // section header banner
function newPage(addHeader)  // add page with optional running header
function bodyTxt(text, width) // paragraph text with line wrapping
function subHdr(text)        // sub-section header
function setPP(n, label)     // update progress bar during build
```

---

## Colour Palette (jsPDF RGB arrays)

```javascript
const C = {
  bg:   [13, 17, 23],    // Dark background
  s1:   [22, 27, 34],    // Surface 1
  s2:   [28, 35, 51],    // Surface 2
  ac:   [45, 140, 255],  // Accent blue
  ok:   [63, 185, 80],   // Green (good)
  cr:   [248, 81, 73],   // Red (critical)
  hi:   [240, 136, 62],  // Orange (high)
  me:   [210, 153, 34],  // Yellow (medium)
  lo:   [63, 185, 80],   // Green (low)
  pv:   [163, 113, 247], // Purple (normalization)
  tx:   [230, 237, 243], // Primary text
  t2:   [139, 148, 158], // Secondary text
  t3:   [110, 118, 129], // Tertiary text
  b:    [48, 54, 61],    // Border
  acbg: [20, 40, 70],    // Accent background
};
```

---

## Section Colour Map (DC2 for PDF bars)

```javascript
const DC2 = {
  'Schema & Architecture': C.ac,   // Blue
  'Keys & Integrity':      C.cr,   // Red
  'Normalization':         C.pv,   // Purple
  'Security':              [255, 107, 107], // Light red
  'Indexing & Performance': C.hi,  // Orange
  'Relationships':         C.lo,   // Green
  'Query Analysis':        C.me,   // Yellow
};
```

---

## Progress Bar

During PDF generation, a progress bar shows build stages:
```
S1 — Business context... (12%)
S2 — Executive summary... (25%)
S3 — Schema inventory... (40%)
S4 — Security findings... (55%)
S5 — Integrity findings... (70%)
S6 — Performance findings... (80%)
S7 — Comparison... (88%)
Appendix... (95%)
Done! (100%)
```
