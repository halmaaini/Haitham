# Haitham

Personal site for Haitham Al Maaini.

## Home page

`index.html` is a single-page site whose hero scrubs a pre-rendered assembly
video (`watch-desktop.mp4` / `watch-mobile.mp4`) with scroll position.

**File placement.** Keep `index.html`, `watch-desktop.mp4`, `watch-mobile.mp4`,
`poster.jpg` and the `fonts/` directory together in the same directory. The
page picks the mobile or desktop encode at load time based on viewport width
and pointer type. Fonts are self-hosted so the loader paints without waiting
on a third-party round trip.

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
what makes scrubbing frame-accurate. The mobile file is made from the desktop
master with a chroma offset that keeps the gold dial numerals from washing out:

```bash
ffmpeg -i watch-desktop.mp4 -an -vf "scale=1280:720:flags=lanczos" \
  -c:v libx264 -preset slow -crf 29 \
  -x264-params "chroma-qp-offset=-6:aq-mode=3:aq-strength=0.9:deblock=-1,-1" \
  -g 1 -keyint_min 1 -sc_threshold 0 -pix_fmt yuv420p -profile:v high \
  -movflags +faststart watch-mobile.mp4
```

**Sharing and search.** `og.jpg` is the 1200×630 link-preview card. The
favicon set (`favicon.ico`, `favicon-32.png`, `icon-192.png`,
`apple-touch-icon.png`) is the "HA" monogram. `robots.txt` and `sitemap.xml`
point search engines at `https://www.almaaini.com/`, and `404.html` is the
not-found page.

## Archive

`archive/` holds retired pages (the old AI resources hub, the UAE AI policy
register, and the AI prompts & learning page). They stay in the repo for
reference but are excluded from deployment via `.vercelignore`, and their old
URLs redirect to the home page (`vercel.json`).
