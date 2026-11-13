# Asuka Web Player

The standalone web build of **Asuka** — a retro vintage PC runner — compiled to
WebAssembly and running entirely in the browser. Two cores are built in: **FM
Towns** (Geartowns) and **NEC PC-98** (NP2kai). This deployment boots the FM
Towns demo; the PC-98 core is selected at runtime with `?machine=pc98` or a
`machine` field in the manifest.

**This deployment is turnkey.** Open the page and it boots by itself: the FM
Towns BIOS and *The New Zealand Story* (Ving, 1989) are bundled with the site, so
there is nothing to upload. A minimal loading bar tracks the ~7 MB asset
download, then the game starts running with the on-screen touch pad already
enabled.

The interface is deliberately minimal — a black letterboxed 4:3 canvas with a
translucent on-screen pad. There are no upload panels, file lists, or debug
cards: if the bundled assets ever fail to load, the loader simply shows an error
and a **Retry** button. A floating **🎮 ⇄ ⌨️** button swaps the on-screen pad for
a virtual PC-98 keyboard (standard rows plus `HELP`, `STOP`, `ROLL UP/DOWN` and
`F1`-`F5`); it appears on touch devices and whenever the PC-98 core is running.

This repository is a deploy-only mirror of the `web/dist` bundle produced by
`scripts/build_web.sh` in the main [Asuka](https://github.com/M5Devs/Asuka)
repository, plus the bundled BIOS and disc assets. It contains no source; it is
the static site served to players.

## What's here

| Path | Purpose |
| --- | --- |
| `index.html` | The player shell: minimal auto-boot loader, letterboxed canvas, on-screen touch pad. |
| `asuka_web.js` | Emscripten JavaScript glue that loads and drives the WASM module. |
| `asuka_web.wasm` | The emulation core and the Asuka runtime, compiled to WebAssembly. |
| `coi-serviceworker.min.js` | Adds the COOP/COEP headers in-browser so static hosts (e.g. GitHub Pages) get `SharedArrayBuffer`. |
| `_headers` | The COOP/COEP isolation headers (and asset caching) for Cloudflare Pages. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is (no Jekyll processing). |
| `assets/manifest.json` | Declares the bundled BIOS ROMs and disc tracks; the shell reads it to auto-boot. |
| `assets/bios/` | FM Towns firmware ROMs. |
| `assets/media/` | *The New Zealand Story* CD image (a single 3 MB ISO). |

## How auto-boot works

1. The shell fetches `assets/manifest.json`.
2. It downloads every listed BIOS ROM into `/bios` and every disc file into
   `/media` inside the Emscripten filesystem, showing progress.
3. It initialises the engine with `media = /media/<cue>` and starts it.

If `assets/manifest.json` is missing or the assets cannot be fetched, the loader
shows the error and a **Retry** button; there is no manual upload path.

### Bundled assets

Firmware (from the [`Abdess/retrobios`](https://github.com/Abdess/retrobios)
collection):

```
FMT_SYS.ROM  FMT_DOS.ROM  FMT_FNT.ROM  FMT_DIC.ROM  FMT_F20.ROM
MYTOWNS.ROM  MAR_EX0.ROM  MAR_EX1.ROM  MAR_EX2.ROM  MAR_EX3.ROM
```

Disc: *The New Zealand Story (1989)(Ving)(Jp-En)* — a single-track
`MODE1/2048` ISO, so it is a lone 3 MB file with no CUE sheet and no splitting.

### Manifest schema

```jsonc
{
  "title": "The New Zealand Story",
  "machine": "fmtowns",                                     // optional: "fmtowns" (default) or "pc98"
  "cue": "New Zealand Story, The (1989)(Ving)(Jp-En).iso",  // file in /media to boot
  "totalBytes": 7266336,                                    // progress-bar fallback
  "bios":  [{ "name": "FMT_SYS.ROM", "url": "bios/FMT_SYS.ROM", "size": 262144 }],
  "media": [{ "name": "New Zealand Story, The (1989)(Ving)(Jp-En).iso", "url": "media/New%20Zealand%20Story%2C%20The%20%281989%29%28Ving%29%28Jp-En%29.iso", "size": 3072000 }]
}
```

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
  after the worker activates, then auto-boots.

## Hosting notes

- Everything is same-origin and static — any static host works.
- Serve over `https://` (or `localhost`); service workers require a secure
  context, and `coi-serviceworker` needs one to enable isolation.
- Keep `.nojekyll` in place on GitHub Pages, or Jekyll will skip the WASM and
  header files.
- The bundled assets are cached for a year, so a repeat visit boots from cache.

## Controls

Keyboard: arrows = d-pad, Enter = Start, Space = A, Z = B, X/Y = extra fire
buttons, R = reset, F11 = fullscreen. With the PC-98 core, typing is sent to the
emulated PC-98 keyboard (its scan codes differ from PC/AT).

On touch devices a translucent pad is shown automatically: a d-pad bottom-left,
A/B buttons bottom-right, and START at the top centre. The floating **🎮 ⇄ ⌨️**
button switches to a compact virtual PC-98 keyboard. The pad buttons drive the
engine through `asuka_web_key`; the keyboard buttons use `asuka_web_scancode`,
which injects the raw PC-98 scan code so the utility keys work. No synthetic DOM
key events are used, and `touch-action: none` keeps the browser from stealing
presses.

The canvas keeps focus so physical key presses reach the SDL2 keyboard listener.
A connected gamepad is detected too.

## Legal

Asuka is released under the GNU General Public License v3.0 (see `LICENSE`).
The bundled FM Towns BIOS ROMs and *The New Zealand Story* disc image remain the
property of their respective rights holders and are included only so this demo
can boot without a manual upload. Remove them and use the manual flow if you do
not have the right to distribute them.
