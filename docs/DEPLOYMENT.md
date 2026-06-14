# Deployment Guide — DB Schema Analyzer v1.0

## Live Environment

| Property | Value |
|---|---|
| URL | https://db-analyzer-two.vercel.app/ |
| Platform | Vercel Static |
| GitHub Repo | https://github.com/np04799/db-analyzer |
| Branch | `main` |
| Auto-deploy | Yes — every push to `main` triggers a Vercel deployment |

---

## Vercel Configuration (`vercel.json`)

```json
{
  "version": 2,
  "builds": [{ "src": "index.html", "use": "@vercel/static" }],
  "routes": [{ "src": "/(.*)", "dest": "/index.html" }],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Cache-Control", "value": "public, max-age=3600" }
      ]
    }
  ]
}
```

---

## Deploy via GitHub (recommended)

Auto-deploy is already configured. To deploy a change:

```bash
# 1. Make changes to index.html locally

# 2. Commit and push to main
git add index.html
git commit -m "feat: describe your change"
git push origin main

# 3. Vercel detects the push and deploys automatically
#    Deployment takes ~30 seconds
#    Live at: https://db-analyzer-two.vercel.app/
```

---

## Deploy via Vercel CLI

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy to production
cd db-analyzer
vercel --prod
```

---

## Initial Vercel Project Setup (one-time)

1. Go to https://vercel.com/new
2. Click **Import Git Repository** → select `np04799/db-analyzer`
3. Configure:
   - **Framework Preset:** Other
   - **Root Directory:** `./`
   - **Build Command:** *(leave blank)*
   - **Output Directory:** *(leave blank)*
4. Click **Deploy**

---

## Run Locally

No build step required. Open `index.html` directly in any modern browser:

```bash
# Option 1 — Direct file open
open index.html   # macOS
start index.html  # Windows

# Option 2 — Simple HTTP server (avoids any file:// restrictions)
npx serve .
# or
python3 -m http.server 8080
```

---

## Environment Requirements

| Requirement | Details |
|---|---|
| Browser | Chrome 90+, Edge 90+, Firefox 88+, Safari 14+ |
| JavaScript | ES6+ (required) |
| Internet | Only needed for jsPDF CDN on first load. Offline after. |
| Server | None — fully static |
| Node.js | Not required for production. Optional for local server. |

---

## Branching Strategy

| Branch | Purpose |
|---|---|
| `main` | Production — auto-deploys to Vercel |
| `feature/*` | Feature development — requires PR + approval before merging to main |

**Rule:** Never commit directly to `main` without approval. Always use feature branches and raise a PR.

---

## Monitoring

Vercel provides:
- Deployment logs at https://vercel.com/np04799/db-analyzer-two
- Analytics (if enabled) at the same dashboard
- Domain management for custom domain setup

No application-level error tracking is configured in v1 (no backend = no server logs).
