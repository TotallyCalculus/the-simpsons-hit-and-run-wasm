# The Simpsons: Hit & Run — WebAssembly build

A browser (Emscripten/WebAssembly) build of the SHAR engine, running via WebGL
with pthreads. This repository contains **only the compiled engine** — no game
data is included.

▶ **Play:** https://iscle.github.io/the-simpsons-hit-and-run-wasm/?assets=YOUR_ASSET_URL

## You must supply your own game data

The game streams ~1.8 GB of assets on demand and **none of it ships here** — you
have to point the build at a copy you host yourself, from game data you own.
Pass the base URL as a query parameter:

```
https://iscle.github.io/the-simpsons-hit-and-run-wasm/?assets=https://your-host.example/simpsons/
```

Without `?assets=`, the build looks for the data at `/assets/` on the same
origin (handy for local hosting) and will hang at the loading screen if it
isn't there.

### Requirements for the asset host

The engine fetches files with **HTTP Range** requests, and because the page runs
cross-origin isolated (needed for threads), an external asset host must also send
permissive CORS headers:

- `Accept-Ranges: bytes` / honor `Range` (return `206 Partial Content`)
- `Access-Control-Allow-Origin: *`
- `Access-Control-Allow-Headers: Range`
- `Access-Control-Expose-Headers: Content-Range, Content-Length, Accept-Ranges`

The data is served as the raw game files/folders (e.g. `.../simpsons/scripts/...`),
laid out exactly as in the original data directory.

## How the pieces fit

| File | Purpose |
|------|---------|
| `index.html` | Page shell: canvas, loading bar, fullscreen, saves, tab-close guard |
| `SRR2.js` / `SRR2.wasm` | The compiled engine |
| `coi-serviceworker.js` | Adds COOP/COEP headers so `SharedArrayBuffer` (threads) works on static hosts like GitHub Pages |

Saves persist in the browser (IndexedDB). Movies play via an in-wasm FFmpeg
decoder. Controls follow the retail PC layout (WASD to move/drive, Space jump,
Shift sprint, left-click action / enter–exit car, right-click attack, numpad
camera).

## Notes

- Static hosts can't set headers, so `coi-serviceworker.js` reloads the page
  once on first visit to gain cross-origin isolation.
- GitHub Pages can't host the assets themselves (1 GB site cap, 100 MB/file git
  limit), which is why they're external and user-supplied.
