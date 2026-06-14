# Scoring System — DB Schema Analyzer v1.0

## Health Score Formula

The health score is a **weighted composite** of 6 section scores, each calculated independently.

### Step 1 — Section Score

Each section starts at **100** and deducts penalty points per finding:

```
Section Score = max(0, 100 − Σ(penalty × domain_weight))
```

| Severity | Penalty |
|---|---|
| Critical | −15 pts |
| High | −8 pts |
| Medium | −4 pts |
| Low | −1 pt |

### Step 2 — Composite Health Score

Section scores are combined using fixed weights:

| Section | Weight | Categories Included |
|---|---|---|
| Keys & Integrity | **25%** | Integrity |
| Normalization | **20%** | Normalization |
| Schema & Architecture | **20%** | Architecture, Naming, Operational |
| Security | **15%** | Security |
| Indexing & Performance | **10%** | Indexing, Performance |
| Relationships | **10%** | Relationships |

```
Health Score = Σ(Section Score × Section Weight)
             = (Integrity × 0.25) + (Normalization × 0.20) + (Schema × 0.20)
             + (Security × 0.15) + (Indexing × 0.10) + (Relationships × 0.10)
```

Query Analysis findings are reported but **do not affect the health score** (informational only).

---

## Grade Scale

| Score | Grade | Meaning |
|---|---|---|
| 90–100 | **A** | Excellent — production-ready with minor polish needed |
| 80–89 | **B** | Good — a few improvements recommended |
| 70–79 | **C** | Fair — notable issues to address before scaling |
| 60–69 | **D** | Poor — significant problems requiring immediate attention |
| 0–59 | **F** | Critical — unsafe to deploy; major rework needed |

---

## Worked Example

**Schema:** `users` table with:
- No PRIMARY KEY (Critical, Integrity)
- Plaintext password `pwd VARCHAR(20)` (Critical, Security)
- Missing `created_at` (Medium, Operational)
- Reserved word column `NAME` (Medium, Naming)
- Missing FK index (High, Indexing)

**Section scores:**

| Section | Penalty | Score |
|---|---|---|
| Keys & Integrity | −15 (no PK) | 85 |
| Security | −15 (plaintext pwd) | 85 |
| Schema & Architecture | −4 −4 (2×medium) | 92 |
| Indexing & Performance | −8 (FK index) | 92 |
| Normalization | 0 | 100 |
| Relationships | 0 | 100 |

**Composite:**
```
= (85×0.25) + (100×0.20) + (92×0.20) + (85×0.15) + (92×0.10) + (100×0.10)
= 21.25 + 20 + 18.4 + 12.75 + 9.2 + 10
= 91.6 → rounded to 92
```
**Grade: A** — the penalties are spread thinly enough across sections that the composite stays high. This illustrates why fixing **Critical issues in high-weight sections** (Integrity, Security) has the most impact on the score.

---

## Multi-Schema Scoring

When multiple schemas are submitted:
- Each schema is scored **independently**
- The **global health score** is calculated from the combined finding set across all schemas
- The Comparison tab shows per-schema scores side by side
- The Schema Intelligence card flags quality gaps ≥25 points between schemas

---

## Tips for Improving Your Score

| Action | Impact |
|---|---|
| Add PRIMARY KEY to all tables | +15 pts to Integrity per table |
| Fix plaintext password storage | +15 pts to Security |
| Add UNIQUE constraint on email | +8 pts to Integrity |
| Add indexes on all FK columns | +8 pts per FK to Indexing |
| Add `created_at` to all tables | +4 pts to Schema per table |
| Fix FLOAT for monetary columns | +8 pts to Integrity |
| Remove repeating column groups | +8 pts to Normalization |
