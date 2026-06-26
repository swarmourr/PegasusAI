# PegasusAI Website

Marketing and documentation website for **Pegasus AI** — a next-generation AI-driven scientific workflow management system developed by USC ISI, UTK, RENCI, and UMass, funded by the U.S. National Science Foundation (NSF Award #2513101).

Live site: [https://pegasus-isi.github.io/pegasus-ai/](https://pegasus-isi.github.io/pegasus-ai/)

---

## Project Structure

```
/
├── index.html              # Main landing page
├── blog.html               # Blog / coming soon page
├── 404.html                # Custom 404 error page
├── robots.txt              # SEO crawler rules
├── sitemap.xml             # SEO sitemap
├── .nojekyll               # Disables Jekyll processing on GitHub Pages
├── assets/
│   ├── img/                # Images (logo, partner logos)
│   └── data/
│       ├── team.json       # Team member data
│       └── posts.json      # Blog post metadata
└── blog/
    └── archive/            # Archived blog posts (Markdown)
```

---

## Tech Stack

- **HTML / CSS / Vanilla JS** — no build step required
- **Tailwind CSS** — loaded via CDN
- **Lucide Icons** — loaded via CDN
- **Particles.js** — background particle animation
- **Google Fonts** — Outfit + JetBrains Mono
- **Marked.js** — Markdown rendering for blog posts

---

## Local Development

No build step or package manager needed. Just open the file in a browser:

```bash
# Option 1 — Open directly
open index.html

# Option 2 — Serve locally (recommended to avoid CORS issues with fetch)
npx serve .
# or
python3 -m http.server 8080
# then visit http://localhost:8080
```

> **Note:** Opening `index.html` directly via `file://` may cause CORS errors when fetching `assets/data/team.json` or `assets/data/posts.json`. Use a local server instead.

---

## Deployment

### GitHub Pages

This site is deployed via **GitHub Pages** from the `main` branch root.

**First-time setup:**

1. Go to your repo on GitHub
2. Navigate to **Settings → Pages**
3. Under **Source**, select `Deploy from a branch`
4. Set **Branch** to `main` and folder to `/ (root)`
5. Click **Save**

GitHub will publish the site at:
```
https://pegasus-isi.github.io/pegasus-ai/
```

**Deploy an update:**

```bash
git add .
git commit -m "your message"
git push origin main
```

Deployment typically takes 1–2 minutes. Check the **Actions** tab on GitHub to monitor progress.

---

### Cloudflare Pages (Recommended for Production)

Cloudflare Pages offers faster global CDN, free custom domains, and automatic HTTPS.

**First-time setup:**

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages**
2. Click **Create application → Pages → Connect to Git**
3. Select your GitHub repo (`pegasus-ai`)
4. Configure the build settings:

| Setting | Value |
|---|---|
| Production branch | `main` |
| Build command | *(leave empty)* |
| Build output directory | `/` (root) |

5. Click **Save and Deploy**

Cloudflare will assign a URL like:
```
https://pegasusai.pages.dev
```

**Connect a custom domain:**

1. In Cloudflare Pages → your project → **Custom domains**
2. Click **Set up a custom domain**
3. Enter your domain (e.g. `pegasusai.com`)
4. Follow the DNS instructions to point your domain to Cloudflare Pages

**Deploy an update:**

```bash
git add .
git commit -m "your message"
git push origin main
```

Cloudflare Pages automatically detects the push and redeploys in ~30 seconds.

---

## Updating Content

### Team members
Edit [`assets/data/team.json`](assets/data/team.json). Each entry follows this shape:

```json
{
  "name": "Full Name",
  "role": "Title",
  "institution": "University / Org",
  "image": "https://...",
  "links": {
    "github": "https://github.com/...",
    "website": "https://..."
  }
}
```

### Blog posts
1. Add a Markdown file to `blog/archive/`
2. Register it in [`assets/data/posts.json`](assets/data/posts.json)

---

## License

NSF-funded open research project. See individual source files for details.
