# Changelog — DB Schema Analyzer

---

## [1.0.0] — 2026-06-14

### Features
- 40+ rule engine across 10 categories
- DDL parser with balanced-paren body extraction
- Domain detection (7 domains) with priority weighting
- Multi-schema support (up to 5 schemas)
- Query anti-pattern detection
- Health score (weighted composite, 0–100, A–F grade)
- Clickable score strip tiles to filter issues
- Score hint ⓘ popover (formula, weights, grade scale)
- Results tabs: Issues · Summary · Solutions · Comparison · Roadmap
- Schema Intelligence card in Summary tab
- 4-phase remediation roadmap
- PDF export (9 sections + Appendix, A4, client-side jsPDF)
- 4 built-in sample schemas
- File upload (drag-and-drop + click)
- Platform selector in Step 1 with auto-lock on DDL entry
- Overview strip: "What is DB Schema Analyzer?" with horizontal steps
- How It Works banner (5-step pipeline above footer)
- Footer: tool description, categories, platforms
- Single column layout, full width content
- Font size 16px base

### Bug Fixes

| Commit | Fix |
|---|---|
| `75ba7355` | JS syntax error — spread-in-ternary in PDF builder (9 instances) |
| `6296710a` | PDF export crash — jsPDF color args passed as array (13 instances) |
| `f1102e54` | DDL parser — balanced paren body extraction (stopped at first `)`) |
| `fe7c7aff` | Right-side blank space — removed max-width cap on main content |
| `dab99293` | Step cards not full width — removed max-width from main content |

### UI Changes (chronological)

| Commit | Change |
|---|---|
| `9f3fd5c2` | DB platform in Step 1, clickable tiles, Solutions tab, score hint |
| `cc6c2c95` | Platform lock on DDL entry, removed per-schema platform dropdown |
| `cda3b0db` | Schema Intelligence card, nav cleanup, footer, font 16px |
| `2998dcbd` | How It Works horizontal steps banner |
| `7994e52f` | Overview strip above HIW, HIW heading centered |
| `ab6653d3` | How It Works moved above footer, overview center aligned |
| `949aa309` | Two-column overview: left steps + right schema info |
| `15a63116` | Left steps commented out, horizontal steps below description |
| `420c5824` | Left nav sidebar removed, single column layout, DB icon removed |
| `dab99293` | All 3 step cards full width |

---

## [Upcoming — v2.0.0]

See [ROADMAP.md](ROADMAP.md).
