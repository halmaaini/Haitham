# Build Brief: Scroll-Scrub Luxury Watch Assembly Site

You are building a single-page marketing site for a luxury mechanical chronograph **pocket watch**. The hero is a scroll-driven video scrub: as the user scrolls, ~140 scattered watch components assemble into a finished pocket watch.

The video is already produced. **Do not generate 3D. Do not use Three.js for the hero.** The hero is a pre-rendered MP4 scrubbed by scroll position. This is the same technique scoutmotors.com uses.

---

## Assets you are given

| File | Spec |
|---|---|
| `watch-desktop.mp4` | 1920×1080, H.264, all-intra, silent, ~15.1s, 24fps |
| `watch-mobile.mp4` | 1280×720, H.264, all-intra, silent, same content |
| `poster.jpg` | First frame of the video |

Both videos are encoded with a keyframe on **every frame** (`-g 1 -keyint_min 1 -sc_threshold 0`). This is what makes frame-accurate scrubbing possible. Do not re-encode them without preserving this.

If `poster.jpg` is missing, generate it:
```bash
ffmpeg -i watch-desktop.mp4 -frames:v 1 -q:v 2 poster.jpg
```

---

## The four phases

The video contains four narrative beats. Content copy must be tied to these scroll positions.

| # | Progress | What is on screen |
|---|---|---|
| 01 | 0.00 | Components scattered across the frame, suspended in a black void |
| 02 | 0.33 | The movement has assembled at center — gear train, column wheel, balance |
| 03 | 0.66 | Dial seated, case closed, sapphire crystal lowering |
| 04 | 0.92 | Complete pocket watch, centered, chain hanging below |

Clip boundaries sit at **5.04s** and **10.08s**. They were generated as start-frame → end-frame chains, so the seams are visually seamless (measured scene-change score at the join: 0.0165 vs a 0.3+ threshold for a visible cut). Do not add crossfades. There is nothing to hide.

---

## What to build

### Hero
- A `<video>` element inside a `position: sticky` container.
- The scroll container is roughly `520vh` tall. The video is pinned for its full height.
- Scroll progress `0 → 1` maps to `video.currentTime = progress * duration`.
- `object-fit: cover`, full bleed.
- Radial vignette overlay on top of the video. Nothing behind it.

### Below the hero
Standard scrolling content. Three or four short sections: the movement, the dial, enquiries. Serif body copy, uppercase monospace section labels, generous vertical rhythm.

### Chrome
- A fixed left rail listing the four phases. The active one brightens as the user scrolls.
- A caption near the bottom that swaps text at each phase boundary.
- A 1px progress line along the top edge.
- A "Scroll" cue that fades out once the user moves.

---

## Technical requirements — these are not optional

These are the things that break. Address each one explicitly.

**1. Damp the scrub.**
Do not assign `currentTime` straight from scroll position. Lerp toward it:
```js
current += (target - current) * 0.12;
```
The lag is the entire feel of the effect. Without it the video feels glued to the mouse wheel.

**2. Throttle the seek.**
Only assign `currentTime` when the delta exceeds one frame (`1/60`). Assigning it 60 times a second will saturate the video decoder and stutter.

**3. Pick the encode before setting `src`.**
```js
const small = innerWidth < 820 || matchMedia("(pointer: coarse)").matches;
video.src = small ? "watch-mobile.mp4" : "watch-desktop.mp4";
```
If you set `src` first and swap it later, phones download the desktop file.

**4. Warm the Safari decoder.**
Safari will not seek a video it has never played. On `loadedmetadata`, call `play()` then immediately `pause()`. Wrap it — the promise rejects when autoplay is blocked, and that is fine.

**5. Video attributes.**
`playsinline muted preload="auto"`. No `autoplay`, no `loop`, no `controls`.

**6. Stop the rAF loop when the hero is off screen.**
Use an `IntersectionObserver`. Do not run a render loop for a video nobody can see.

**7. Show a loader.**
Wait for `canplaythrough` before revealing the page. Track real buffered progress via the `progress` event and `video.buffered.end(0)`. Add a timeout fallback (~6s, `readyState >= 2`) so a slow connection does not hang forever.

**8. `prefers-reduced-motion: reduce`.**
Collapse the hero to `100vh`, seek to the final frame, disable the scrub entirely. Hide the phase rail.

**9. Must be served over HTTP.**
Seeking requires HTTP range requests. `file://` will not work. If someone opens the HTML directly, the video will fail to load. Note this in the README.

---

## What NOT to do

- **No `<canvas>` image-sequence approach.** A 363-frame JPEG sequence is heavier than the MP4 and buys nothing here.
- **No GSAP or ScrollTrigger dependency** unless the project already uses it. A `scroll` listener plus `requestAnimationFrame` is ~30 lines and has no bundle cost.
- **No crossfades at the clip joins.** They are already seamless.
- **No Three.js in the hero.** Real-time WebGL cannot match the photoreal quality of the pre-rendered video, and it costs a black loading screen.
- **No `autoplay`.** The video never plays. It is scrubbed.

---

## Optional: an interactive 3D section

Only if the brief calls for interactivity, and only *below* the hero, never as a replacement for it.

- React Three Fiber with `@react-three/drei`.
- Load a real watch GLB (bought from Sketchfab / TurboSquid, or modeled in Blender). Each part must be a separate named object with its origin at its own center. Draco-compress.
- `<Environment preset="studio" />` — the reflection probe is 80% of the "expensive" look.
- Steel: `metalness: 1`, `roughness: 0.15`. Crystal: `MeshPhysicalMaterial`, `transmission: 1`, `ior: 1.77`.
- `ACESFilmicToneMapping`, subtle bloom, black background, one strong rim light.
- Purpose: let the user grab the finished movement and spin it. **Do not re-run the assembly** — that repeats the hero.
- Lazy-load it. The hero must never wait on it.

---

## Deliverables

1. `index.html` (or a component, if the project is a framework)
2. A `README.md` covering: file placement, how to make the poster, and the HTTP-serving requirement
3. Placeholder copy for the four phase captions and the three body sections. Write it restrained — short sentences, specific nouns, no adjective stacking.

## Definition of done

- Scrolling the hero scrubs the video smoothly at 60fps on a mid-range laptop.
- Seams at 5.04s and 10.08s are invisible while scrubbing.
- Phone loads `watch-mobile.mp4`, never the desktop file.
- Safari on iOS scrubs, not just plays.
- Reduced-motion users see the finished watch and never a scrub.
- No console errors.
