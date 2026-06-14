# Architecture — DB Schema Analyzer v1.0

## Overview

DB Schema Analyzer is a **single-file, client-side web application**. All analysis logic runs in the user's browser — no backend server, no database, no API calls. The entire application is one `index.html` file (~125 KB) served as a static asset from Vercel.

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

## Module Breakdown

### 1. DDL Parser (`parseDDL`, `extractTableBody`)

**File:** `index.html` — JS block
**Purpose:** Converts raw SQL DDL into a structured JavaScript object tree.

**Key functions:**
- `extractTableBody(ddl)` — walks balanced parentheses to correctly extract table body, handling `DEFAULT gen_random_uuid()`, `DEFAULT NOW()`, and inline `REFERENCES table(id)` without breaking on nested parens
- `parseDDL(ddl)` — splits table body by comma/newline (depth-aware), classifies each line as column, PK constraint, FK constraint, or UNIQUE constraint

**Output structure:**
```js
[{
  name: 'users',
  columns: [{ name, type, nullable, hasPK, hasUnique, hasDefault, autoInc }],
  constraints: { pk: bool, fks: [{col, ref}], uniques: [string] }
}]
```

**Handles:**
- `CREATE TABLE IF NOT EXISTS`
- Quoted/bracketed identifiers: `` `name` ``, `"name"`, `[name]`
- Inline `REFERENCES` on column definitions
- Standalone `FOREIGN KEY ... REFERENCES` syntax
- `CONSTRAINT name PRIMARY KEY/FOREIGN KEY/UNIQUE`
- Function calls in DEFAULT values (`NOW()`, `gen_random_uuid()`, `nextval()`)

---

### 2. Rule Engine (`runRules`)

**File:** `index.html` — JS block
**Purpose:** Runs 40+ deterministic rules against parsed tables and returns an array of findings.

Each finding:
```js
{
  severity: 'critical' | 'high' | 'medium' | 'low',
  category: 'Integrity' | 'Security' | 'Naming' | 'Normalization' | 'Relationships' | 'Indexing' | 'Performance' | 'Operational' | 'Architecture' | 'Query',
  title: string,
  table: string,
  description: string,
  fix: string,       // SQL remediation
  tags: string[],
  schemaName: string
}
```

See [RULE_ENGINE.md](RULE_ENGINE.md) for full rule documentation.

---

### 3. Domain Detection (`detectDomain`)

**Purpose:** Infers the business domain from the Business Overview text to weight findings appropriately.

**Domains:** E-commerce, Healthcare, Finance, SaaS, HR/Payroll, Logistics, Education, General

**Priority weights (`DOM_PRI`):** Healthcare boosts Security and Integrity weights; Finance boosts Integrity and Performance; E-commerce boosts Performance and Indexing.

---

### 4. Scoring Engine (`scoreSecs`, `calcScore`)

**Purpose:** Converts findings into section scores and a composite health score.

See [SCORING.md](SCORING.md) for full formula documentation.

---

### 5. Schema Intelligence Engine (`generateSchemaInsight`, `renderSchemaInsight`)

**Purpose:** Analyzes schema structure and count to give strategic recommendations in the Summary tab.

**Single-schema logic:**
- Detects domain mixing across 6 concern groups (auth, billing, content, ops, audit, tenant)
- Recommends schema splits when 3+ concern groups detected
- Suggests future schemas based on domain (SaaS → tenant schema, Healthcare → HIPAA audit trail)

**Multi-schema logic:**
- Flags quality gaps ≥25 points between schemas
- Detects duplicate table names across schemas
- Identifies single-table schemas that may not need separation
- Notes cross-schema FK dependencies

---

### 6. PDF Builder (`buildPDF`, `exportPDF`)

**Purpose:** Generates a multi-page A4 PDF report using jsPDF.

See [PDF_REPORT.md](PDF_REPORT.md) for section documentation.

---

### 7. UI Layer

**Layout:** CSS Grid — `220px nav | 1fr main`
**Key components:**
- `Step 1` — Business overview + domain detection + platform selector (with lock)
- `Step 2` — Schema cards (up to 5) with DDL textarea + file upload + query input
- `Step 3` — Analysis options + progress bar
- Results — Score strip (clickable tiles) + tabbed sections (Issues, Summary, Solutions, Comparison, Roadmap)
- How It Works — horizontal steps banner below header
- Footer — tool description, categories, platforms

---

## Data Flow

```
User types DDL
    │
    ▼
parseDDL() ──── extractTableBody() ──── balanced paren walk
    │
    ▼
runRules(tables, queries, overview, domain)
    │
    ├── Per-table rules (R01–R25)
    ├── Cross-table rules (R26–R27)
    └── Query rules (R41–R43)
    │
    ▼
scoreSecs(findings) ──── section scores (0–100)
    │
    ▼
calcScore(secs) ──── weighted composite health score
    │
    ▼
generateSchemaInsight(result) ──── strategic recommendations
    │
    ▼
renderAll(result) ──── DOM update (Issues, Summary, Solutions, Comparison, Roadmap)
    │
    └── exportPDF() ──── jsPDF 9-section report + Appendix
```

---

## File Structure

```
db-analyzer/
├── index.html          # Entire application (HTML + CSS + JS)
├── vercel.json         # Vercel routing + security headers
├── README.md           # Project overview
├── .gitignore
└── docs/
    ├── ARCHITECTURE.md  # This file
    ├── RULE_ENGINE.md   # Rule documentation
    ├── SCORING.md       # Scoring formula
    ├── PDF_REPORT.md    # PDF section guide
    ├── DEPLOYMENT.md    # Deployment instructions
    ├── CHANGELOG.md     # Version history
    └── ROADMAP.md       # v2 planned features
```

---

## Privacy & Security

- **No data transmission** — `fetch()` is never called with user data. All processing is synchronous JavaScript in the browser.
- **No storage** — `localStorage`, `sessionStorage`, and cookies are not used.
- **No telemetry** — No analytics, tracking, or error reporting calls.
- **Security headers** (via `vercel.json`): `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`
- **Offline capable** — Works after first load with no network connection.

---

## Migration Path to v2

The v1 architecture is deliberately structured for clean migration:

| v1 (current) | v2 (planned) |
|---|---|
| Vanilla JS | React + TypeScript |
| Client-side rule engine | Node.js API route (same rules) |
| jsPDF client-side | Puppeteer / pdfkit server-side |
| No auth | next-auth |
| No persistence | PostgreSQL schema history |
| No DB connection | Direct DB introspection |

See [ROADMAP.md](ROADMAP.md) for full v2 feature list.
