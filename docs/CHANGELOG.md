# Changelog — DB Schema Analyzer

All notable changes to this project are documented here.

---

## [1.0.0] — 2026-06-14

### Initial Release

#### Features
- **Rule Engine** — 40+ deterministic DDL analysis rules across 10 categories
- **DDL Parser** — Balanced-paren body extraction supporting `DEFAULT NOW()`, `gen_random_uuid()`, inline `REFERENCES table(id)` syntax
- **Domain Detection** — Auto-detects 7 business domains (E-commerce, Healthcare, Finance, SaaS, HR/Payroll, Logistics, Education) from business overview text with priority weighting
- **Multi-Schema Support** — Up to 5 schemas analyzed simultaneously
- **Query Analysis** — Optional SQL query submission for anti-pattern detection (SELECT *, UPDATE without WHERE, implicit JOIN)
- **Health Score** — Weighted composite score (0–100) with A–F grade across 6 sections
- **Clickable Score Tiles** — Critical/High/Medium/Low tiles filter Issues tab on click with toggle off
- **Score Hint (ⓘ)** — Hover popover explaining scoring formula, section weights, grade scale
- **Results Tabs** — Issues · Summary · Solutions · Comparison · Roadmap
- **Solutions Tab** — Key action points per finding with effort labels (Urgent/High/Medium/Major/Quick/Minor) and severity filter
- **Schema Intelligence Card** — Strategic recommendations in Summary tab for schema splits, quality gaps, duplicate tables, future schema needs
- **Schema Comparison Tab** — Side-by-side scores, table counts, finding breakdown for 2+ schemas
- **4-Phase Roadmap** — Week 1 / Weeks 2–3 / Month 1–2 / Month 2–3 remediation plan
- **PDF Export** — 9-section A4 report (Cover + S1–S9 + Appendix) generated client-side via jsPDF
- **Sample Schemas** — 4 built-in samples: E-commerce, Healthcare, SaaS, Legacy ERP
- **File Upload** — Drag-and-drop or click upload for `.sql`, `.txt`, `.ddl`, `.json` files
- **Platform Selector** — DB platform in Step 1 with auto-lock once DDL is entered; propagates to all schemas
- **How It Works Banner** — Horizontal 5-step pipeline below header
- **Footer** — Tool description, what it analyzes, supported platforms, privacy badge

#### UI
- Dark theme with CSS custom properties
- Font size 16px base for readability
- Responsive layout (mobile: nav hidden, single column)
- Left nav: Steps navigation + privacy tip
- Collapsible query input per schema
- Expand/collapse finding cards with Copy SQL button

#### Privacy & Security
- Zero data transmission — all processing in browser
- No localStorage, sessionStorage, or cookies
- Vercel security headers: X-Frame-Options DENY, X-Content-Type-Options nosniff
- Fully offline capable after first load

---

### Bug Fixes (during development)

| Commit | Fix |
|---|---|
| `75ba7355` | JS syntax error — spread-in-ternary in PDF builder (9 instances: `st(cond ? ...C.x : ...C.y)` → `st(...(cond ? C.x : C.y))`) |
| `6296710a` | PDF export crash — jsPDF color args passed as array instead of spread (13 instances: `st([r,g,b])` → `st(r,g,b)`) |
| `f1102e5407` | DDL parser — balanced paren body extraction; parser was stopping at first `)` in `DEFAULT NOW()` or `REFERENCES table(id)`, seeing only the first column |
| `9f3fd5c2` | DB platform moved to Step 1; clickable score tiles; Solutions tab; score hint icon |
| `cc6c2c95` | Platform dropdown locks on DDL entry; removed per-schema platform dropdown from Step 2 |
| `cda3b0db` | Schema Intelligence card; nav cleanup; footer; font size 16px |
| `fe7c7aff` | Right-side blank space fixed — removed `max-width: 860px` cap on main content |
| `2998dcbd` | How It Works moved to horizontal steps banner |

---

## [Upcoming — v2.0.0]

See [ROADMAP.md](ROADMAP.md) for planned features.
