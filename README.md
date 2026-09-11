# WA$TED x TRINITY: SHOGUN — Event Landing Page

Promo landing page for **SHOGUN** at The Circle OC, Huntington Beach — Saturday, September 19, 2026.
Presented by WA$TED x TRINITY.

**Live:** https://xbased420.github.io/wasted-shogun/

Single self-contained `index.html`. Inline CSS and JS only; the only external
dependencies are Google Fonts and SoundCloud's official embed.

## What's in it

- Official WA$TED mark embedded as a traced inline SVG (~6.7KB) in the header and footer —
  vector, so it stays crisp at any size, and recolourable via `currentColor`
- Fully procedural hero — SVG turbulence + silhouette, no raster assets
- Animated film grain, drifting scanlines, a CRT roll band and a VHS tracking tear
- Glitching `SHOGUN` headline with RGB channel split and slice displacement
- Kinetic slam-in section headers, pointer parallax, scroll-velocity shake
- Auto-fitting display type — measured against its container, so it can never clip
- Music autoplays on load where the browser allows it; where it doesn't, every
  interaction retries until playback starts. The viewer can always pause from the
  PRESS PLAY control, and a manual pause is never overridden

## Nothing moves with the music

This is the single most important thing to know before editing this file.

The page has **no tempo, no beat clock, no audio analysis and no per-frame
JavaScript.** While the track plays, the background does not scale, breathe,
brighten or thicken. Neither does the type, the waveform, the CTA or the PRESS
PLAY button. Every one of those was built, shipped and then deliberately cut.

Verified with playback forced on and the position swept across the intro, the
first drop, the breakdown and the peak: `.plate__beat` reports `transform: none`
at every sample, `.plate__grade` holds opacity 1, `#grain` holds 0.34, and **no
inline style is written to any element at any point.**

### What was removed, and why

| Removed | Was |
|---|---|
| `camPush`, and later a real analysis-driven version | hero background scaling on the kick |
| `ctaKick` | CTA glow pulsing per bar |
| `stripPulse` | hazard strip tinting per bar |
| `kick`, `pressKick`, `wipe` | PRESS PLAY idle heartbeat, beat ring, light sweep |
| `logoKick`, `shogunKick` | WA$TED mark and SHOGUN headline scaling on the kick |
| `waveKick` | waveform bar bouncing on the kick |
| `#strobe`, `#beat` | two stacked full-viewport brightness flashes |

Two separate faults produced the same symptom, and both are worth knowing so
neither gets reintroduced:

1. **The tempo was wrong.** Everything ran off a hardcoded `BPM = 155`. The track
   is **161.5 BPM** — measured by autocorrelating the spectral flux of the real
   audio and confirmed against the inter-onset histogram. Over its 4:06 that grid
   falls 26 beats, more than six bars, behind the music. It was out of time with
   the song for almost the entire play.
2. **`steps()` easing.** Two of the PRESS PLAY animations snapped between discrete
   values instead of gliding. That is what read specifically as a *twitch* rather
   than as drift.

Fixing the tempo was tried: the track was decoded to PCM and analysed offline
(low-band energy for the kick, broadband RMS for the arrangement) and the page was
driven off that instead. It worked and it was correctly in time. It was still cut,
because a still page reads better for this design. **The conclusion is not "get
the sync right" — it is that this page does not move with the music.**

### The one exception

The `SHOGUN` glitch and the VHS tracking tear fire at eleven fixed timestamps:

```js
var DROPS = [21.2, 57.6, 69.9, 83.6, 95.6, 107.6, 142.0, 155.6, 177.9, 191.6, 203.6];
```

These are the points where the arrangement actually steps up, measured off the
audio. They are one-shot effects a few hundred milliseconds long — eleven of them
across four minutes — so they read as events rather than as a rhythm. With the
track stopped, a randomised idle timer fires them instead, exactly as it always
has. `DROP_OFF` nudges them if one lands early or late.

They belong to **this** track. If the SoundCloud URL ever changes, either
re-measure them or delete the array and let the idle timer run all the time.

## Performance notes

- **No custom properties are written to `:root` per frame.** A root variable write
  invalidates style for the entire document; measured at 6.7ms/frame against a
  16.7ms budget. The only per-frame writes left in the file are pointer parallax on
  `.plate__cam` and scroll smear on `.shogun-shake`, both written straight to the
  element.
- **The waveform fill is one `clip-path` animation** running the length of the
  track, not per-bar class toggling across 64 elements.

Measured on a 4x-CPU-throttled 400px profile, scrolling the full page with the
track playing, three runs each: **57.8–58.8fps, zero long tasks, zero severe
stalls**, indistinguishable from the reactive build it replaced. Script time is
consistently lower (134–164ms vs 170–210ms) and the file is 17KB smaller.

## The logo

The mark is defined once as an SVG `<symbol id="wastedMark">` at the top of `<body>` and
referenced with `<use>` wherever it appears, so it costs its ~6.7KB once. It's filled with
`currentColor`, so the whole mark recolours from one line:

```css
.logo{ color:var(--gold); }   /* official khaki — change to var(--paper) for white knockout */
```

## Accessibility

`prefers-reduced-motion` stops the glitch, tear, parallax, scanline drift and Ken
Burns, and drops the grain to a flat 0.2 — verified: the hero transform stays
`none` and no inline styles are written. There is no full-page flashing anywhere
in the file. Motion is lightened on screens under 640px.
