# Textmill Size Optimizer — build spec

Handoff spec for a new thread. Everything needed to build without re-deriving context.

## What this is

A client-side PDF size optimizer in the Textmill universe: user drops a PDF, gets back a
smaller one that is still text-readable (text layer preserved, image bloat removed).
Compression is the product — this is NOT part of the split/export pipeline, which already
copies pages losslessly from the original.

**Differentiator:** every commercial "compress PDF" site uploads the file to a server.
This one honors Textmill's promise: "Everything stays on your machine — nothing is uploaded."

## Decisions already made (don't relitigate)

1. **Engine: Ghostscript compiled to WebAssembly.** It downsamples/recompresses images,
   subsets fonts, and rewrites streams while PRESERVING the text layer. Proven in-browser:
   - https://github.com/laurentmmeyer/ghostscript-pdf-compress.wasm (demo: 24 MB scan → 3.9 MB in ~4 s)
   - https://github.com/krmanik/local-pdf-tools (compress/merge/split, all client-side)
2. **Rejected: rasterize-with-pdf.js + rebuild-with-pdf-lib.** Destroys the text layer,
   ~1 s/page on scanned books, and often loses to Ghostscript on size. Do not build this.
3. **Standalone page, not a feature of index.html.** The WASM bundle is ~10–20 MB;
   it must never sit in index.html's critical path. Ship as `optimizer.html` beside it,
   loading gs-wasm lazily (ideally only after a file is dropped). Cross-link the two pages
   later; don't couple them.
4. **Never output a larger file.** JBIG2/bilevel scans are often already near-optimal
   (fixture: 631-page Hegel scan is ~55 KB/page) and can GROW under recompression. Always
   show original size vs. result size; if result ≥ original, say so and offer no download
   (or offer it explicitly labeled). No magic promises.

## UX

- Same skeleton as index.html's start screen: wordmark, drop zone ("Drop a PDF here or
  click to choose"), privacy lock line. One screen, no navigation.
- After drop: pick a quality preset, then one primary action. Presets map to Ghostscript
  `-dPDFSETTINGS`: **screen** (72 dpi, smallest), **ebook** (150 dpi, recommended default),
  **printer** (300 dpi, gentle). Label them by outcome ("smallest / balanced / high quality"),
  not by Ghostscript jargon.
- Progress: Ghostscript runs in a Web Worker; surface its page-by-page console output as a
  simple status line (reuse textmill's status-bar pattern — errors visible, never silent).
- Result card: original size → new size (± %), then Download. Optionally a before/after
  page thumbnail via pdf.js if cheap; skip if it bloats the page.
- Errors (encrypted PDFs, malformed files, WASM load failure offline) show in the status
  line with plain-language explanations.

## Design language (match index.html)

- Dark theme, CSS variables: `--bg #121317`, `--panel #1a1c21`, `--accent #5aa2e0`,
  text `#e9e7e2`, dim `#8f939c`; hairline borders `rgba(255,255,255,.07)`.
- Newsreader (Google Fonts) for the wordmark — "Text**mill**" style with accent-italic
  span; system sans for UI at 13–14 px.
- Single self-contained HTML file, no build step, CDN dependencies only. Version string
  "v0.9.2" pattern with a console.log on load.

## Technical notes

- Start from laurentmmeyer's approach: Emscripten-built gs, virtual FS (write input to
  MEMFS, run `gs -sDEVICE=pdfwrite -dPDFSETTINGS=/ebook -o out.pdf in.pdf`, read output).
  Check whether a maintained npm/CDN build exists before vendoring; the wasm artifact must
  be hosted somewhere reliable (cdnjs won't have it — likely GitHub Pages alongside the app,
  or unpkg/jsdelivr from an npm package).
- Web Worker mandatory — gs blocks its thread for seconds-to-minutes.
- Memory: input + output + gs heap live in WASM memory simultaneously; test with 100 MB+
  inputs and fail gracefully (catch abort, suggest the file is too large).
- **License: Ghostscript is AGPL.** Distributing the app means making its source available —
  fine for a public single-file HTML app (the source IS the app), but confirm this sits
  right before shipping, and credit Ghostscript in the help/about text.

## Validation

Test with:
1. A bloated born-digital PDF (e.g. slide deck or report with oversized images) — expect
   large savings (often 5–20×), text still selectable/searchable in the output.
2. The Hegel fixture (631-page JBIG2 scan, 35 MB, `hegel-aesthetics.pdf`) — expect little
   or no savings at ebook preset; verify the "result not smaller" path behaves honestly.
   At screen preset verify output opens correctly in Preview/Chrome and pages are legible.
3. An encrypted or corrupt PDF — expect a clear status-line error, no silent failure.
4. Offline load — expect a clear "libraries failed to load" message (index.html has
   this pattern; copy it).

## Out of scope (v1)

- OCR / adding text layers to scans (tesseract.js — separate tool, separate decision).
- Batch processing, merge/split (splitting is index.html's job).
- Integration into index.html's UI beyond a link.
