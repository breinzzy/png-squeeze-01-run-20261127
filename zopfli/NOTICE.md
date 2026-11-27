# Vendored zopfli (WebAssembly)

This directory vendors the `@gfx/zopfli` package (version 1.0.15), a
JavaScript/WebAssembly binding of Google's zopfli compression library.

- Upstream: https://github.com/gfx/universal-zopfli-js
- zopfli itself: https://github.com/google/zopfli
- License: Apache License 2.0 (see the LICENSE.md files in the original
  package; the zopfli C library is also Apache 2.0, Google).

Files:
- `index.js` — the `@gfx/zopfli` wrapper (dist/index.js).
- `libzopfli.js` — the Emscripten-generated glue (dist/libzopfli.js).
- `libzopfli-wasm.json` — the compiled zopfli WebAssembly module, base64
  encoded (dist/libzopfli-wasm.json).
- `node_modules/base64-js/` — the `base64-js` dependency used by the glue
  (MIT license, see its LICENSE file).

These files are used as-is, unmodified, to compress the filtered image data
with the real zopfli algorithm (1–2 iterations: `numiterations: 2` on most
streams, `numiterations: 1` on the two slowest). The rest of
this entry (PNG decode, form/filter screening, the custom optimal-parse
DEFLATE, and all orchestration) is original code written for this bounty.
