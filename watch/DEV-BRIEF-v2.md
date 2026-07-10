# Dev Brief — Update `/watch`

You are **updating an existing page**, not rebuilding it. The current implementation is good. It needs new assets, one more phase, and four bug fixes.

Live page: `/watch` (Meridian — Caliber 140 Pocket Chronograph)

---

## 1. Replace the assets

| File | Spec | Note |
|---|---|---|
| `watch-desktop.mp4` | 1920×1080, H.264, all-intra, silent, **20.042s**, 24fps, 26 MB | replaces old 15.1s file |
| `watch-mobile.mp4` | 1280×720, all-intra, silent, 20.042s, 12 MB | |
| `poster.jpg` | final frame | |

Both videos have a keyframe on **every frame** (`-g 1 -keyint_min 1 -sc_threshold 0`). Verified: 480 of 481 frames are keyframes. **Do not re-encode them.** If you must, preserve all-intra or scrubbing breaks.

Brand name changed. The dial reads **ALMAINI**, not Meridian. Update `<title>`, meta description, the loader wordmark, `#title h1`, and the footer.

---

## 2. Four phases become five

The video is four 5-second clips joined seamlessly. Scene keyframes land on **exact quarters**.

| # | Label | Progress |
|---|---|---|
| 01 | Scattered | 0.00 |
| 02 | Converging | 0.25 |
| 03 | Movement | 0.50 |
| 04 | Cased | 0.75 |
| 05 | Complete | 1.00 |

### Code change

Current:
```js
var BOUNDS = [0.33, 0.66, 0.92];
```

Replace with **midpoints**, so each label lights as its frame arrives rather than after it has passed:
```js
var MARKS = [0.125, 0.375, 0.625, 0.875];
```

And the phase lookup:
```js
var i = 0;
for (var k = 0; k < MARKS.length; k++) if (current >= MARKS[k]) i = k + 1;
setPhase(i);
```

Add a fifth `.ph` to `#rail` and a fifth entry to `CAPTIONS`. Suggested copy (rewrite freely, but keep it short):

```js
var CAPTIONS = [
  { n:"01", t:"Every part, at rest." },
  { n:"02", t:"They begin to find each other." },
  { n:"03", t:"The movement takes its shape." },
  { n:"04", t:"Dial, case, sapphire." },
  { n:"05", t:"Wound by hand. Carried for life." }
];
```

Also bump the scroll track: `#hero{height:520vh}` → `600svh` (see bug 3).

---

## 3. Four bugs to fix

### Bug 1 — caption race condition

`setPhase` fires a bare 280ms `setTimeout`. Scroll fast across two boundaries inside that window and two timers land out of order, showing the wrong caption.

```js
var capTimer = null;
function setPhase(i){
  if (i === phase) return;
  phase = i;
  phases.forEach(function(el,k){ el.classList.toggle("on", k===i); });
  caption.classList.remove("show");
  clearTimeout(capTimer);
  capTimer = setTimeout(function(){
    if (phase !== i) return;          // a newer phase already won
    capN.textContent = CAPTIONS[i].n;
    capT.textContent = CAPTIONS[i].t;
    caption.classList.add("show");
  }, 260);
}
```

### Bug 2 — seeking to exactly `duration`

Current:
```js
var t = current * video.duration;
```
At `current === 1` this lands on the very last instant. Browsers clamp inconsistently there — you can get a black frame or trip `ended`. Clamp it:
```js
var t = current * (video.duration - FRAME);   // FRAME = 1/24
```

### Bug 3 — `vh` mixed with `svh`

```css
#hero{height:520vh}          /* large viewport */
#hero .stick{height:100svh}  /* small viewport */
```
`heroProgress()` divides by `innerHeight`. On mobile, when the toolbar retracts, `innerHeight` grows past `svh` but the track was measured in `vh`. Progress drifts and may never cleanly reach 1.0.

Pick one unit. Use `svh` for both: `#hero{height:600svh}`, `#pin{height:100svh}`.

### Bug 4 — caption flickers on load

`reveal()` adds `.show`, then the first `tick()` calls `setPhase(0)`, which immediately strips it and re-adds it 280ms later.

Initialise `phase = 0` (not `-1`) and add `.on` to the first `.ph` in markup or on init.

---

## 4. One performance note

The Google Fonts `<link rel="stylesheet">` is render-blocking. The loader — whose entire job is to appear instantly — waits on a round-trip to `fonts.googleapis.com` before it paints.

Self-host Cormorant Garamond and IBM Plex Mono, or drop to a system serif stack. Not a blocker; it is the difference between a black beat and no black beat.

---

## 5. Leave these alone — they are already correct

- `DAMP = 0.12` on the scrub. The lag is the feel.
- `FRAME = 1/24` seek throttle. Matches the source framerate.
- Choosing the encode **before** setting `src`.
- The Safari decoder warm-up (`play()` then `pause()` on `loadedmetadata`).
- `IntersectionObserver` gating the rAF loop.
- Real buffered progress on the loader, with the 6s timeout fallback.
- `prefers-reduced-motion` collapsing the hero.

---

## 6. What NOT to do

- **No crossfades at the clip joins.** They are already seamless — measured. Median frame-to-frame change is 0.0011; the largest join scores 0.0643. A visible cut scores above 0.30. Adding a dissolve would create the artefact you are trying to hide.
- **No canvas image sequence.** 481 JPEGs is heavier than the MP4 and buys nothing.
- **No `autoplay`.** The video never plays. It is scrubbed.
- **No GSAP/ScrollTrigger dependency** unless the project already has it.
- **Do not re-encode the MP4s** without preserving all-intra.

---

## 7. Definition of done

- Scrolling the hero scrubs smoothly at 60fps on a mid-range laptop.
- All five rail labels light in sequence; captions never show the wrong text when scrolling fast.
- Scrolling to the very bottom lands on the finished watch, not a black frame.
- Phone loads `watch-mobile.mp4`, never the desktop file. Verify in the Network tab.
- iOS Safari scrubs, not just plays.
- Reduced-motion users see the finished watch and never a scrub.
- No console errors.

---

## Known, accepted, not bugs

- **24fps judders under very slow scrolling.** Interpolating to 48fps would fix it and double the file. Deferred.
- **The second scene (the rosette) is the softest moment** in the sequence. It is ~3 seconds in and moving fast. Deferred.
