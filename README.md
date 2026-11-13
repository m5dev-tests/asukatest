# Asuka Web Player

The standalone web build of **Asuka** — a retro vintage PC runner — compiled to
WebAssembly and running entirely in the browser. The first supported core is the
**FM Towns** (Geartowns) emulation plugin.

This repository is a deploy-only mirror of the `web/dist` bundle produced by
`scripts/build_web.sh` in the main [Asuka](https://github.com/M5Devs/Asuka)
repository. It contains no source; it is the static site served to players.

## What's here

| File | Purpose |
| --- | --- |
| `index.html` | The player shell: drag & drop, disc selector, on-screen touch pad, fullscreen. |
| `asuka_web.js` | Emscripten JavaScript glue that loads and drives the WASM module. |
| `asuka_web.wasm` | The emulation core and the Asuka runtime, compiled to WebAssembly. |
| `coi-serviceworker.min.js` | Adds the COOP/COEP headers in-browser so static hosts (e.g. GitHub Pages) get `SharedArrayBuffer`. |
| `_headers` | The same COOP/COEP isolation headers for Cloudflare Pages. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is (no Jekyll processing). |

## Running it

The emulator's CD reader uses a WebAssembly thread, which needs
`SharedArrayBuffer`. Browsers only expose that to **cross-origin isolated**
documents, so the page must be served with:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

- **Cloudflare Pages** applies these from `_headers` automatically.
- **GitHub Pages** cannot send custom headers, so `coi-serviceworker.min.js`
  installs a service worker that adds them on the client. The page reloads once
  after the worker activates.

Then open the page, drag in the FM Towns BIOS ROMs and a disc image (or use the
pickers), choose the disc, and press **Boot**. Uploaded files are stored in the
browser's IndexedDB and never leave the machine.

## Hosting notes

- Everything is same-origin and static — any static host works.
- Serve over `https://` (or `localhost`); service workers require a secure
  context, and `coi-serviceworker` needs one to enable isolation.
- Keep `.nojekyll` in place on GitHub Pages, or Jekyll will skip the WASM and
  header files.

## License

Asuka is released under the GNU General Public License v3.0. See `LICENSE`.
