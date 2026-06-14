# Rule Engine — DB Schema Analyzer v1.0

The rule engine runs **40+ deterministic checks** against parsed DDL. Rules are grouped into categories, each mapped to a report section and scoring dimension.

---

## Rule Categories & Weights

| Category | Section | Score Weight |
|---|---|---|
| Integrity | Keys & Integrity | 25% |
| Normalization | Normalization | 20% |
| Architecture, Naming, Operational | Schema & Architecture | 20% |
| Security | Security | 15% |
| Indexing, Performance | Indexing & Performance | 10% |
| Relationships | Relationships | 10% |
| Query | Query Analysis | — (informational) |

---

## Schema Rules (R01–R27)

### Integrity

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R01 | 🔴 Critical | Missing PRIMARY KEY on table | `ALTER TABLE t ADD COLUMN id SERIAL PRIMARY KEY` |
| R02 | 🔴 Critical | Empty table definition (no columns parsed) | Review and resubmit DDL |
| R14 | 🟠 High | FLOAT/REAL/DOUBLE used for monetary column | `ALTER TABLE t ALTER COLUMN price TYPE DECIMAL(12,2)` |
| R19 | 🟠 High | Missing UNIQUE constraint on email column | `ALTER TABLE t ADD CONSTRAINT uq_t_email UNIQUE (email)` |
| R22 | 🟡 Medium | NULL allowed on key column (email, name, status, type) | `ALTER TABLE t ALTER COLUMN col SET NOT NULL` |

### Security

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R15 | 🔴 Critical | Password column shorter than 60 chars (plaintext risk) | Rename to `password_hash VARCHAR(255)`, enforce bcrypt/argon2 |
| R16 | 🔴 Critical | PII column: ssn, sin, nin, passport, national_id, tax_id | Encrypt with pgcrypto or column-level encryption |
| R17 | 🔴 Critical | Payment card column: card_number, cvv, card_cvv | Replace with payment token (Stripe/Braintree) |
| R23 | 🟠 High | Token/secret column stored in plain TEXT/VARCHAR | Hash validation tokens; encrypt refresh tokens |

### Naming

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R03 | 🟠 High | Reserved SQL word used as table name | `ALTER TABLE t RENAME TO t_data` |
| R07 | 🟡 Medium | ALLCAPS table name (legacy Oracle/DB2 style) | `ALTER TABLE T RENAME TO t` |
| R08 | 🟡 Medium | Mixed snake_case and camelCase in same table | Standardise to snake_case |
| R12 | 🟡 Medium | Reserved SQL word used as column name | `ALTER TABLE t RENAME COLUMN col TO col_value` |
| R13 | 🟢 Low | Column name ≤2 chars (cryptic) | Rename to descriptive name |

### Normalization

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R20 | 🟠 High | Repeating column group: addr1, addr2, addr3 (1NF violation) | Extract to child table |
| R21 | 🟡 Medium | Derived/computed column: total, balance, full_name, final_total | Remove or use generated column |
| R04 | 🟡 Medium | Wide table (>30 columns) | Vertical partition into related tables |

### Operational

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R05 | 🟡 Medium | Missing `created_at` timestamp | `ALTER TABLE t ADD COLUMN created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` |
| R06 | 🟢 Low | Missing `updated_at` timestamp | Add column + trigger |
| R09 | 🟢 Low | Single-column table | Review if table is incomplete or should merge |
| R10 | 🟢 Low | No soft-delete on user/customer/employee table | `ALTER TABLE t ADD COLUMN deleted_at TIMESTAMPTZ DEFAULT NULL` |

### Relationships

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R11 | 🟠 High | FK column has no index | `CREATE INDEX idx_t_col ON t(col)` |
| R18 | 🟠 High | `_id` / `_no` column without FK constraint | Add `FOREIGN KEY (col) REFERENCES ref_table(id)` |
| R26 | 🟡 Medium | Table has no FK relationships in or out (isolated) | Review relationships in context of full schema |

### Architecture

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R27 | 🟡 Medium | Business overview domain doesn't match schema tables | Submit complete schema or clarify overview |

### Performance / Indexing

| Rule | Severity | Trigger | Fix |
|---|---|---|---|
| R24 | 🟢 Low | IP address stored as VARCHAR | `ALTER TABLE t ALTER COLUMN ip TYPE INET USING ip::INET` |
| R25 | 🟢 Low | CHAR(n>4) where VARCHAR preferred | `ALTER TABLE t ALTER COLUMN col TYPE VARCHAR(n)` |

---

## Query Rules (R41–R43)

Query rules fire only when SQL queries are submitted in the Query Input field.

| Rule | Severity | Trigger | Example |
|---|---|---|---|
| R41 | 🟡 Medium | `SELECT *` in query | `SELECT * FROM users` → specify columns |
| R42 | 🔴 Critical | `UPDATE` or `DELETE` without `WHERE` | `DELETE FROM users` → always add WHERE |
| R43 | 🟢 Low | Implicit JOIN (comma syntax) | `FROM t1, t2 WHERE ...` → use explicit `JOIN` |

---

## Severity Penalty Reference

| Severity | Points Deducted | Meaning |
|---|---|---|
| 🔴 Critical | −15 | Fix before next release. Data loss, security breach, or corruption risk. |
| 🟠 High | −8 | Fix this sprint. Integrity, security, or significant performance issue. |
| 🟡 Medium | −4 | Plan to fix. Maintainability, normalization, or operational gap. |
| 🟢 Low | −1 | Best practice. Cosmetic or minor improvement. |

---

## Domain Priority Weighting

When a business domain is auto-detected, certain category penalties are multiplied:

| Domain | Boosted Categories | Multiplier |
|---|---|---|
| Healthcare | Security, Integrity | 2×, 1.5× |
| Finance | Integrity, Security, Performance | 2×, 1.8×, 1.3× |
| E-commerce | Performance, Integrity, Indexing | 1.5×, 1.5×, 1.4× |
| SaaS | Relationships, Performance | 1.4×, 1.3× |

---

## Adding New Rules

Rules are defined inside `runRules(tables, queryTxt, overview, domain)` in `index.html`. Each rule calls `add()`:

```js
function add(severity, category, title, table, description, fix, tags) { ... }
```

To add a rule:
1. Open `index.html`, find `// R[next number]`
2. Add an `add()` call with the appropriate severity, category, and fix SQL
3. Test with a DDL that triggers it
4. Verify score impact in the Results > Summary tab
