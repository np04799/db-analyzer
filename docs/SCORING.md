# Scoring — DB Schema Analyzer

## Overview

Every analysis produces a **Health Score** from 0–100 with an A–F grade.
The score is a weighted composite of 6 section scores, adjusted by domain multipliers.

---

## Step 1 — Findings → Section Scores

Each finding reduces its section score by a penalty:

| Severity | Penalty |
|----------|---------|
| Critical | −15 pts |
| High     | −8 pts  |
| Medium   | −4 pts  |
| Low      | −1 pt   |

**Section Score = max(0, 100 − Σ penalties × domain_multiplier)**

Findings map to sections via `SECTION_MAP`:

| Finding Category | Section |
|-----------------|---------|
| Keys & Integrity | Keys & Integrity |
| Security | Security |
| Normalization | Normalization |
| Schema & Architecture, Naming, Operational | Schema & Architecture |
| Indexing, Performance | Indexing & Performance |
| Relationships | Relationships |
| Query | Query Analysis (informational only — not scored) |

---

## Step 2 — Weighted Composite Score

```
Health Score =
  (Keys & Integrity    × 0.25) +
  (Normalization       × 0.20) +
  (Schema & Arch       × 0.20) +
  (Security            × 0.15) +
  (Indexing & Perf     × 0.10) +
  (Relationships       × 0.10)
```

Total weight = 1.00 (100%)

**Query Analysis** findings are shown in the Issues tab but **do not affect the score**.

---

## Grade Scale

| Grade | Score Range | Meaning |
|-------|------------|---------|
| A | 90–100 | Excellent — production ready |
| B | 80–89  | Good — minor improvements recommended |
| C | 70–79  | Fair — notable issues to address |
| D | 60–69  | Poor — significant problems present |
| F | 0–59   | Critical — unsafe to scale |

---

## Domain Multipliers

When a business domain is detected from the overview text, penalties in critical
categories are multiplied to reflect the higher stakes of that domain:

| Domain | Category | Multiplier | Rationale |
|--------|----------|-----------|-----------|
| Healthcare | Security | 2.0× | HIPAA — PHI protection is mandatory |
| Healthcare | Integrity | 1.5× | Patient data accuracy is life-critical |
| Finance / Banking | Integrity | 2.0× | Financial data must be exact |
| Finance / Banking | Security | 1.8× | PCI-DSS and SOX regulations |
| Finance / Banking | Performance | 1.3× | Transaction throughput requirements |
| E-commerce | Performance | 1.5× | Order processing speed matters |
| E-commerce | Integrity | 1.5× | Order and inventory accuracy |
| E-commerce | Indexing | 1.4× | Product search performance |
| SaaS | Relationships | 1.4× | Multi-tenant data isolation |
| SaaS | Performance | 1.3× | Concurrent user scalability |

Domain is detected automatically from the business overview text in Step 1.
Users can also select a domain explicitly.

---

## Example Calculation

Schema with 2 Critical findings in Security + 1 High in Integrity (Healthcare domain):

```
Security section:
  Raw penalties = 2 × 15 = 30
  With Healthcare multiplier = 30 × 2.0 = 60 pts deducted
  Section score = max(0, 100 − 60) = 40

Integrity section:
  Raw penalties = 1 × 8 = 8
  With Healthcare multiplier = 8 × 1.5 = 12 pts deducted
  Section score = max(0, 100 − 12) = 88

Health Score (assuming other sections = 100):
  = (88 × 0.25) + (100 × 0.20) + (100 × 0.20) + (40 × 0.15) + (100 × 0.10) + (100 × 0.10)
  = 22 + 20 + 20 + 6 + 10 + 10
  = 88 → Grade B
```

---

## Multi-Schema Scoring

When multiple schemas are submitted, findings from all schemas are merged:
- `allF` = flattened findings from all schemas with `schemaName` tag
- The global health score uses the combined `allF`
- Each schema also gets its own individual score shown in the Comparison tab
- Schema Intelligence card compares individual scores and flags quality gaps

---

## Platform Mismatch — No Score Impact

Platform type incompatibilities (e.g. `SERIAL` on MySQL) are advisory only.
They show a warning banner in the Summary tab but do not affect the health score.
This is intentional — a schema is not "worse" because of a platform selection mismatch,
it may simply mean the wrong platform was selected in Step 1.
