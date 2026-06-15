# Architecture — DB Schema Analyzer v1.0

## Overview

DB Schema Analyzer is a **single-file, client-side web application**. All analysis logic runs in the user's browser — no backend server, no database, no API calls. The entire application is one `index.html` file (~131 KB) served as a static asset from Vercel.

```
┌─────────────────────────────────────────────────────────┐
│                     Browser (Client)                     │
│                                                         │
│  ┌──────────┐   ┌──────────────┐   ┌────────────────┐  │
│  │  UI/UX   │──▶│  DDL Parser  │──▶│  Rule Engine   │  │
│  │ (HTML/   │   │ (parseDDL)   │   │ (runRules)     │  │
│  │  CSS/JS) │   └──────────────┘   └───────┬────────┘  │
│  │          │                              │            │
│  │          │   ┌──────────────┐   ┌───────▼────────┐  │
│  │          │◀──│  PDF Builder │◀──│ Scoring Engine │  │
│  │          │   │  (jsPDF)     │   │ (scoreSecs)    │  │
│  └──────────┘   └──────────────┘   └────────────────┘  │
└─────────────────────────────────────────────────────────┘
         │
         │ Static file only
         ▼
┌─────────────────┐
│  Vercel CDN     │
│  (index.html)   │
└─────────────────┘
```

---

## Page Structure (v1.0 current)

```
Header bar (sticky)
  └── Logo · v1.0 badge · Rule Engine badge · Privacy First badge

Overview strip (full width)
  └── "What is DB Schema Analyzer?" — heading, description, badges
  └── Horizontal steps: ① Business Overview → ② Schemas → ③ Analyze

Main content (single column, full width)
  └── Step 1 — Business Overview + domain detection + platform selector (lockable)
  └── Step 2 — Schema cards (up to 5) + DDL textarea + file upload + query input
  └── Step 3 — Analysis options + progress bar
  └── Results — Score strip + tabs (Issues · Summary · Solutions · Comparison · Roadmap)

How It Works banner
  └── 5-step horizontal pipeline with arrows

Footer (3-column)
  └── DB Schema Analyzer tagline
  └── What It Analyzes (6 categories)
  └── Supported Platforms (6 DBs)
  └── Bottom bar: version info + Privacy First badge
```

---

## Module Breakdown

### 1. DDL Parser (`parseDDL`, `extractTableBody`)

**Purpose:** Converts raw SQL DDL into a structured JavaScript object tree.

**Key design:** Uses balanced-paren walking instead of regex to correctly handle:
- `DEFAULT gen_random_uuid()` — nested parens in defaults
- `DEFAULT NOW()` — function calls in defaults
- Inline `REFERENCES table(id)` — parens inside column definition
- `CREATE TABLE IF NOT EXISTS`
- Quoted identifiers: `` `name` ``, `"name"`, `[name]`

**Output:**
```js
[{
  name: 'users',
  columns: [{ name, type, nullable, hasPK, hasUnique, hasDefault, autoInc }],
  constraints: { pk: bool, fks: [{col, ref}], uniques: [string] }
}]
```

---

### 2. Rule Engine (`runRules`)

Runs 40+ rules. Each finding:
```js
{
  severity: 'critical' | 'high' | 'medium' | 'low',
  category: 'Integrity' | 'Security' | 'Naming' | 'Normalization' |
            'Relationships' | 'Indexing' | 'Performance' |
            'Operational' | 'Architecture' | 'Query',
  title, table, description, fix, tags, schemaName
}
```

See [RULE_ENGINE.md](RULE_ENGINE.md) for full rule documentation.

---

### 3. Domain Detection (`detectDomain`)

Infers business domain from Business Overview text. Supports 7 domains: E-commerce, Healthcare, Finance, SaaS, HR/Payroll, Logistics, Education. Detected domain boosts penalty weights for relevant categories.

---

### 4. Scoring Engine (`scoreSecs`, `calcScore`)

Converts findings into section scores and a composite health score. See [SCORING.md](SCORING.md).

---

### 5. Schema Intelligence Engine (`generateSchemaInsight`)

Analyzes schema structure to give strategic recommendations in the Summary tab.
- Single schema: detects domain mixing, recommends splits, suggests future schemas
- Multi-schema: flags quality gaps, duplicate tables, cross-schema FKs, tiny schemas

---

### 6. Platform Lock (`checkPlatformLock`)

The Primary Database Platform dropdown in Step 1:
- **Unlocked** — user can freely change platform, propagates to all schemas
- **Locks automatically** when any schema has DDL content
- **Unlocks** when all DDL is cleared or schemas with DDL are removed
- Triggered on: DDL input, file upload, sample load, schema removal

---

### 7. PDF Builder (`buildPDF`)

Generates a multi-page A4 PDF using jsPDF. See [PDF_REPORT.md](PDF_REPORT.md).

---

## Data Flow

```
User types DDL
    │
    ▼
parseDDL() ── extractTableBody() ── balanced paren walk
    │
    ▼
runRules(tables, queries, overview, domain)
    │
    ├── Per-table rules (R01–R25)
    ├── Cross-table rules (R26–R27)
    └── Query rules (R41–R43)
    │
    ▼
scoreSecs(findings) ── section scores (0–100 each)
    │
    ▼
calcScore(secs) ── weighted composite health score
    │
    ▼
generateSchemaInsight(result) ── strategic recommendations
    │
    ▼
renderAll(result) ── DOM update
    │
    └── exportPDF() ── jsPDF 9-section report + Appendix
```

---

## File Structure

```
db-analyzer/
├── index.html          # Entire application (~131 KB)
├── vercel.json         # Vercel routing + security headers
├── README.md           # Project overview
├── .gitignore
└── docs/
    ├── ARCHITECTURE.md
    ├── RULE_ENGINE.md
    ├── SCORING.md
    ├── PDF_REPORT.md
    ├── DEPLOYMENT.md
    ├── CHANGELOG.md
    └── ROADMAP.md
```

---

## Privacy & Security

- No `fetch()` calls with user data
- No `localStorage`, `sessionStorage`, or cookies
- No analytics or telemetry
- Vercel headers: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`
- Fully offline capable after first load
