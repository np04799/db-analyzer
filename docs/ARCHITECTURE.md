# Architecture — DB Schema Analyzer

## Overview

DB Schema Analyzer is a **single-file, zero-backend, client-side web application**.
The entire application — HTML, CSS, and JavaScript — lives in `index.html` (~180KB).

```
Browser
  └── index.html (HTML + CSS + JS)
        ├── DDL Parser
        ├── Rule Engine (50+ rules)
        ├── Scoring Engine
        ├── Schema Intelligence
        ├── ER Diagram Engine (pure SVG)
        ├── TAB_RENDERERS registry
        └── PDF Builder (jsPDF 2.5.1 via CDN)
```

No server. No database. No build step. No framework.

---

## Data Flow

```
User Input (DDL + queries + overview + platform)
       ↓
parseDDL() — balanced-paren walker
       ↓
tables[] — structured representation
       ↓
runRules(tables, queries, overview, domain)
       ↓
findings[] — severity, category, title, description, fix, tags
       ↓
scoreSecs(findings) → sectionScores{}
calcScore(sectionScores) → healthScore (0–100)
       ↓
generateSchemaInsight(result) → insightHTML
detectPlatformMismatch(tables, platform) → mismatches[]
       ↓
renderAll(result) → populates all 6 result tabs
buildERDiagram(result) → SVG string
```

---

## DDL Parser

### `extractTableBody(ddl, start)`
A balanced-parenthesis walker that correctly handles:
- Nested function calls: `DEFAULT gen_random_uuid()`, `DEFAULT NOW()`
- Inline `REFERENCES table(column)` syntax
- `CREATE TABLE IF NOT EXISTS`
- Quoted identifiers: backtick, double-quote, square bracket

### `parseDDL(ddl) → tables[]`
Returns an array of table objects:
```javascript
{
  name: 'users',
  columns: [
    {
      name: 'id',
      type: 'SERIAL',
      nullable: false,
      hasPK: true,
      hasUnique: false,
      hasDefault: true,
      autoInc: true,
      rawType: 'SERIAL'  // preserves original with (n) for length checks
    }
  ],
  constraints: {
    pk: true,
    fks: [{ col: 'tenant_id', ref: 'tenants', onDelete: 'CASCADE' }],
    uniques: ['email'],
    checks: ['status IN (...)']
  }
}
```

---

## Rule Engine

### `runRules(tables, queries, overview, domain) → findings[]`

50+ rules grouped by category. Each rule uses the `add()` helper:
```javascript
add(severity, category, title, table, description, fix, tags)
```

**Rule groups:**
1. Per-table structural rules (R01–R07)
2. Per-column security rules (R09–R14)
3. Per-column naming rules (R20–R26, N8)
4. Per-column integrity rules (R05, N6)
5. Per-column normalization rules (R15–R17, N7)
6. Per-column operational rules (R33–R34, N9)
7. Per-column performance rules (R38–R41)
8. New rules — N1–N9 (per-table/column)
9. Cross-table relationship rules (R27–R32, N2, N3, N5)
10. Domain/SaaS rules (N4, N10)
11. Query analysis rules (R42–R44, if queries submitted)
12. Platform mismatch detection (advisory, no score impact)

---

## Scoring Engine

### `scoreSecs(findings) → sectionScores{}`
Maps findings to sections and calculates section scores:
```javascript
Section Score = max(0, 100 − Σ(penalty × domain_multiplier))
```

### `calcScore(sectionScores) → healthScore`
Weighted composite:
```javascript
(Integrity × 0.25) + (Normalization × 0.20) + (Schema & Arch × 0.20) +
(Security × 0.15) + (Indexing × 0.10) + (Relationships × 0.10)
```

---

## Tab System — TAB_RENDERERS Registry

```javascript
const TAB_RENDERERS = {
  ts:    (r) => renderSummaryTab(r),
  tc:    (r) => renderComparisonTab(r),
  tdiag: (r) => renderDiagramTab(r),
};

function swTab(tp, btn) {
  // ... activate tab UI ...
  if (result && TAB_RENDERERS[tp]) TAB_RENDERERS[tp](result);
}
```

