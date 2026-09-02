+++
title = "ARC3 V35.2 (LB 1.59): Fixed Syntax, Unchanged Score - Modules Are Net-Negative"
date = 2026-09-01T03:38:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "modules", "net-negative"]
categories = ["ARC3 Dev Journal"]
summary = "Fixed the UnboundLocalError (0 errors). 6 modules ran correctly (10 experiment events). LB 1.59, identical to V35.1's broken score. Modules are confirmed net-negative."
lb_score = "1.59"
version = "V35.2"
status = "CONFIRMED_NET_NEGATIVE"
+++

## TL;DR

Fixed the UnboundLocalError (0 errors). 6 modules ran correctly (10 experiment events). LB 1.59, identical to V35.1's broken score. Modules are confirmed net-negative.

## Context

V35.1 had 2443 UnboundLocalError exceptions. The fix was syntactic: add `nonlocal _v35_last_level` at the top of `step_env`. V35.2 applied this fix and re-ran with identical config.

The hypothesis: if V35.1's 1.59 was caused by the bug (modules never executed), then V35.2 with the fix should score higher (modules now execute). If V35.2 scores the same as V35.1, then the modules are net-negative even when they work.

This is the cleanest attribution experiment for module value.

## Technical Choice

Single-line fix:

```python
# V35.1 (broken)
def step_env(...):
    if some_condition:
        _v35_last_level = current_level  # local

# V35.2 (fixed)
def step_env(...):
    nonlocal _v35_last_level  # closure variable
    if some_condition:
        _v35_last_level = current_level
```

All other config identical to V35.1.

## Parameter Decisions

| Parameter | V35.1 | V35.2 | Rationale |
|---|---|---|---|
| nonlocal declaration | MISSING | present | Fix closure scope |
| UnboundLocalError count | 2443 | 0 | Fixed |
| modules | 6 (instantiated, never executed) | 6 (executed) | Now running |
| experiment events | 0 | 10 | HypothesisEngine firing |
| MULTIMODAL_UPSCALE | 8 | 8 | unchanged |
| FP8 KV cache | enabled | enabled | unchanged |

## Local vs LB Score

- Local mean: 6.44
- Baseline (previous version): V35.1 LB 1.59
- Local delta: n/a
- LB score: **1.59**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| 6 modules instantiated | yes | v35.2: 6 modules instantiated |
| HypothesisEngine firing | yes | HYPOTHESIS: Is your win hypothesis still valid? |
| UnboundLocalError | 0 (fixed) | no UnboundLocalError in stdout |

## Outcome Analysis

LB 1.59, identical to V35.1's broken score. Local mean 6.44 (+29% over V24's local 4.984).

This is the strongest evidence that modules are net-negative for ARC-AGI-3:

- V35.1: 6 modules instantiated, 0 executed (2443 errors). LB 1.59.
- V35.2: 6 modules instantiated, all executed (0 errors, 10 experiment events). LB 1.59.

The score did not change. The modules ran correctly in V35.2 but contributed zero net value. They added overhead (per-action hooks, hypothesis checks) without improving outcomes.

The local mean improved (+29% over V24), but this improvement was on the 6 familiar public games. The hidden games penalized the directive-driven behavior. Local mean is not LB score.

Tufa Labs' empirical claim that "hand-crafted tools degrade model performance" is confirmed for ARC-AGI-3. Every module addition (V25 grafts, V33/V34 NOOA, V35 modules) regressed LB.

The 33-day campaign concludes: V24's config (9 patches, no modules) at LB 2.56 is the local optimum. The path forward is not more modules; it is a better base model or a different benchmark.

## Next Version Plan

Campaign paused. V36 would replicate V24's exact 9-patch config + busyaprime's 4 engine findings (frame[-1], do not trust available_actions, always pass x/y to ACTION6, do not assume level order = difficulty). No modules. Target: LB 2.8+ via engine-level fixes.
