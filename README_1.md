# WA$TED x TRINITY: SHOGUN — Event Landing Page

Promo landing page for **SHOGUN** at The Circle OC, Huntington Beach — Saturday, September 19, 2026.
Presented by WA$TED x TRINITY.

**Live:** https://xbased420.github.io/wasted-shogun/

Single self-contained `index.html`. Inline CSS and JS only; the only external
dependencies are Google Fonts and SoundCloud's official embed.

## What's in it

- Fully procedural hero — SVG turbulence + silhouette, no raster assets
- Animated film grain, drifting scanlines, CRT roll and a low-amplitude flicker
- Glitching `SHOGUN` headline with RGB channel split and slice displacement
- Kinetic slam-in section headers, pointer parallax, scroll-velocity shake
- Auto-fitting display type — measured against its container, so it can never clip
- Music starts on load where the browser allows it, otherwise on first interaction;
  the viewer can always pause from the PRESS PLAY control

## Beat-reactive motion

The page runs a beat clock off the SoundCloud widget's reported playback position.
SoundCloud's iframe is cross-origin, so real frequency analysis isn't possible —
this is a timed sync at a fixed BPM, but it lands on the kick.

Two constants at the top of the script tune it:

```js
var BPM       = 155;  // track tempo
var OFFSET_MS = 0;    // nudge +/- to line the kick up with the audio
```

Driven by the clock: hero push and brightness, the beat flash overlay, CTA glow,
the PRESS PLAY ring, waveform bars, the hazard-strip tint, a glitch every 4 bars
and a VHS tracking tear every 8.

## Accessibility

`prefers-reduced-motion` stops the strobe, glitch, tear, parallax and Ken Burns
while leaving the layout and palette identical. Flicker is low-amplitude and
capped under 3 Hz. Motion is lightened on screens under 640px.
