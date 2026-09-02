# Ugrow — Vercel deployment

Static site. No build step, no dependencies. Everything in this folder IS the site;
Vercel serves it from the project root exactly as-is.

```
/                 -> index.html
/work             -> work.html
anything else     -> 404.html
```

---

## Deploy — pick one

### A. Drag and drop (fastest, no git)

1. Zip the **contents** of this folder (not the folder itself — `index.html` must be
   at the top level of the zip).
2. Go to https://vercel.com/new → drop the zip.
3. Framework preset: **Other**. Build command: leave empty. Output directory: leave empty.
4. Deploy.

### B. Vercel CLI

```bash
npm i -g vercel
cd "ugrow in vercel"
vercel          # first run: links/creates the project, deploys a preview
vercel --prod   # push it live
```

### C. Git (recommended once it's live — every push redeploys)

```bash
cd "ugrow in vercel"
git init
git add .
git commit -m "Ugrow landing page"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

Then in Vercel: **Add New → Project → import the repo**. Framework preset **Other**,
build command empty, output directory empty.

---

## Custom domain (ugrow.agency)

Vercel project → **Settings → Domains** → add `ugrow.agency` and `www.ugrow.agency`.
Set the apex (`ugrow.agency`) as primary and let Vercel redirect `www` to it.
Vercel prints the exact DNS records to add at your registrar. HTTPS is issued
automatically — nothing to configure.

`vercel.json` also carries a `www.ugrow.agency → ugrow.agency` redirect as a backstop,
which mirrors what the Hostinger `.htaccess` did. Harmless to keep either way.

---

## What `vercel.json` replaces from `.htaccess`

| `.htaccess` rule            | On Vercel                                                     |
|-----------------------------|---------------------------------------------------------------|
| Force HTTPS                 | Automatic — every Vercel domain is HTTPS-only                  |
| `www` → apex                | Domain settings + the redirect in `vercel.json`                |
| `/work` serves `work.html`  | `"cleanUrls": true`                                            |
| Strip `.html` from URLs     | `"cleanUrls": true` (308s `/work.html` → `/work`)              |
| `/index.html` → `/`         | Explicit redirects in `vercel.json`                            |
| `ErrorDocument 404`         | Automatic — `404.html` at the root is used for unknown paths   |
| gzip / deflate              | Automatic (gzip + brotli on the edge)                          |
| MIME types                  | Automatic                                                      |
| `Accept-Ranges` for mp4     | Automatic — range requests work, videos seek and stream        |
| Security headers            | `headers` block in `vercel.json`                               |
| Browser caching             | `headers` block in `vercel.json` — see the note below          |

**Caching note.** The `.htaccess` cached images and video for 1 year. Filenames here
aren't content-hashed, so a 1-year cache means a returning visitor keeps seeing the old
`prateek-hero.jpg` long after you replace it. This config uses 7 days plus
`stale-while-revalidate` instead: still served instantly from cache, but swaps in a new
image within a day or so of you deploying it. HTML is always revalidated, so text and
copy changes are live the moment the deploy finishes.

---

## Videos

`/videos` only holds `README.txt` right now — the `.mp4` files were never added. The
pages fall back to the poster stills, so nothing looks broken. Drop the clips in with
the exact filenames listed in `videos/README.txt` and redeploy when you have them.

---

## Before you go live on a `*.vercel.app` URL

Both pages declare `<link rel="canonical" href="https://ugrow.agency/...">` and
`robots.txt` allows indexing. That's correct once the real domain is attached. If the
site will sit on `ugrow.vercel.app` for a while and you'd rather Google not index that
preview URL, add this to `<head>` in `index.html` and `work.html` and remove it when
you point the domain:

```html
<meta name="robots" content="noindex">
```

---

## Keeping this in sync with `public_html/`

`public_html/` (the Hostinger copy) is still the other deploy target. This folder is a
straight copy of it minus `.htaccess`, plus `vercel.json`. If you edit one, copy the
changed file to the other — or drop `public_html/` once Vercel is the only host.
