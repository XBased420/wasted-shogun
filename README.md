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

## Beat-reactive motion

SoundCloud's iframe is cross-origin, so real frequency analysis isn't possible.
Instead every beat-driven effect is a **CSS animation exactly one bar long**, and
JS phase-locks them all to the track with a single negative `animation-delay`.
After that the compositor runs the motion and **JS does no per-frame work at all**.

Two constants at the top of the script tune it:

```js
var BPM       = 155;  // track tempo
var OFFSET_MS = 0;    // nudge +/- to line the kick up with the audio
```

Driven by the clock: hero push, CTA glow, the PRESS PLAY ring, the hazard strip,
the logo, a glitch every 4 bars and a VHS tracking tear every 8. The clock
re-locks only on a seek, never on a timer.

**The waveform is not beat-reactive.** It reads as playback position only — one
`clip-path` animation running the length of the track. It used to also bounce on
the kick; that was removed because the bounce read as random against the music.
Don't reattach it to the beat clock.

**No full-page brightness flashing.** Both flash layers were removed deliberately —
a constant fluorescent flicker (`#strobe`) and a beat-synced radial pulse
(`#beat`). Stacked, they read as incessant. The analog-broadcast texture that
gives the page its look is separate and untouched: film grain, scanlines, the CRT
roll band, the tracking tear and the headline glitch. Don't reintroduce a
full-viewport opacity flash.

## Performance notes

The motion system deliberately avoids two things that cost a lot on phones:

1. **No custom properties are written to `:root` per frame.** A root variable write
   invalidates style for the entire document; measured at 6.7ms/frame against a
   16.7ms budget. Values that must change per frame are written directly to the one
   element that uses them (pointer parallax on `.plate__cam`, scroll smear on
   `.shogun-shake`). Everything else is a CSS animation.
2. **The waveform fill is one `clip-path` animation** running the length of the
   track, not per-bar class toggling across 64 elements. It cannot step, and it
   carries no beat animation at all.

Measured on a 4x-CPU-throttled 400px profile, scrolling the full page with the
beat motion running: **59.5fps, zero long tasks, 544ms style recalc** (was 50.3fps
and 4,002ms).

## The logo

The mark is defined once as an SVG `<symbol id="wastedMark">` at the top of `<body>` and
referenced with `<use>` wherever it appears, so it costs its ~6.7KB once. It's filled with
`currentColor`, so the whole mark recolours from one line:

```css
.logo{ color:var(--gold); }   /* official khaki — change to var(--paper) for white knockout */
```

## Accessibility

`prefers-reduced-motion` stops the glitch, tear, beat animations, parallax and
Ken Burns while leaving the layout and palette identical. There is no full-page
flashing at all. Motion is lightened on screens under 640px.
