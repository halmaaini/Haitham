# Haitham

this will be for public pages

## /watch — Meridian scroll-scrub site

`watch.html` is a single-page site whose hero scrubs a pre-rendered assembly
video (`watch-desktop.mp4` / `watch-mobile.mp4`) with scroll position.

**File placement.** Keep `watch.html`, `watch-desktop.mp4`, `watch-mobile.mp4`
and `poster.jpg` together in the same directory. The page picks the mobile or
desktop encode at load time based on viewport width and pointer type.

**Poster.** `poster.jpg` is the first frame of the video. To regenerate it:

```bash
ffmpeg -i watch-desktop.mp4 -frames:v 1 -q:v 2 poster.jpg
```

**Must be served over HTTP.** Frame-accurate seeking relies on HTTP range
requests, so opening `watch.html` directly from disk (`file://`) will not
work — the video will fail to load. Any static server is fine locally:

```bash
python3 -m http.server 8000
# → http://localhost:8000/watch.html
```

On Vercel (this repo's host) it is served at `/watch` via `cleanUrls`.

**Do not re-encode the videos** without preserving the all-intra keyframe
settings (`-g 1 -keyint_min 1 -sc_threshold 0`) — a keyframe on every frame is
what makes scrubbing frame-accurate.
