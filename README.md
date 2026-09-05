# ARC3 Dev Journal

Kaggle ARC Prize 2026 (ARC-AGI-3) — 33 days of AI agent development, from LB 0.87 to LB 2.56.

Live site: https://ocean240812.github.io/arc3-dev-agi-journal/

## What This Is

A development journal documenting 21 leaderboard submissions between 2026-07-31 and 2026-09-01. Each post covers one submission: technical choice, parameter decision, local vs LB score, patch verification, and root-cause analysis.

## Headline Result

V24 reached LB 2.56 on 2026-08-25 by upgrading `MULTIMODAL_UPSCALE` from 4 to 8 (256x256 to 512x512 vision rendering). Every subsequent module addition (V25 grafts, V33/V34 NOOA, V35 modules) regressed below V24.

## Tech Stack

- Model: Qwen3.8-27B-FP8
- Serving: vLLM 0.19.0 + FP8 KV cache
- Framework: TAAF (Tufa Labs duck harness) — forked
- Vision: MULTIMODAL_UPSCALE=8 (512x512 grid rendering)
- Pipeline: Kaggle code competition rerun
- Instrumentation: level probe graft (Tufa-style per-game JSONL)

## Author

Vincent — ocean240812@gmail.com

## Build

This site is built with Hugo + PaperMod theme. To run locally:

```bash
git clone --recursive https://github.com/ocean240812/arc3-dev-agi-journal.git
cd arc3-dev-agi-journal
hugo server -D
```

Deployment is automated via GitHub Actions on push to `main`.
