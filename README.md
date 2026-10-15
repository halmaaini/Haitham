# Haitham

Personal site for Haitham Al Maaini.

## Home page

`index.html` is a single-page site whose hero scrubs a pre-rendered assembly
video with scroll position. There are three encodes of the same 20s video:

| File | Size | Served to |
|---|---|---|
| `watch-desktop.mp4` | 1920×1080 | desktops and large screens |
| `watch-portrait.mp4` | 810×1080, centre crop of the desktop master | phones held upright |
| `watch-mobile.mp4` | 1280×720 | phones held sideways |

An upright phone only ever shows the centre of a 16:9 frame (`object-fit:
cover`), so the portrait file spends every pixel on what is actually visible.
The page picks the encode before setting `src` and swaps it on rotation.

**File placement.** Keep `index.html`, the three videos, `poster.jpg` and the
`fonts/` directory together in the same directory. Fonts are self-hosted so the
loader paints without waiting on a third-party round trip.

**Poster.** `poster.jpg` is the final frame of the video (the finished watch).
To regenerate it:

```bash
ffmpeg -sseof -0.2 -i watch-desktop.mp4 -frames:v 1 -update 1 -q:v 2 poster.jpg
```

**Must be served over HTTP.** Frame-accurate seeking relies on HTTP range
requests, so opening `index.html` directly from disk (`file://`) will not
work — the video will fail to load. Any static server is fine locally:

```bash
python3 -m http.server 8000
# → http://localhost:8000/
```

**Do not re-encode the videos** without preserving the all-intra keyframe
settings (`-g 1 -keyint_min 1 -sc_threshold 0`) — a keyframe on every frame is
what makes scrubbing frame-accurate. The portrait file is cut from the desktop
master with a chroma offset that keeps the gold dial numerals from washing out:

```bash
ffmpeg -i watch-desktop.mp4 -an -vf "crop=810:1080:555:0" \
  -c:v libx264 -preset slow -crf 24 \
  -x264-params "chroma-qp-offset=-4:aq-mode=3:aq-strength=0.9:deblock=-1,-1" \
  -g 1 -keyint_min 1 -sc_threshold 0 -pix_fmt yuv420p -profile:v high \
  -movflags +faststart watch-portrait.mp4
```

**Sharing and search.** `og.jpg` is the 1200×630 link-preview card. The
favicon set (`favicon.ico`, `favicon-32.png`, `icon-192.png`,
`apple-touch-icon.png`) is the "HA" monogram. `robots.txt` and `sitemap.xml`
point search engines at `https://www.almaaini.com/`, and `404.html` is the
not-found page. The page head carries schema.org JSON-LD (`Person`, `WebSite`,
`ProfilePage`) linking the site to the LinkedIn profile, and `llms.txt` is a
plain-language summary for AI assistants.

**Security headers** are set in `vercel.json`. The Content-Security-Policy only
applies on `www.almaaini.com`, so Vercel's preview toolbar keeps working on
preview deployments. If you add a third-party script, font or embed, add its
origin to that policy or it will be blocked.

## Archive

`archive/` holds retired pages (the old AI resources hub, the UAE AI policy
register, and the AI prompts & learning page). They stay in the repo for
reference but are excluded from deployment via `.vercelignore`, and their old
URLs redirect to the home page (`vercel.json`).