Every registered tab re-renders fresh on every activation — no stale DOM state.

**All render functions are self-contained** — they define all required variables
internally and do not depend on outer scope variables.

---

## ER Diagram Engine

### `buildERDiagram(result) → { svg, svgW, svgH, count }`

**Layout strategies:**
- ≤50 tables → `layoutSugiyama()` — topological layered layout
- 51+ tables → `layoutClusters()` — group by schema, grid within clusters

**Edge types:**
- Explicit FK (from `constraints.fks`) → solid line, 1.8px, opacity 0.75
- Inferred FK (column ends in `_id`, name matches a table) → dashed line, 1.2px, opacity 0.45
- No duplicate edges — explicit trumps inferred for same table pair

**Column icons:**
- 🔑 PK columns — gold (#D29922)
- 🔗 FK columns — schema colour
- ◈ UNIQUE columns — green (#3FB950)

**Interactions:** scroll to zoom (0.2×–3×), drag to pan
**Export:** SVG (vector), PNG (2× retina via canvas)

---

## Schema Intelligence

### `generateSchemaInsight(result) → insights[]`

Analyses the overall schema shape and returns recommendations:
- **Single schema:** domain mixing, future schema needs, quality gaps
- **Multi-schema:** cross-schema comparisons, score deltas, quality gaps, duplicate tables

Rendered as the 🧠 Schema Intelligence card in the Summary tab.

---

## Platform Mismatch Detection

### `detectPlatformMismatch(tables, platform) → mismatches[]`

Checks every column type against the selected platform's incompatible type list.
Result is attached to `result.platformMismatches` and rendered as an advisory
warning banner in the Summary tab. No score impact.

---

## PDF Builder

### `buildPDF(result) → jsPDF document`

Uses jsPDF 2.5.1 (loaded from CDN). Sections:
1. Cover page (title, score, grade, date)
2. About This Analysis (methodology, catches vs limitations)
3. S1 — Business Context
4. S2 — Executive Summary (section scores, charts)
5. S3 — Schema Inventory
6. S4 — Security Findings
7. S5 — Integrity Findings
8. S6 — Performance Findings
9. S7 — Multi-Schema Comparison
10. Appendix — Full findings list

---

## State Management

```javascript
let schemas = [];   // array of schema objects (DDL, name, platform, queries)
let result = null;  // last analysis result — populated by runAnalysis()
let scCtr = 0;      // schema counter for unique IDs
```

No localStorage, no cookies, no session storage.
All state is in-memory and cleared on page refresh.

---

## File Structure

```
index.html
├── <style>          CSS (custom properties, dark theme)
├── <body>           HTML structure
│   ├── .hd          Header bar
│   ├── .main        Main content
│   │   ├── Step 1   Business overview + platform selector
│   │   ├── Step 2   Schema cards (DDL textarea + file upload)
│   │   ├── Step 3   Analyze button + progress bar
│   │   └── #res     Results area
│   │       ├── .tbs Tab buttons
│   │       └── .tp  Tab panels (ti, ts, tsol, tc, tr, tdiag)
│   └── footer       HIW banner, footer columns
└── <script>
    ├── Constants & config
    ├── parseDDL()
    ├── extractTableBody()
    ├── runRules()         ← 50+ rules
    ├── scoreSecs()
    ├── calcScore()
    ├── PLATFORM_TYPES
    ├── detectPlatformMismatch()
    ├── buildERDiagram()
    ├── layoutSugiyama()
    ├── layoutClusters()
    ├── renderDiagramTab()
    ├── generateSchemaInsight()
    ├── renderSchemaInsight()
    ├── TAB_RENDERERS
    ├── swTab()
    ├── renderAll()
    ├── renderSummaryTab()
    ├── renderComparisonTab()
    ├── renderSolutionsTab()
    ├── calcRoadmap()
    ├── buildPDF()
    ├── SAMPLES (6 sample schemas)
    ├── loadSample()
    ├── runAnalysis()
    └── init
```
