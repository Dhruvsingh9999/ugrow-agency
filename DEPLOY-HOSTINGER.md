# Deploying ugrow.agency on Hostinger

Everything that goes live is inside **`public_html/`**. Nothing else in this
folder gets uploaded.

---

## Before you upload — 2 things to fix

These are the only placeholders left. The site works without them, but it
won't be finished.

### 1. The Calendly link (most important)

I wired the CTA to a guessed URL because you didn't give me the real one:

```
https://calendly.com/ugrow/strategy-call
```

Replace it with your actual booking URL. From this folder:

```bash
cd "public_html" && sed -i 's|https://calendly.com/ugrow/strategy-call|YOUR_REAL_LINK|g' index.html work.html
```

It appears **twice** (once in `index.html`, once in `work.html`).
If that link is wrong, every "Book a strategy call" button is dead.

### 2. Client logos

`public_html/images/logos/brand-1.svg` … `brand-6.svg` currently render as
dashed "YOUR LOGO 1–6" boxes. I did **not** invent client names — putting
made-up brands in a "Trusted by" strip is claiming clients you may not have.

Replace each file with a real client logo (SVG or transparent PNG, keep the
same filename). If you don't have six clients yet, delete the extra `<li>`
rows from the `.logo-row` list in `index.html`, or delete the whole
`<section class="trust">` block.

---

## Uploading to Hostinger

### Option A — File Manager (easiest)

1. **hPanel → Files → File Manager**
2. Open the **`public_html`** folder that's already there and **delete
   everything inside it** (Hostinger puts a `default.php` placeholder page
   there — leave that and it may show instead of your site).
3. On your computer, go *inside* this project's `public_html` folder, select
   all the contents, and zip them → `site.zip`.
   **Zip the contents, not the folder.** If you zip the folder itself your
   site lands at `ugrow.agency/public_html/` and shows a 404.
4. Upload `site.zip` into `public_html`, right-click it → **Extract**.
5. Delete `site.zip`.

You should end up with `index.html` sitting *directly* in `public_html` —
i.e. `public_html/index.html`, not `public_html/public_html/index.html`.

### Option B — FTP (better for the video files)

Hostinger's File Manager gets slow with large media. For the reels use FTP.

1. **hPanel → Files → FTP Accounts** — copy the host, username, port (21).
2. Connect with FileZilla.
3. Drag the **contents** of `public_html/` into the remote `public_html/`.

### Turn on SSL

**hPanel → Security → SSL** → install the free Let's Encrypt certificate for
`ugrow.agency`. Wait for it to say *Active* before testing.

The included `.htaccess` force-redirects `http → https`. If you upload before
the certificate is active you'll get a redirect loop — install SSL first, or
comment out the HTTPS block until it's ready.

---

## The videos

`public_html/videos/` is empty. It needs **14** files, in two sets:

```
reel-01.mp4  reel-03.mp4  reel-05.mp4  reel-07.mp4  reel-09.mp4   ← 9:16 vertical
reel-02.mp4  reel-04.mp4  reel-06.mp4  reel-08.mp4  reel-10.mp4   ← 16:9 horizontal

work-01.mp4  work-02.mp4  work-03.mp4  work-04.mp4                ← 4:5 vertical
```

The `reel-*` set feeds the marquee on the landing page and alternates
vertical/horizontal, so orientation matters. The `work-*` set is the "After"
half of each Before → After pair on `work.html`. Full notes are in
`public_html/videos/README.txt`.

**Keep each file under 3 MB.** They autoplay on scroll — ten 20 MB files will
stall the page on mobile data, which is most of your traffic. H.264 MP4, 30fps,
Handbrake "Vimeo/YouTube 720p30" at RF 26 works well.

Until you upload them, the still poster images show and the page looks fine.

---

## What I changed from your original file

Your original is untouched at `ugrow-landing-page_9.html` in this folder, as a
backup. `public_html/index.html` is the deploy copy, with these changes:

**Renamed to `index.html`** — Hostinger serves `index.html` by default. A file
called `ugrow-landing-page_9.html` would have meant visitors landing on a
directory listing or a 404.

**Localised the video posters — 36 MB → 0.32 MB.** This was the big one. Your
posters were hotlinked to a Higgsfield CDN
(`d8j0ntlcm91z4.cloudfront.net`) as **7 MB PNGs each**. Two problems: the page
was pulling ~36 MB of images on every visit, and those are generation URLs on
someone else's bucket — when they expire, your portfolio silently goes blank.
I downloaded all seven, cropped each to the exact aspect ratio its tile uses
(9:16 or 16:9, matching `object-fit: cover` so no downloaded pixel is wasted),
and saved them as progressive JPEGs in `images/posters/`. Now self-hosted and
115× smaller.

**Your `work.html` is now wired in.** The "See the work" and "See all work"
buttons had pointed at a page that didn't exist. Your Before → After page now
fills that slot, with the same production treatment as `index.html`: its 12
hotlinked CDN images localised, the dead `#BOOKING_LINK` and `href="#"` fixed,
`index.html` links changed to `/`, and social/canonical/favicon tags added.

**Wired up the booking CTA.** `href="#BOOKING_LINK"` was a placeholder that
did nothing. Now points at Calendly with `target="_blank" rel="noopener"`.
(The nav and hero "Book a call" buttons still scroll down to the closing
section rather than jumping straight out — that section sells the call before
asking for it, so it converts better. Left as designed.)

