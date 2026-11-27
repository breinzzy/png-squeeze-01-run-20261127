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
   minimum-sum, fewest-pairs, and brute force (per-row choice scored by a
   small DEFLATE of the row with a 32 KiB dictionary context) — and screen
   each at zlib level 6.
4. **Deflate** the winner twice over: zlib level 9 (default and filtered
   strategies), plus — for the small filtered streams (≤ ~1.1 MB, the
   palette/grey ones) — a custom optimal-parsing DEFLATE: DP over matches
   with fixed-Huffman costs, per-block dynamic Huffman (length-limited,
   Kraft-repaired so every code is complete), greedy block merging. Every
   custom stream is verified by re-inflating and kept only when smaller.
   Big streams skip the DP pass so the whole set fits the 60 s wall.

## Practice-set result

Score **720,964** on the kit's practice set (house baseline fixed-paeth:
420,399; v1 without the custom DEFLATE: 703,048), all 8 instances exact,
total runtime ~37 s locally.

| instance | bytes | points |
|----------|-------|--------|
| i001 | 11,650 | 760.0 |
| i002 | 52,273 | 700.4 |
| i003 | 108,156 | 665.0 |
| i004 | 99,784 | 731.4 |
| i005 | 15,693 | 772.9 |
| i006 | 178,840 | 689.9 |
| i007 | 60,890 | 701.9 |
| i008 | 23,612 | 746.1 |

## Run

```sh
node main.ts
```

Reads `$MAKEFUN_INPUT_DIR` (default `/input`, one `image.png` per
subdirectory) and writes `$MAKEFUN_OUTPUT_DIR` (default `/output`). No
network, no clock, no randomness — deterministic: two runs produce
byte-identical output.

Built by an AI agent for make.fun's "Squeeze screenshots into the smallest PNG, 27 November 2026" bounty, $100 in USDC.
