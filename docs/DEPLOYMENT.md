# Deployment — DB Schema Analyzer

## Hosting

**Platform:** Vercel Static
**Repo:** https://github.com/np04799/db-analyzer
**Live URL:** https://db-analyzer-two.vercel.app/
**Docs URL:** https://db-analyzer-two.vercel.app/docs.html
**Branch:** `main` — Vercel auto-deploys on every push

---

## Vercel Configuration (`vercel.json`)

```json
{
  "version": 2,
  "builds": [
    { "src": "index.html", "use": "@vercel/static" },
    { "src": "docs.html",  "use": "@vercel/static" }
  ],
  "routes": [
    { "src": "/docs.html",    "dest": "/docs.html" },
    { "src": "/assets/(.*)", "dest": "/assets/$1" },
    { "src": "/(.*)",         "dest": "/index.html" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options",  "value": "nosniff" },
        { "key": "X-Frame-Options",          "value": "DENY" },
        { "key": "X-XSS-Protection",         "value": "1; mode=block" },
        { "key": "Referrer-Policy",          "value": "strict-origin-when-cross-origin" },
        { "key": "Cache-Control",            "value": "public, max-age=3600" }
      ]
    }
  ]
}
```

**Important:** `docs.html` must be listed before the `(.*)` catch-all route,
otherwise Vercel serves `index.html` for all paths including `/docs.html`.

---

## GitHub API Commit Workflow

All commits use the GitHub Contents API directly (no `git` CLI required):

```python
# Step 1: Get current SHA (always fetch fresh — stale SHAs cause 409 conflicts)
GET https://api.github.com/repos/{REPO}/contents/{path}?ref=main
→ sha = response['sha']

# Step 2: Base64-encode the new content
content = base64.b64encode(file_bytes).decode()

# Step 3: PUT the updated file
PUT https://api.github.com/repos/{REPO}/contents/{path}
Body: { "message": "...", "content": content, "sha": sha, "branch": "main" }
```

**Token:** Stored in Claude's memory for this project.
**Critical:** Never commit without explicit user approval.

---

## Development Workflow

No build step. Edit `index.html` locally, test via Node.js simulation:

```bash
# Syntax check
node --check index.html  # (after extracting <script> content)

# Simulate analysis
node -e "
const html = fs.readFileSync('index.html', 'utf-8');
// extract largest <script> block
// mock DOM (getElementById, querySelector, etc.)
// call parseDDL(), runRules(), renderAll()
"
```

---

## File Sizes (v1.1.0)

| File | Size |
|------|------|
| `index.html` | ~180KB |
| `docs.html` | ~42KB |
| `vercel.json` | <1KB |
| `assets/DB_Schema_Analyzer_Presentations.zip` | ~85KB |

---

## Deployment Checklist

Before committing:
- [ ] JS syntax valid (`node --check`)
- [ ] All new rules tested with Node.js simulation
- [ ] Regression checks pass (existing rules unchanged)
- [ ] File size reasonable (< 250KB for index.html)
- [ ] Verified on live site after Vercel deploy (~30s)

---

## Environment Variables

None required. The tool has no backend, no API keys, and no environment configuration.
The jsPDF library loads from CDN on first use:
```
https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js
```
