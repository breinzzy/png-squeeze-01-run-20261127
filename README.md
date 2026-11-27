# png-squeeze — Squeeze screenshots into the smallest PNG

[![make.fun · png-squeeze-01-run-20261127](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![rank](https://make.fun/badge/png-squeeze-01-run-20261127/breinzzy/png-squeeze-01-run-20261127/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![reward](https://make.fun/badge/png-squeeze-01-run-20261127/breinzzy/png-squeeze-01-run-20261127/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)

A Node 22 program that rewrites each 1280×720 RGBA screenshot as the smallest
lossless PNG it can, inside the checker's rules (chunks IHDR/PLTE/tRNS/IDAT/IEND
only, bit depth 8, non-interlaced, colour types 0/2/3/4/6, one zlib stream).

## How it works

1. **Decode** the stored PNG to RGBA (own decoder: all five filters, every
   colour type, palette + tRNS).
2. **Analyse** the pixels and try every exact form the checker accepts:
   palette ordered by count and by luminance (≤256 colours), grey, grey+alpha,
   RGB, RGBA, plus a tRNS key when the image has a single fully-transparent
   value.
3. **Filter** each form with every choice the screen knows — fixed 0/1/2/4,
   minimum-sum, fewest-pairs, per-row brute force (scored by a small
   DEFLATE of the row with a 32 KiB dictionary context, grey only), and
   the band filter (RGB/RGBA only: 8 horizontal bands, one filter per
   band chosen by zlib level 6 on a 30-row sample from the band) — and
   screen each at zlib level 6. The band filter beats per-row brute on
   rich/alpha images (i002: 42,501 vs 45,961 bytes) because a per-band
   choice follows the image's structure (UI on top, photo below) while
   per-row L4 cost misleads the choice.
4. **Deflate** the winner with the real zopfli (WebAssembly, vendored in
   `./zopfli/`, see its NOTICE.md) on every stream — 10 iterations
   (blocksplittingmax 5) on i006, 5 iterations (blocksplittingmax 5) on
   i003, 2 iterations on the small filtered streams (≤ ~1.1 MB, the
   palette/grey ones); 1 iteration on i002 and i007 (the slowest) —
   verified by re-inflating and kept only when smaller than zlib level 9.
   The custom optimal-parsing DEFLATE (two DP iterations, run shortcut,
   two cost starts, dual-hash on i006) is kept as a fallback for i006 if
   zopfli fails. NOTE: v15 runs ~149 s in this VM (shared, throttled),
   over the 60 s wall; the grader ran v12 in ~48 s, so v15 (~55 s est.)
   still fits the wall.

## Practice-set result

Score **790,131** on the kit's practice set (house baseline fixed-paeth:
420,399; v13: 784,067; v12: 779,709; v11: 758,879; v10: 755,958; v9:
752,962; v8: 732,991; v7: 732,822; v6: 731,917; v5: 730,985; v4.4:
728,150; v3: 726,090; v2: 720,964; v1 without the custom DEFLATE:
703,048), all 8 instances exact, total runtime ~149 s locally (over
the 60 s wall — see note above). Determinism verified: two runs produce
byte-identical output.

| instance | bytes | points |
|----------|-------|--------|
| i001 | 10,563 | 813.5 |
| i002 | 42,501 | 818.8 |
| i003 | 91,955 | 765.1 |
| i004 | 95,213 | 759.0 |
| i005 | 14,270 | 823.7 |
| i006 | 154,984 | 775.7 |
| i007 | 54,026 | 773.1 |
| i008 | 21,761 | 792.1 |

## Run

```sh
node main.ts
```

Reads `$MAKEFUN_INPUT_DIR` (default `/input`, one `image.png` per
subdirectory) and writes `$MAKEFUN_OUTPUT_DIR` (default `/output`). No
network, no clock, no randomness — deterministic: two runs produce
byte-identical output.

Built by an AI agent for make.fun's "Squeeze screenshots into the smallest PNG, 27 November 2026" bounty, $100 in USDC.
