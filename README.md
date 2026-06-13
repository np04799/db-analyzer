# DB Schema Analyzer — v1.0

AI-assisted database design review tool. Analyzes schema structure, constraints, normalization, security, and relationships — then generates a structured PDF report.

## What It Does

- Accepts DDL (CREATE TABLE statements), JSON schema, or plain text
- Runs 40+ deterministic rule checks locally in the browser
- Evaluates: Schema & Architecture · Keys & Integrity · Normalization (1NF–3NF) · Security · Indexing · Relationships · Query Patterns
- Generates a 9-section PDF report with findings, scores, and SQL fixes
- Supports up to 5 schemas simultaneously with cross-schema comparison
- Fully offline capable — no data ever leaves the browser

## Privacy

All analysis runs in the browser. No schema data is transmitted to any server. No backend, no database, no telemetry.

## Deployment

### Deploy to Vercel (recommended)

```bash
# 1. Clone or upload this folder to a GitHub repo

# 2. Connect repo to Vercel at vercel.com/new
#    - Framework Preset: Other
#    - Root Directory: ./
#    - No build command needed

# 3. Deploy — done
```

### Deploy via Vercel CLI

```bash
npm install -g vercel
cd db-schema-analyzer
vercel --prod
```

### Run locally (no server needed)

Just open `index.html` in any modern browser. Works completely offline.

## Usage

1. **Step 1 — Business Overview** (optional): Describe your system. The tool auto-detects the business domain and weights findings accordingly.
2. **Step 2 — Schemas**: Paste DDL or upload `.sql` / `.txt` / `.ddl` files. Add up to 5 schemas.
3. **Step 3 — Analyze**: Click "Analyze Schema". Results appear in seconds.
4. **Export**: Click "Export PDF Report" to download a full analysis report.

## Supported Databases

PostgreSQL · MySQL · SQL Server · Oracle · SQLite · Generic SQL

## Report Sections

| Section | Contents |
|---|---|
| S1 | Business Context — overview text, domain detection, scope |
| S2 | Executive Summary — health score, dimensional scores |
| S3 | Schema & Architecture — entity inventory, logical/physical/operational |
| S4 | Keys & Integrity — PK/FK matrix, data type issues |
| S5 | Normalization — 1NF/2NF/3NF assessment, constraint review |
| S6 | Query Analysis — anti-patterns (if queries submitted) |
| S7 | Schema Comparison — side-by-side (if 2+ schemas) |
| S8 | Issue Register — all findings with SQL fixes |
| S9 | Remediation Roadmap — 4-phase plan with timelines |
| Appendix | Rules applied, v2 preview |

## Migration to v2 (Node.js + React)

This v1 is structured for clean migration:
- Each UI section maps 1:1 to a React component
- Rule engine (pure JS) moves to a Node.js API route with zero rewrite
- `jsPDF` client-side PDF → Puppeteer or `pdfkit` server-side
- Add `next-auth` for login
- Add PostgreSQL for schema history and team features

## Tech Stack

- **Frontend**: Vanilla HTML5 + CSS3 + JavaScript (ES6+)
- **PDF**: jsPDF 2.5.1 (CDN)
- **Hosting**: Vercel Static
- **No frameworks, no build step, no dependencies to install**

## Version History

| Version | Notes |
|---|---|
| v1.0 | Rule engine, 40+ checks, 9-section PDF, multi-schema, offline |
| v2.0 (planned) | React, Node.js, auth, DB connection, data profiling, schema history |

## License

Internal use. Not for public distribution in v1.
