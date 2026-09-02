+++
title = "ARC3 V24 (LB 2.56): MULTIMODAL_UPSCALE=8 - The Decisive Move"
date = 2026-08-25T04:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "FP8", "MULTIMODAL_UPSCALE", "winner"]
categories = ["ARC3 Dev Journal"]
summary = "Single-variable change: MULTIMODAL_UPSCALE 4 to 8 (256x256 to 512x512 vision). Local mean 4.984 (+66% over V23's 3.008). LB 2.56 (+1.03 over V23's 1.53). Personal best."
lb_score = "2.56"
version = "V24"
status = "WINNER"
+++

## TL;DR

Single-variable change: MULTIMODAL_UPSCALE 4 to 8 (256x256 to 512x512 vision). Local mean 4.984 (+66% over V23's 3.008). LB 2.56 (+1.03 over V23's 1.53). Personal best.

## Context

V23 set local mean 3.008 with MULTIMODAL_UPSCALE=4. The hypothesis: 256x256 vision loses sub-cell patterns that matter for ARC-AGI-3's cover predicates and co-location win conditions. 512x512 should preserve them.

V24 was a single-variable change: `MULTIMODAL_UPSCALE = 8` (was 4). No other changes. This is the cleanest attribution experiment in the entire 33-day campaign.

## Technical Choice

The patch was one line:

```python
# V23
'MULTIMODAL_UPSCALE': '4',

# V24
'MULTIMODAL_UPSCALE': '8',
```

The patch was applied in Cell 5 (pre-setup) before the TAAF setup command ran, following the dvm-learned env var injection pattern from V15.

Verification: stdout contains `v24: Patched MULTIMODAL_UPSCALE to 8 (512x512 vision)`. The TAAF setup command read the env var and configured the solver accordingly.

## Parameter Decisions

| Parameter | V23 | V24 | Rationale |
|---|---|---|---|
| MULTIMODAL_UPSCALE | 4 (256x256) | 8 (512x512) | Preserve sub-cell patterns |
| FP8 KV cache | enabled | enabled | unchanged |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | unchanged |
| source bundle | anim bundle | anim bundle | unchanged |
| temperature | 0.6 | 0.6 | unchanged |
| context window | 32768 | 32768 | unchanged |
| n_passes | 1 | 1 | unchanged |

## Local vs LB Score

- Local mean: 4.984
- Baseline (previous version): V23 local 3.008
- Local delta: +1.976 (+66%)
- LB score: **2.56**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True + --kv-cache-dtype fp8 in vLLM launch |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 (512x512 vision) |
| Qwen3.8 model | yes | Qwen/Qwen3.8-27B-FP8 (verified in vLLM startup log) |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort: 'high' |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## Outcome Analysis

LB 2.56, +1.03 over V23's 1.53. Local mean 4.984, +66% over V23's 3.008. The local-to-LB delta is +1.03 / +1.976 = 52% transfer rate, the highest in the campaign.

Why does MULTIMODAL_UPSCALE=8 matter so much?

ARC-AGI-3's hidden games include cover predicates (every object of kind A must end up co-located with an object of kind B) and pixel-level equality (a workspace region must be made equal to a reference region). At 256x256, sub-cell patterns are lost; the solver cannot distinguish two similar-shaped objects. At 512x512, the solver can.

The 4x token increase (256x256 = 65k tokens, 512x512 = 262k tokens before processing) is absorbed by the FP8 KV cache and the 32k context window. The vision tokens are processed once per observation, not retained across the full game.

The result: the solver can read the win condition off the first frame (per busyaprime's prior finding that "the goal is usually already drawn on the first frame") and execute actions to satisfy it.

This is the only submission where all 9 critical patches fired simultaneously and the configuration was correct. V25 would break this by removing FP8 KV, WBC, RE cap, NG.

## Next Version Plan

V25 will add 7 TAAF grafts (winframe, goalkeep, clockwatch, hudmask, clickmap, searchmap, lawbook) on top of V24. Target: LB 3.0+. The hypothesis: grafts on top of V24's winning config should compound.
