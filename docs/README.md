# DB Schema Analyzer — v1.0

> A free, privacy-first database design review tool. Paste your DDL and get an instant scored report with actionable SQL fixes — no account, no uploads, no data leaves your browser.

🔗 **Live:** https://db-analyzer-two.vercel.app/
📦 **Repo:** https://github.com/np04799/db-analyzer

---

## What It Does

DB Schema Analyzer runs 40+ deterministic rules against your `CREATE TABLE` DDL and produces a structured report covering:

- **Keys & Integrity** — missing PKs, FK constraints, UNIQUE on email, data type correctness
- **Normalization** — 1NF repeating groups, 3NF derived columns, wide table detection
- **Security** — plaintext passwords, unencrypted PII (SSN, NIN), raw card numbers, token storage
- **Naming** — reserved words as table/column names, ALLCAPS, mixed conventions, cryptic names
- **Relationships** — missing FK constraints on `_id` columns, isolated tables, cross-schema FKs
- **Indexing** — missing indexes on all FK columns
- **Query Analysis** — `SELECT *`, `UPDATE/DELETE` without `WHERE`, implicit JOIN syntax
- **Schema Intelligence** — recommends schema splits, flags domain mixing, suggests future schemas

---

## Quick Start

1. Open **https://db-analyzer-two.vercel.app/**
2. Describe your system in **Step 1** and select your database platform
3. Paste `CREATE TABLE` statements in **Step 2** (or upload a `.sql` / `.txt` / `.ddl` file)
4. Click **Analyze Schema**
5. Review results across Issues, Summary, Solutions, Comparison, and Roadmap tabs
6. Click **Export PDF Report** to download a full analysis document

No login required. Works completely offline after first load.

---

## Documentation

| File | Description |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design, tech stack, module breakdown |
| [docs/RULE_ENGINE.md](docs/RULE_ENGINE.md) | All 40+ rules with descriptions and examples |
| [docs/SCORING.md](docs/SCORING.md) | How the health score is calculated |
| [docs/PDF_REPORT.md](docs/PDF_REPORT.md) | PDF report structure and all 9 sections |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Vercel deployment guide |
| [docs/CHANGELOG.md](docs/CHANGELOG.md) | Version history and release notes |
| [docs/ROADMAP.md](docs/ROADMAP.md) | v2 planned features |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES6+) |
| PDF Export | [jsPDF 2.5.1](https://github.com/parallax/jsPDF) (CDN) |
| Hosting | Vercel Static |
| Build | None — single HTML file, no bundler |

---

## Supported Platforms

PostgreSQL · MySQL · SQL Server · Oracle · SQLite · Generic SQL

---

## License

Internal use — v1. Not for public distribution.
