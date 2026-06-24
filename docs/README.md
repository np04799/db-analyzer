# DB Schema Analyzer

**Browser-based database schema linter and health checker.**
Paste `CREATE TABLE` DDL → get instant findings, health score, ER diagram, and SQL fixes.

🔗 **Live tool:** https://db-analyzer-two.vercel.app/
📖 **Documentation:** https://db-analyzer-two.vercel.app/docs.html

---

## Features

- **50+ deterministic rules** across 8 categories — no AI, no guessing
- **Health score (A–F)** with domain-aware weighting (Healthcare, Finance, E-commerce, SaaS)
- **ER Diagram** — pure SVG, Sugiyama layout, explicit + inferred FK relationships, zoom/pan, SVG/PNG download
- **Platform mismatch detection** — PostgreSQL, MySQL, SQL Server, Oracle, SQLite, Generic SQL
- **Multi-schema support** — up to 5 schemas, side-by-side Comparison tab
- **Schema Intelligence** — split/merge recommendations, domain mixing detection
- **PDF export** — 9-section stakeholder report including About This Analysis page
- **Query anti-pattern detection** — SELECT *, UPDATE without WHERE, implicit JOINs
- **SQL fix suggestions** — every finding includes a ready-to-run ALTER TABLE fix
- **Zero backend** — 100% client-side, no data ever leaves the browser

---

## Rule Categories

| Category | Rules | Highlights |
|----------|-------|-----------|
| Keys & Integrity | 9 | Missing PK, FLOAT for money, DECIMAL without precision |
| Security | 6 | Plaintext passwords, PII (SSN/NIN), card numbers, tokens |
| Normalization | 4 | 1NF (repeating groups, comma-separated values), 3NF |
| Naming Conventions | 7 | Reserved words, ALLCAPS, boolean prefix, cryptic names |
| Relationships | 6 | Missing FKs, nullable FK no ON DELETE, self-ref FK, junction PK |
| Operational | 6 | Missing timestamps, tenant_id (SaaS), audit table (Healthcare/Finance) |
| Performance | 4 | IP as VARCHAR, missing indexes on FK/email/status columns |
| Query Analysis | 3 | SELECT *, UPDATE without WHERE, implicit JOIN |

---

## Sample Schemas

| Sample | Description | Expected Score |
|--------|-------------|----------------|
| 🛒 E-commerce | 4 tables with FK relationships | ~70/C |
| 🏥 Healthcare | Patient records, HIPAA-relevant | ~75/C |
| ☁️ SaaS Platform | Multi-tenant with explicit FKs | ~85/B |
| ⚠️ Legacy ERP | ALLCAPS, missing constraints | ~60/D |
| 💀 Badly Designed | 49 findings, all categories triggered | ~52/F |
| 🔀 Multi-Schema | Auth (92/A) vs Billing (62/D) side-by-side | Varies |

---

## Tech Stack

- **Frontend:** Vanilla HTML5 / CSS3 / JavaScript (ES6+) — single file
- **PDF:** jsPDF 2.5.1 (CDN)
- **Hosting:** Vercel Static (auto-deploy from `main`)
- **Build:** None — no build step, no framework, no backend

---

## Repository Structure

```
├── index.html          # Entire application (HTML + CSS + JS)
├── docs.html           # Documentation site
├── vercel.json         # Routing config (serves docs.html directly)
├── assets/
│   └── DB_Schema_Analyzer_Presentations.zip
└── docs/
    ├── README.md
    ├── ARCHITECTURE.md
    ├── CHANGELOG.md
    ├── DEPLOYMENT.md
    ├── PDF_REPORT.md
    ├── ROADMAP.md
    ├── RULE_ENGINE.md
    └── SCORING.md
```

---

## Documentation

| File | Contents |
|------|----------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | System design, parser, rule engine, tab system |
| [RULE_ENGINE.md](RULE_ENGINE.md) | All 50+ rules with severity and fix reference |
| [SCORING.md](SCORING.md) | Health score formula, weights, domain multipliers |
| [PDF_REPORT.md](PDF_REPORT.md) | PDF export structure and section reference |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [ROADMAP.md](ROADMAP.md) | Planned features and monetisation strategy |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Vercel deployment and GitHub workflow |

---

## Privacy

No schema data, DDL, or query text is ever sent to any server.
All analysis runs synchronously in the browser via JavaScript.
The tool works fully offline after the first page load.
