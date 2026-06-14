# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"구름빵" (Cloud Bread) — a Korean children's AR storybook. Two standalone HTML files, no build step, no package manager.

## Running the Project

Serve from a local HTTP server (required for Geolocation API and HTTPS context for AR):

```bash
# Python
python -m http.server 8080

# Node.js (npx)
npx serve .
```

Open `book.html` for the interactive book, `ar.html` for the GPS-based AR experience. The AR scene requires a mobile device with GPS and a browser that supports HTTPS (use a tunnel like ngrok for local testing on device).

## Architecture

### book.html — Canvas Flip-Book Reader

All logic is inline JavaScript (~390 lines). The app runs a `requestAnimationFrame` loop and transitions through a state machine:

- **black** → loading/black screen
- **fadein** → cover image fades in with candle audio
- **cover** → static cover display
- **opening** → animated flip to first spread
- **book** → interactive page flipping (6 spreads, 12 pages)

Page-flip animation uses 60 canvas slices drawn per frame with perspective foreshortening. On the final page, a button navigates to `ar.html` and sets `sessionStorage.item("caught")` state.

### ar.html — GPS-Based AR Experience

Inline JavaScript (~250 lines) layered on top of A-Frame + AR.js (both loaded from CDN).

Target coordinates are hardcoded at the top of the script:
```js
const TARGET_LAT = 37.655934;
const TARGET_LNG = 127.013159;  // near Seoul, South Korea
```

Distance thresholds:
- **> 80m** — "too far" overlay, cloud entity hidden
- **≤ 80m** — cloud visible
- **≤ 15m** — "catch cloud" button appears

The cloud is built from multiple overlapping `<a-sphere>` entities with a CSS `@keyframes` pulse animation. Catching the cloud plays a shrink animation, sets `sessionStorage`, and redirects to `book.html`.

### GPS distance calculation

Uses the Haversine formula implemented inline in `ar.html`. No external geo library.

## External Dependencies (CDN only)

| Library | Source | Used in |
|---|---|---|
| A-Frame 1.4.0 | `aframe.io/releases/1.4.0/aframe.min.js` | ar.html |
| AR.js NFT | `raw.githack.com/AR-js-org/AR.js/.../aframe-ar-nft.js` | ar.html |
| AR.js Location | `raw.githack.com/AR-js-org/AR.js/.../aframe-ar.js` | ar.html |

No npm dependencies. No TypeScript. No bundler.

## Key Conventions

- All JavaScript is inline in the HTML `<script>` tags — no separate `.js` files.
- Audio files (`candle.mp3`, `rain.mp3`) are faded in/out via a custom `fadeAudio()` helper in `book.html`.
- `sessionStorage.item("caught")` is the only cross-page state; `book.html` reads it to decide whether to show a "cloud caught" congratulations frame.
- Image assets follow the naming pattern `page{N}_{left|right}.png` for book spreads.
