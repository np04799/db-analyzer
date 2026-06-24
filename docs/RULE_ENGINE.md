# Rule Engine — DB Schema Analyzer

## Overview

The rule engine runs 50+ deterministic rules across 8 categories against parsed DDL.
Every rule is a static check — no AI, no probability. Each finding either fires or it doesn't.

## Architecture

```
parseDDL(ddl) → tables[]
     ↓
runRules(tables, queries, overview, domain) → findings[]
     ↓
scoreSecs(findings) → sectionScores{}
     ↓
calcScore(sectionScores) → healthScore (0–100)
```

## Rule Categories & Full Reference

### 🔑 Keys & Integrity (R01–R08, N6)

| ID | Rule | Severity |
|----|------|----------|
| R01 | Missing PRIMARY KEY | Critical |
| R02 | Empty table definition | Critical |
| R03 | FLOAT/REAL for monetary column | High |
| R04 | Missing UNIQUE constraint on email | High |
| R05 | NULL allowed on key columns (email, name, status) | Medium |
| R06 | TEXT used for bounded-length field | Low |
| R07 | Wide table (30+ columns) | Medium |
| N6  | DECIMAL/NUMERIC without explicit precision and scale | Medium |

### 🛡 Security (R09–R14)

| ID | Rule | Severity |
|----|------|----------|
| R09 | Plaintext password storage (pwd/pass VARCHAR < 60) | Critical |
| R10 | Unprotected PII — SSN / NIN / passport | Critical |
| R11 | Payment card data column | Critical |
| R12 | Sensitive token/API key stored as plain VARCHAR | High |
| R13 | Role/permission as unconstrained VARCHAR | High |
| R14 | No soft-delete pattern on user/account tables | Medium |

### 📐 Normalization (R15–R19, N7)

| ID | Rule | Severity |
|----|------|----------|
| R15 | Repeating column groups — 1NF violation (addr1/addr2/addr3) | High |
| R16 | Derived/computed column (total, full_name, final_amount) | Medium |
| R17 | Duplicate column semantics (total + final_total + amount) | Medium |
| N7  | Comma-separated values stored in a column — 1NF violation | High |

### ✏️ Naming Conventions (R20–R26, N8)

| ID | Rule | Severity |
|----|------|----------|
| R20 | Reserved word as table name (USER, ORDER, GROUP) | High |
| R21 | Reserved word as column name | Medium |
| R22 | ALLCAPS table name | Medium |
| R23 | Mixed naming conventions (snake_case + camelCase) | Medium |
| R24 | Cryptic column name (≤2 characters) | Low |
| R25 | Inconsistent plural/singular table names across schema | Low |
| N8  | Boolean column missing is_/has_/can_ prefix | Low |

### 🔗 Relationships (R27–R32, N2, N3, N5)

| ID | Rule | Severity |
|----|------|----------|
| R27 | _id column without FK constraint | High |
| R28 | Missing index on FK column | High |
| R29 | Isolated table (no FK relationships in or out) | Medium |
| R30 | Cross-schema FK gap (table referenced but not in schema) | Medium |
| N2  | Nullable FK column without ON DELETE rule | Medium |
| N3  | Self-referencing FK not declared (parent_id, manager_id) | Medium |
| N5  | Junction/pivot table missing composite PRIMARY KEY | Medium |

### ⚙️ Operational (R33–R37, N4, N9, N10)

| ID | Rule | Severity |
|----|------|----------|
| R33 | Missing created_at timestamp | Medium |
| R34 | Missing updated_at timestamp | Low |
| R35 | Single-column table | Low |
| N4  | Missing tenant_id/org_id in SaaS domain | High |
| N9  | Missing created_by / updated_by on transactional tables | Low |
| N10 | No audit table in Healthcare/Finance/Banking domain | High |

### ⚡ Performance & Indexing (R38–R41)

| ID | Rule | Severity |
|----|------|----------|
| R38 | IP address stored as VARCHAR (use INET) | Low |
| R39 | CHAR(n) where n > 4 (use VARCHAR instead) | Low |
| R40 | Missing index on email/username/slug column | Medium |
| R41 | Missing index on status/type column | Low |

### 🔍 Query Analysis (R42–R44)

| ID | Rule | Severity |
|----|------|----------|
| R42 | SELECT * in query | Medium |
| R43 | UPDATE/DELETE without WHERE clause | Critical |
| R44 | Implicit JOIN syntax (comma-join) | Low |

### 🔒 Platform Compatibility (Advisory — no score impact)

Triggered when DDL types don't match the selected platform. Shows a warning banner in Summary tab.

| Platform | Flagged as incompatible |
|----------|------------------------|
| PostgreSQL | AUTO_INCREMENT, NVARCHAR, DATETIME, NUMBER, VARCHAR2 |
| MySQL | SERIAL, BIGSERIAL, TIMESTAMPTZ, JSONB, INET, BOOLEAN |
| SQL Server | SERIAL, AUTO_INCREMENT, TIMESTAMPTZ, JSONB, BOOLEAN |
| Oracle | SERIAL, AUTO_INCREMENT, BOOLEAN, TIMESTAMPTZ, JSONB |
| SQLite | SERIAL, TIMESTAMPTZ, JSONB, INET, UUID, BOOLEAN |

## New Rules Added in v1.1 (Task 1)

10 new rules were added:
- **N1** status/type VARCHAR without CHECK constraint (Medium)
- **N2** Nullable FK without ON DELETE rule (Medium)
- **N3** Self-referencing FK not declared (Medium)
- **N4** Missing tenant_id in SaaS domain (High)
- **N5** Junction table missing composite PK (Medium)
- **N6** DECIMAL/NUMERIC without precision/scale (Medium)
- **N7** Comma-separated values in column (High)
- **N8** Boolean column missing is_/has_ prefix (Low)
- **N9** Missing created_by/updated_by (Low)
- **N10** No audit table in regulated domain (High)

Additionally, a pre-existing bug in the plaintext password rule was fixed — the parser
strips `(n)` from `VARCHAR(n)` so the length check now uses `col.rawType` fallback.

## Rule Engine Flow

```javascript
function runRules(tables, queries, overview, domain) {
  const findings = [];
  function add(severity, category, title, table, description, fix, tags) {
    findings.push({ severity, category, title, table, description, fix, tags });
  }

  // Per-table rules
  tables.forEach(t => {
    // Per-column rules
    t.columns.forEach(col => { ... });
    // Cross-column rules
    // Relationship rules
  });

  // Cross-table rules
  // Query rules (if queries submitted)
  // New rules (N1–N10)
  // Audit table rule (N10) — cross-schema check

  return findings;
}
```

## Adding a New Rule

1. Find the appropriate section in `runRules()`
2. Use the `add(severity, category, title, table, description, fix, tags)` helper
3. Severity: `'critical'` | `'high'` | `'medium'` | `'low'`
4. Category must match a scored section key in `SECTION_MAP`
5. Test with Node.js simulation before committing
