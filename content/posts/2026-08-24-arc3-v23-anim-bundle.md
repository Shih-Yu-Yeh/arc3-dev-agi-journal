+++
title = "ARC3 V23 (LB 1.53): Anim Bundle + Qwen3.8 Wheelhouse Fix"
date = 2026-08-24T01:43:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "anim-bundle"]
categories = ["ARC3 Dev Journal"]
summary = "Switched to jakobbrggen anim bundle. Fixed wheelhouse owner. 1-pass. LB 1.53, recovered from V21's 0.00. New personal best above V17's 1.43."
lb_score = "1.53"
version = "V23"
status = "IMPROVEMENT"
+++

## TL;DR

Switched to jakobbrggen anim bundle. Fixed wheelhouse owner. 1-pass. LB 1.53, recovered from V21's 0.00. New personal best above V17's 1.43.

## Context

V21's 2-pass experiment produced 0.00 twice. The 2-pass code was abandoned. V23 reverted to 1-pass.

Two changes: (1) switched to `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle, which has animation-awareness built in; (2) fixed the wheelhouse owner from `jeroencottaar` to `driessmit1`, which had been causing silent dependency resolution failures.

## Technical Choice

Anim bundle: the `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle includes:
- Animation-awareness (waits for game board to settle before reading)
- Hard noop guard (prevents infinite loops)
- Qwen3.8 model patch (auto-resolves to `jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`)

Wheelhouse fix: changed `WHEELHOUSE_OWNER = 'jeroencottaar'` to `WHEELHOUSE_OWNER = 'driessmit1'`. The `jeroencottaar` wheelhouse had been deprecated; `driessmit1/arc3-vllm-h100-wheelhouse-v3` is the current canonical wheelhouse.

No MULTIMODAL_UPSCALE patch in V23. The anim bundle defaults to MULTIMODAL_UPSCALE=4.

## Parameter Decisions

| Parameter | V21 | V23 | Rationale |
|---|---|---|---|
| source bundle | V19 bundle | jakobbrggen anim bundle | Animation-awareness |
| wheelhouse owner | jeroencottaar | driessmit1 | Fix deprecation |
| n_passes | 2 | 1 | Revert from V21 |
| MULTIMODAL_UPSCALE | 8 | 4 (stock) | Revert to stock |
| FP8 KV cache | enabled | enabled | kept |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | unchanged |

## Local vs LB Score

- Local mean: 3.008
- Baseline (previous version): V21 LB 0.00
- Local delta: n/a (V21 was 0.00)
- LB score: **1.53**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## Outcome Analysis

LB 1.53, recovered from V21's 0.00 and set a new personal best above V17's 1.43.

Attribution: the anim bundle's animation-awareness is the most likely contributor. Without it, the solver may read mid-animation frames and issue wrong actions. With it, the solver waits for the board to settle.

The wheelhouse fix is necessary but not sufficient. Without the correct wheelhouse, vLLM cannot install. With it, vLLM installs but does not necessarily improve LB.

MULTIMODAL_UPSCALE=4 (stock) is the bottleneck. V23 local mean 3.008. The next version (V24) will upgrade to MULTIMODAL_UPSCALE=8 and target local mean 5.0+.

## Next Version Plan

V24: upgrade MULTIMODAL_UPSCALE from 4 to 8. Single-variable change. Target LB 2.0+.