**Graceful fallbacks for the photos.** The two Prateek images carry an
`onerror` that removes the broken `<img>` and lets your existing `.photo` CSS
placeholder show through, so a missing file never renders as a broken-image
icon. (Both real photos are now in place — this is just a safety net.)

**Your four photos, installed and optimised** (see the section below).

**SEO and social.** Added canonical URL, Open Graph and Twitter card tags, plus
a 1200×630 share image (`images/og-cover.jpg`). Without these, the link pasted
into WhatsApp or LinkedIn shows a bare URL with no preview — which matters a
lot for how you'll actually distribute this.

**Favicons** — `favicon.ico`, `favicon.svg`, `apple-touch-icon.png`, 192/512
PWA icons and `site.webmanifest`, built from your ugrow wordmark and green
arrow.

**`.htaccess`** — force HTTPS, `www` → root domain, clean URLs (`/work` not
`/work.html`, with `.html` and `/index.html` 301-ing to the canonical form),
gzip, a year of caching on static assets, correct MIME types, `Accept-Ranges`
so videos can seek instead of downloading whole, and basic security headers.

**A branded 404 page** instead of Hostinger's default error screen.

**`robots.txt` + `sitemap.xml`** pointing at `https://ugrow.agency/`.

Also added `width`/`height` on images to stop layout shift, `fetchpriority="high"`
on the hero, and lazy loading on the logo strip.

---

## Your photos — where each one went

All four are installed in `public_html/images/`, re-encoded as progressive
JPEGs. **4.1 MB → 0.6 MB** with no visible quality loss.

| Source you gave me | Now lives at | Used by | Size |
|---|---|---|---|
| `prateek-hero.jpg` | `images/prateek-hero.jpg` | Hero portrait (4:5) | 402 KB → 179 KB |
| `prateek-headshot.jpg` | `images/prateek-headshot.jpg` | 64px circle, closing card | 57 KB → 23 KB |
| `prateek-founder.jpg` | `images/prateek-founder.jpg` | `reel-10` poster tile | 462 KB → 185 KB |
| `main-image.png` | `images/main-image.jpg` | *nothing yet — see below* | 3.1 MB → 169 KB |

Your originals are untouched in the project root. They sit outside
`public_html`, so they won't upload.

**Two things you should know:**

**`prateek-founder.jpg` was a screenshot, not a clean photo.** It had a black
letterbox bar across the bottom, a garbled watermark ("Saprian Martoartor
Thasiolod Mapftonigay / Bapo:424,225"), and a circular zoom button burnt into
the bottom-right corner. All of that would have shipped to your live site. I
cropped the frame above the UI so the photo is clean — but if you have the
original export without the overlay, use that instead; you'll get back the
bottom fifth of the image.

**`main-image.png` isn't used anywhere yet.** It's your strongest portrait, but
nothing in the HTML references it. Two options: swap it in as the hero (replace
`prateek-hero.jpg`), or build the founder section — the stylesheet already has
`.founder` CSS at line 203 of `index.html` with no matching markup, so that
section was designed and never built. Say the word and I'll do either. If you
want neither, delete `images/main-image.jpg` to save 169 KB.

The two reel tiles that had grey placeholder posters (`reel-08`, `reel-10`) now
use face-safe 16:9 crops from your hero and founder shots, so they read as real
work rather than empty slots.

### And the `work.html` you added

Same treatment as the landing page. It was hotlinking the **same Higgsfield CDN
PNGs** — 12 image slots pulling roughly **62 MB** per visit, from URLs that will
eventually expire. Those are now five local files in `images/work/`, cropped to
the exact ratios your CSS asks for (4:5 for the Before/After pairs, 4:3 for the
CRM row), totalling **251 KB**.

The three CRM tiles are separate files even though two came from the same
source image, so you can swap in a real WhatsApp screenshot, a real ads
dashboard and a real booking confirmation one at a time:

```
images/work/before.jpg          the "Before" still, all four pairs
images/work/after-poster.jpg    poster behind work-01…04.mp4
images/work/crm-whatsapp.jpg    "A lead entering the WhatsApp sequence"
images/work/crm-dashboard.jpg   "Cost per qualified booking"
images/work/crm-booking.jpg     "Viewing booked from inside the chat"
```

Worth knowing: all four Before/After rows currently use the *same* before-image
and the *same* after-poster, so the section reads as one example repeated four
times. Four distinct property shots would make it land much harder.

---

## After it's live — check these

- [ ] `https://ugrow.agency` loads (not a directory listing, not Hostinger's placeholder)
- [ ] `http://ugrow.agency` redirects to `https://`
- [ ] `www.ugrow.agency` redirects to `ugrow.agency`
- [ ] `/work` loads the portfolio page
- [ ] `/anything-fake` shows the branded 404, not Hostinger's
- [ ] Favicon appears in the browser tab
- [ ] Paste the URL into WhatsApp — the preview card shows the cover image
- [ ] Every "Book a strategy call" opens the right Calendly page
- [ ] Open it on an actual phone and scroll the whole way down

Then submit `https://ugrow.agency/sitemap.xml` in Google Search Console.

---

## One thing worth knowing

I could only verify this locally against a test server — I haven't seen it on
Hostinger's actual stack. The `.htaccess` uses standard Apache 2.4 directives
that LiteSpeed (what Hostinger runs) supports, but if you get a **500 error**
right after uploading, the `.htaccess` is the cause. Rename it to
`htaccess.txt` to confirm the site comes back, then re-add the blocks one at a
time to find the one your plan doesn't allow.
