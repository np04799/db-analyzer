# Roadmap — DB Schema Analyzer

---

## v1.0 — Current ✅

All v1 features are complete and live. See [CHANGELOG.md](CHANGELOG.md) for full feature list.

**Key constraints of v1:**
- Schema Only analysis (no live DB connection)
- No user accounts or persistence
- No team features
- Client-side PDF only
- Single HTML file architecture

---

## v2.0 — Planned

### Tech Stack Migration

| Layer | v1 | v2 |
|---|---|---|
| Frontend | Vanilla HTML/CSS/JS | React + TypeScript |
| Backend | None | Node.js (Next.js API routes) |
| Database | None | PostgreSQL |
| Auth | None | next-auth (Google, GitHub, email) |
| PDF | jsPDF (client) | Puppeteer or pdfkit (server) |
| Hosting | Vercel Static | Vercel (full-stack) |

---

### Planned Features

#### Data Profiling (requires DB connection)
- Connect directly to PostgreSQL, MySQL, or SQL Server
- Auto-extract schema (no DDL needed)
- Null rate analysis per column
- Cardinality analysis (high/low cardinality flags)
- Dead column detection (columns that are always NULL)
- Value distribution sampling
- Row count and table size

#### Query Workload Analysis
- Slow query log import (.csv or direct from `pg_stat_statements`)
- Index usage statistics (which indexes are actually used)
- N+1 query pattern detection
- Most expensive queries ranked by total time

#### Schema History & Tracking
- Save analysis results per project
- Track health score over time (trend chart)
- Before/after comparison when schema changes
- Export historical reports

#### Team Features
- User accounts and workspaces
- Assign findings to team members
- Mark findings as resolved / won't fix / accepted risk
- Jira / Linear integration (create tickets from findings)
- Slack notifications on new analysis

#### Enterprise
- SSO (SAML, Okta, Azure AD)
- Role-based access (viewer, analyst, admin)
- Audit log of who ran what analysis
- Private deployment (self-hosted Docker)
- SLA-backed support

#### Expanded Rule Engine
- Composite index recommendations
- Partial index opportunities
- Partition strategy recommendations (range, hash, list)
- CHECK constraint suggestions
- Row-level security (RLS) recommendations for multi-tenant schemas
- JSON/JSONB column anti-patterns (PostgreSQL)
- Missing cascade rules on FK
- Circular FK dependency detection

#### UI Improvements
- Dark/light theme toggle
- Schema diagram visualisation (entity-relationship diagram auto-generated)
- Inline SQL editor with syntax highlighting
- Side-by-side diff view for schema changes
- Mobile-optimised layout

---

## v3.0 — Future Ideas

- AI-assisted rule suggestions (LLM analysis of schema context)
- Natural language query: "Why is my schema slow?" 
- Migration script generator (generates full ALTER TABLE migration from findings)
- CI/CD integration (GitHub Action that fails PR if score drops below threshold)
- Schema registry (store canonical schemas, diff against them)

---

## Prioritisation

| Priority | Feature | Effort |
|---|---|---|
| P0 | React migration + TypeScript | High |
| P0 | User auth (next-auth) | Medium |
| P0 | Schema history (PostgreSQL) | Medium |
| P1 | Direct DB connection | High |
| P1 | Team features + assignments | High |
| P1 | Jira/Linear integration | Medium |
| P2 | Data profiling | High |
| P2 | Query workload analysis | High |
| P3 | SSO + enterprise | High |
| P3 | Schema diagram visualisation | Medium |

---

## Contribution

To propose a new rule or feature, open an issue on GitHub:
https://github.com/np04799/db-analyzer/issues
