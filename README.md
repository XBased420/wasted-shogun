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
- Motion driven by an offline analysis of the actual track, not a tempo guess
- Music autoplays on load where the browser allows it; where it doesn't, every
  interaction retries until playback starts. The viewer can always pause from the
  PRESS PLAY control, and a manual pause is never overridden

## How the page reacts to the music

SoundCloud's iframe is cross-origin, so the page cannot run a live analyser on it.
So the track was analysed **once, offline**: fetched, decoded to PCM at 22.05kHz,
short-time Fourier transform at 1024/256, then two envelopes pulled out of it —
low-band energy (20–150Hz, the kick) and broadband RMS (the arrangement). Both are
quantised to one byte per sample and embedded in the page as base64:

```js
var TRACK = {
  durMs:    245967,
  offsetMs: 0,      // nudge + / - if the reaction sits early or late
  kickHz:   30,     // 7,379 samples of kick
  enHz:     7.5,    // 1,845 samples of loudness
  drops:    [21.2, 57.6, 69.9, ...],   // the 11 measured energy jumps
  kick:   '...',
  energy: '...'
};
```

A single `requestAnimationFrame` loop reads the widget's playback position,
interpolates both envelopes, and writes to exactly three elements. There is **no
BPM constant anywhere in this file**, on purpose — see below.

| What moves | Driven by |
|---|---|
| Hero plate scale (`.plate__beat`) | kick envelope, gated by loudness² |
| Hero vignette (`.plate__grade` opacity) | broadband loudness |
| Film grain density (`#grain` opacity) | broadband loudness |
| `SHOGUN` glitch + VHS tear | the 11 measured drops |

A hard techno bassline never returns to silence, so the raw low-band level sits on
a high floor and the kick is only the peak above it. The loop reads the local floor
and ceiling out of the data either side of the playhead (±1.2s) and takes the kick
as the distance between them. Without that the hero just sits permanently zoomed in
and jitters — which is the exact failure this replaced.

### Why the old version felt random

The page used to run every "beat-reactive" effect off a fixed `BPM = 155` constant.
**The track is 161.5 BPM.** Over its 4:06 that grid falls 26 beats — more than six
bars — behind the music. It was out of time with the song for almost the entire
play. That is what read as janky, and it is the same root cause behind the flash
layers, the waveform bounce, the type scaling and the PRESS PLAY twitch, all of
which were removed in earlier passes for looking arbitrary.

Nothing in the page runs on a tempo now. It follows the record.

### ⚠️ The envelopes belong to ONE track

`TRACK.kick` and `TRACK.energy` are this specific master of *Can You Feel My Heart
(SHOGUN Remix)*. **Change the SoundCloud URL and the reaction is wrong** — the page
will keep moving, but to the wrong song. To retarget it, regenerate the data:

1. Open the new track's SoundCloud page and pull its progressive stream URL
2. `decodeAudioData` it to mono PCM
3. STFT at 1024/256; per frame take √Σ|X(k)|² for bins 1–7 (low band) and the
   broadband RMS
4. Peak-hold the low band to 30Hz and mean the RMS to 7.5Hz, normalise each against
   its 97th percentile, gamma 0.75 / 0.85, quantise to `Uint8`, base64
5. Drops = where a ±1.5s smoothed RMS steps up by more than 38/255, minimum 8s apart

Set `durMs` to the real duration and re-check `offsetMs`.

### Tuning

`TRACK.offsetMs` is the only knob you should normally need. Positive values push the
reaction later. The analysis came from the MP3 transcoding while the widget may
stream HLS, so a few tens of milliseconds of encoder delay is possible.

## Deliberately still

**The PRESS PLAY button does not animate at all.** It carried three: an idle
heartbeat (`kick`), a beat ring while playing (`pressKick`), and a light sweep
across the label (`wipe`). All removed. Two of them used `steps()` easing, which
made them snap rather than glide — that is what read as a twitch. The button
still changes state (red border and text while playing, play/pause icon, hover
flip); it just holds still.

**No type moves with the music.** The WA$TED mark (header and footer) and the
`SHOGUN` headline used to scale on the kick — `logoKick` and `shogunKick`. Both
were removed. Keep the type still; put motion on surfaces and glows instead.

**The waveform is not reactive.** It reads as playback position only — one
`clip-path` animation running the length of the track.

**No full-page brightness flashing.** Both flash layers were removed deliberately —
a constant fluorescent flicker (`#strobe`) and a beat-synced radial pulse
(`#beat`). Stacked, they read as incessant. The analog-broadcast texture that
gives the page its look is separate and untouched: film grain, scanlines, the CRT
roll band, the tracking tear and the headline glitch. Don't reintroduce a
full-viewport opacity flash.

**The CTA and the hazard strip no longer pulse on the beat.** `ctaKick` and
`stripPulse` are gone. The CTA keeps its own slow 2.4s breathe, which never claimed
to be musical.

## Performance notes

The motion system deliberately avoids two things that cost a lot on phones:

1. **No custom properties are written to `:root` per frame.** A root variable write
   invalidates style for the entire document; measured at 6.7ms/frame against a
   16.7ms budget. The reaction loop writes straight to the three elements that use
   it (`.plate__beat`, `.plate__grade`, `#grain`), plus pointer parallax on
   `.plate__cam` and scroll smear on `.shogun-shake`.
2. **The waveform fill is one `clip-path` animation** running the length of the
   track, not per-bar class toggling across 64 elements.

The rAF loop replaced three infinite CSS animations, and is cheaper than what it
replaced. Measured on a 4x-CPU-throttled 400px profile, scrolling the full page
with the track playing:

| | old (fixed BPM) | new (real analysis) |
|---|---|---|
| fps | 57.5 | **58.1** |
| worst frame | 107ms | **34ms** |
| janky frames | 8 | **2** |
| style recalc | 790ms | **603ms** |
| script | 247ms | **202ms** |

## The logo

The mark is defined once as an SVG `<symbol id="wastedMark">` at the top of `<body>` and
referenced with `<use>` wherever it appears, so it costs its ~6.7KB once. It's filled with
`currentColor`, so the whole mark recolours from one line:

```css
.logo{ color:var(--gold); }   /* official khaki — change to var(--paper) for white knockout */
```

## Accessibility

`prefers-reduced-motion` stops the glitch, tear, parallax and Ken Burns, and the
reaction loop never starts — verified: no inline styles are written at all, the
hero transform stays `none` and the grain holds at a fixed opacity. There is no
full-page flashing. Motion is lightened on screens under 640px.
