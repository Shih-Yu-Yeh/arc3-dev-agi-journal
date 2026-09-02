+++
title = "ARC3 V25 (LB 1.42): 7 Grafts + MM8 - Why Adding Features Regressed"
date = 2026-08-26T07:02:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "grafts", "regression"]
categories = ["ARC3 Dev Journal"]
summary = "Added 7 TAAF grafts (winframe+goalkeep+clockwatch+hudmask+clickmap+searchmap+lawbook) on top of V24. But lost FP8 KV, WBC, RE cap, NG in the process. LB 1.42, -1.14 from V24."
lb_score = "1.42"
version = "V25"
status = "REGRESSION"
+++

## TL;DR

Added 7 TAAF grafts (winframe+goalkeep+clockwatch+hudmask+clickmap+searchmap+lawbook) on top of V24. But lost FP8 KV, WBC, RE cap, NG in the process. LB 1.42, -1.14 from V24.

## Context

V24 reached 2.56 with 9 patches. The hypothesis: adding 7 TAAF grafts (winframe, goalkeep, clockwatch, hudmask, clickmap, searchmap, lawbook) on top of V24's config should compound to LB 3.0+.

The grafts were ported from `thtennant/taaf-kaggle-source-share-fork`. Each graft adds a specific instrumentation layer:
- winframe: detects when the win condition is met
- goalkeep: tracks goal state across levels
- clockwatch: monitors time spent per level
- hudmask: overlays HUD information on the vision input
- clickmap: tracks click positions
- searchmap: tracks explored regions
- lawbook: maintains a rule library

The expectation: these grafts would give the solver more information, improving LB.

## Technical Choice

Used `thtennant/taaf-kaggle-source-share-fork` as the source bundle. The 7 grafts were enabled via `TAAF_GRAFTS_FLAGS` env var.

The graft installation was verified: stdout shows `TAAF_GRAFTS FEATURES={"clickmap":true,...,"winframe":true} API_VERSION=1` and individual `[goalkeep] armed`, `[hudmask] armed`, etc. lines.

But: the thtennant fork did not include the V24 patches. The fork was based on an older TAAF version that lacked:
- FP8 KV cache (no `ENABLE_FP8_KV=True`)
- Writable bundle copy (no `refreshing writable bundle copy`)
- Reasoning effort cap (no `reasoning_effort: 'high'`)
- Noop guard verification (no `hard_noop_guard = True`)

V25 had MULTIMODAL_UPSCALE=8 and 7 grafts, but lost 4 critical patches V24 had.

## Parameter Decisions

| Parameter | V24 | V25 | Rationale |
|---|---|---|---|
| MULTIMODAL_UPSCALE | 8 | 8 | kept |
| TAAF grafts | 0 | 7 | winframe+goalkeep+clockwatch+hudmask+clickmap+searchmap+lawbook |
| FP8 KV cache | enabled | MISSING | Lost in fork |
| writable bundle copy | yes | MISSING | Lost in fork |
| reasoning effort cap | yes | MISSING | Lost in fork |
| noop guard verified | yes | MISSING | Lost in fork |
| source bundle | jakobbrggen anim | thtennant fork | Different base |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V24 LB 2.56
- Local delta: n/a
- LB score: **1.42**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 in command 0 |
| TAAF grafts features | yes | TAAF_GRAFTS FEATURES={...} |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-repacked-v1 |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| FP8 KV cache | NO | no ENABLE_FP8_KV marker |
| writable bundle copy | NO | no refreshing bundle copy marker |
| reasoning effort cap | NO | no reasoning_effort marker |
| noop guard verified | NO | no hard_noop_guard marker |

## Outcome Analysis

LB 1.42, -1.14 from V24's 2.56. The 7 grafts fired but 4 critical patches were lost.

The attribution is clear: the lost patches caused the regression, not the grafts. Specifically:

- FP8 KV cache: without it, the 512x512 vision tokens (4x larger) consume more memory. The effective context window shrinks, reducing game history retention.
- Writable bundle copy: without it, the bundle is read-only. Some patches cannot write to the bundle directory, silently failing.
- Reasoning effort cap: without it, reasoning effort is max, which increases latency. More games time out.
- Noop guard: without it, the solver can infinite-loop on stuck games, wasting the 9-hour budget.

The lesson: adding features is not free if the foundation regresses. V25 added 7 grafts but lost 4 patches. The net effect was -1.14 LB.

The fix would be to port the 7 grafts onto V24's bundle, not the thtennant fork. V28 would attempt this with text-only ablation instead.

## Next Version Plan

V26/V27 were MM-ablation experiments (text-only vs image-only). V28 will do a text-only ablation on top of V24's config to test whether hidden games prefer image modality.
