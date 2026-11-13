# Asuka Web Player

The standalone web build of **Asuka** — a retro vintage PC runner — compiled to
WebAssembly and running entirely in the browser. The first supported core is the
**FM Towns** (Geartowns) emulation plugin.

**This deployment is turnkey.** Open the page and it boots by itself: the FM
Towns BIOS and *After Burner* (Sega, 1987) are bundled with the site, so there is
nothing to upload. A loading bar tracks the ~225 MB asset download, then the game
starts running with the on-screen touch pad already enabled.

This repository is a deploy-only mirror of the `web/dist` bundle produced by
`scripts/build_web.sh` in the main [Asuka](https://github.com/M5Devs/Asuka)
repository, plus the bundled BIOS and disc assets. It contains no source; it is
the static site served to players.

## What's here

| Path | Purpose |
| --- | --- |
| `index.html` | The player shell: auto-boot loader, drag & drop fallback, disc selector, on-screen touch pad, fullscreen. |
| `asuka_web.js` | Emscripten JavaScript glue that loads and drives the WASM module. |
| `asuka_web.wasm` | The emulation core and the Asuka runtime, compiled to WebAssembly. |
| `coi-serviceworker.min.js` | Adds the COOP/COEP headers in-browser so static hosts (e.g. GitHub Pages) get `SharedArrayBuffer`. |
| `_headers` | The COOP/COEP isolation headers (and asset caching) for Cloudflare Pages. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is (no Jekyll processing). |
| `assets/manifest.json` | Declares the bundled BIOS ROMs and disc tracks; the shell reads it to auto-boot. |
| `assets/bios/` | FM Towns firmware ROMs. |
| `assets/media/` | *After Burner* CD image, split into one BIN per track plus its CUE sheet. |

## How auto-boot works

1. The shell fetches `assets/manifest.json`.
2. It downloads every listed BIOS ROM into `/bios` and every disc file into
   `/media` inside the Emscripten filesystem, showing progress.
3. It initialises the engine with `media = /media/<cue>` and starts it.

If `assets/manifest.json` is missing or the assets cannot be fetched, the shell
falls back to the manual flow: drag the BIOS ROMs and a disc image onto the page
(or use the pickers), pick the disc, and press **Boot**. Uploaded files are kept
in the browser's IndexedDB and never leave the machine.

### Bundled assets

Firmware (from the [`Abdess/retrobios`](https://github.com/Abdess/retrobios)
collection):

```
FMT_SYS.ROM  FMT_DOS.ROM  FMT_FNT.ROM  FMT_DIC.ROM  FMT_F20.ROM
MYTOWNS.ROM  MAR_EX0.ROM  MAR_EX1.ROM  MAR_EX2.ROM  MAR_EX3.ROM
```

Disc: *Afterburner (1987)(Sega)(Jp-En)* — track 1 is `MODE1/2352` data; tracks
2–8 are audio. The original single 230 MB BIN exceeds GitHub's 100 MB file limit,
so it is stored as one BIN per track and described by a multi-`FILE` CUE sheet.
The track LBAs (and therefore the disc TOC) are byte-for-byte identical to the
original image.

### Manifest schema

```jsonc
{
  "title": "After Burner",
  "cue": "Afterburner (1987)(Sega)(Jp-En).cue",   // file in /media to boot
  "totalBytes": 234749961,                         // progress-bar fallback
  "bios":  [{ "name": "FMT_SYS.ROM", "url": "bios/FMT_SYS.ROM", "size": 262144 }],
  "media": [{ "name": "…(Track 01).bin", "url": "media/…", "size": 7126560, "label": "(Track 01)" }]
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

Keyboard: arrows = d-pad, Enter = Start, Space/Z/X/C = fire, R = reset,
F11 = fullscreen. On touch devices the on-screen gamepad is shown automatically.
A connected gamepad is detected too.

## Legal

Asuka is released under the GNU General Public License v3.0 (see `LICENSE`).
The bundled FM Towns BIOS ROMs and *After Burner* disc image remain the property
of their respective rights holders and are included only so this demo can boot
without a manual upload. Remove them and use the manual flow if you do not have
the right to distribute them.
