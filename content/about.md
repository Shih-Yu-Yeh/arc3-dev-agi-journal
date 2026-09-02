+++
title = "About"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Vincent — ocean240812

Software engineer focused on AI agent development, large language model inference, and code competition pipelines.

## Contact

- Email: ocean240812@gmail.com
- Name: Vincent
- Kaggle: [ocean240812](https://www.kaggle.com/ocean240812) (Rank 59 / 2708 teams on ARC-AGI-3)
- GitHub: [@ocean240812](https://github.com/ocean240812)

## Why ARC Prize 2026 (ARC-AGI-3)

ARC-AGI-3 is the first interactive reasoning benchmark for AI agents: 110 hidden games, single pass, 9-hour wall clock, RHAE scoring. I entered to validate three hypotheses:

1. Qwen3.8-27B-FP8 with vision upsampling can score above 2.0 on hidden games.
2. The Tufa Labs duck harness is a sound base; targeted patches outperform custom modules.
3. Local mean score is uncorrelated with leaderboard score on hidden games.

Result: V24 reached LB 2.56 on 2026-08-25 by upgrading `MULTIMODAL_UPSCALE` from 4 to 8. Every subsequent module addition (V25 grafts, V33/V34 NOOA, V35 modules) regressed below V24.

## Tech Stack

| Layer | Choice | Reason |
|-------|--------|--------|
| Model | Qwen3.8-27B-FP8 | Upgraded from Qwen3.6-27B-FP8 at V17; flash-attention compatible, FP8 weights |
| Serving | vLLM 0.19.0 + FP8 KV cache + prefix caching | Required for 65k context window on a single RTX Pro 6000 |
| Framework | TAAF (Tufa Labs duck harness) — forked | 5-step cycle (observe, reason, act, score, update); established baseline |
| Vision | MULTIMODAL_UPSCALE=8 (512x512 grid rendering) | The deciding factor: V23 (upscale=4) = 1.53, V24 (upscale=8) = 2.56 |
| Pipeline | Kaggle code competition rerun | submission.parquet triggers hidden games via internal gateway |
| Instrumentation | level probe graft (Tufa-style per-game JSONL) | Only way to learn hidden game behavior post-hoc |
| Validation | 14-marker patch verification (kaggle kernel_log.json API) | Catches "claimed patch that never fired" (V14, V31) |

## Five Largest Lessons

1. **Local mean is not LB score.** V35.2 local mean 6.44, LB 1.59. Modules only helped on the 6 familiar public games, not the 85 hidden ones.
2. **Every module addition regressed LB.** V25 grafts, V33/V34 NOOA, V35 6-modules. Confirms Tufa Labs' empirical claim that hand-crafted modules degrade model performance.
3. **V31 "7 modules installed" was a lie.** Wiring was broken (`AttributeError: _system_prompt`). 0 events mentioned NOOA across 1348 total events. The "non-fatal" exception silently meant modules never executed.
4. **V35.1 UnboundLocalError x 2443.** A closure scope bug (`_v35_last_level` missing `nonlocal` declaration) crashed step_env on every call. Harder to catch than logic bugs because the kernel still completed.
5. **`MULTIMODAL_UPSCALE=8` was the decisive move.** V23 to V24: one change, +1.03 LB. Vision resolution matters more than reasoning effort for ARC-AGI-3.

## Development Constitution v2.0 (16 rules)

Documented in [Lessons Learned](/lessons-learned/). Highlights:

- Local dry-run is a mandatory pre-step before every push.
- One variable per version. V6 broke this rule and regressed -0.31.
- Never write bare `except Exception: pass`. V31 paid the price.
- Verify event contents, not just event counts. V31 had 1348 events but 0 useful ones.
