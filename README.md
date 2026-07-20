# KBW2026 Attendee Toolkit

Client-side card makers for **Korea Blockchain Week 2026 (with Upbit)**. Two tools — an
**Attendee Badge** (lanyard ID) and a **"See you at KBW2026"** social card — served as a
static site on GitHub Pages. No backend: everything (photo upload, background removal,
compositing, download) runs in the visitor's browser. No image or personal data leaves the device.

## Live URLs

| URL | Page |
| --- | --- |
| `https://kbw2026-with-upbit.github.io/attendee-card/` | Landing (choose a tool) |
| `.../attendee-card/badge/` | Attendee Badge maker |
| `.../attendee-card/seeyou/` | "See you at KBW2026" card maker |

`badge.html`, `see.html`, `classic.html` are thin **redirect stubs** kept only so previously
shared links still work (→ `/badge/`, `/seeyou/`, `/seeyou/`). Don't edit them for content.

## Project layout

```
index.html            ← landing page (the 3 full pages below are the only ones to edit)
badge/index.html      ← Attendee Badge maker      (canonical, clean URL /badge/)
seeyou/index.html     ← See-you card maker        (canonical, clean URL /seeyou/)
badge.html            ← redirect → ./badge/       (stub, do not edit)
see.html, classic.html← redirect → ./seeyou/      (stubs, do not edit)
.nojekyll             ← REQUIRED: lets GitHub Pages serve underscore-prefixed asset files

# Shared assets (referenced from pages via ../ inside /badge/ and /seeyou/):
_tpl_1x1.png, _tpl_9x16.png        ← Badge background templates (1200², 900×1600)
_see1x1.png, _see9x16.png          ← See-you background templates (1200², 900×1600)
_seemask_1x1.png, _seemask_9x16.png← See-you photo-frame (Z) masks
_qr_default.svg                    ← Default Telegram/KBW QR (with center logo)
_thumb_badge.png, _thumb_see.png   ← Landing card previews (actual default renders)
_logo.png                          ← "with Upbit KBW2026" logo, recoloured for dark bg
_og.png                            ← 1200×630 social share image (og:image / twitter:image)
_favicon.png                       ← browser tab icon
```

Only three files hold real logic — `index.html`, `badge/index.html`, `seeyou/index.html`.
Each is self-contained (embedded Neue Montreal fonts + inline CSS/JS); there is no build step.

## How the makers work

Each card is a **template-overlay composite** drawn to a `<canvas>`:

1. Draw the pre-designed background **template PNG** (frame, stickers, headings, Seoul photos…).
2. Composite the user's photo: the image is **cover-fit and clipped to the photo box** (a
   rectangle in the template) in full colour — no background removal. Zoom/drag reposition it.
   When empty, the template's own placeholder shows. (The See-you maker still uses the
   `@imgly/background-removal` pop-out flow for now — see below — until its square-box template
   lands, after which it will match the badge.)
3. Cover the template's placeholder text areas and **re-draw** the user's name / title / company
   (and interest/hashtag chips) at fixed coordinates.
4. Export via `canvas.toBlob()` → PNG download at native size (1200²/900×1600).

All layout numbers (frame box, text baselines, chip anchors, colours) live in the `CFG` object
near the top of each maker's `<script>`. **If a template PNG is re-exported at a different size or
layout, those coordinates must be re-measured.** They are pixel-mapped to the current templates.

Required fields (photo, name, title/role, company) are validated before download; interests/QR are
optional.

## Local development

```bash
python3 -m http.server 8000     # from the repo root
# open http://localhost:8000/  (use a server, not file://, or same-origin asset loads fail)
```

## Deploy

GitHub Pages builds from the `main` branch of `KBW2026-with-Upbit/attendee-card`. Push to `main`
and Pages redeploys in ~1–2 min. `.nojekyll` must stay — without it Jekyll drops every `_*.png`
asset (they'd 404).

### Custom domain (optional)
To serve at e.g. `attendee-card.koreablockchainweek.com`: add a DNS **CNAME** record
(`attendee-card` → `kbw2026-with-upbit.github.io`, DNS-only if behind Cloudflare), add a `CNAME`
file to the repo root, and set the custom domain in Pages settings.

## Notes / known trade-offs

- **Fonts** (Neue Montreal, ~74 KB base64) are embedded per page rather than served as a shared
  cached file — simplest for a no-build static site; swap to a linked `woff2` if page weight matters.
- **`esm.sh` dependency** is the only external runtime call (background-removal model). Everything
  else is same-origin. Consider self-hosting the library if you need to remove third-party calls.
- Social platforms cache link previews; use X/Facebook card debuggers to force-refresh `og:image`.
