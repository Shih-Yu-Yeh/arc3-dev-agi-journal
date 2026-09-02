+++
title = "LB Progress"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Leaderboard Progress — 21 Submissions (2026-07-31 to 2026-09-01)

## Score Timeline

![LB Score Timeline](/images/lb-progress.png)

## Patch Count vs LB Score

![Patch vs LB](/images/patch-vs-lb.png)

The right chart makes a counter-intuitive point: more verified patches do not imply a higher LB score. V24 (9 patches) reached 2.56. V35.2 (10 patches) reached 1.59. The patches that matter are vision resolution and FP8 KV cache — module-style instrumentation consistently regresses.

## All Submissions

| # | Date | Version | LB | Delta | Patches Fired | Status |
|---|------|---------|-----|-------|---------------|--------|
| 1 | 2026-07-31 | Stub | ERROR | — | 0 | Pipeline test failed |
| 2 | 2026-08-01 | V2 | 0.87 | — | 3 (T, CW, VLLM) | Baseline |
| 3 | 2026-08-02 | V4 | 1.06 | +0.19 | 5 | Triple-patch |
| 4 | 2026-08-04 | V6 | 0.75 | -0.31 | 5 | Context budget regression |
| 5 | 2026-08-08 | V9 | 1.10 | +0.35 | 3 | yw8837 replica |
| 6 | 2026-08-09 | V10 | 0.79 | -0.31 | 3 | Pure baseline calibration |
| 7 | 2026-08-10 | V13 | 0.87 | +0.08 | 0 | Registration mode only |
| 8 | 2026-08-11 | V14 | ERROR | — | 3 | Temperature patch never fired |
| 9 | 2026-08-15 | V15 | 0.80 | -0.07 | 3 | DVM fork |
| 10 | 2026-08-17 | V17 | 1.43 | +0.63 | 5 | Qwen3.8 breakthrough |
| 11 | 2026-08-18 | V19 | 1.17 | -0.26 | 9 | All patches fired but LB regressed |
| 12 | 2026-08-22 | V21 | 0.00 | -1.17 | 9 | Dummy submission 3411 bytes |
| 13 | 2026-08-23 | V21 (resubmit) | 0.00 | 0.00 | 9 | Same dummy issue |
| 14 | 2026-08-24 | V23 | 1.53 | +1.53 | 8 | Anim bundle + Qwen3.8 |
| 15 | 2026-08-25 | **V24** | **2.56** | **+1.03** | 9 | **Winner: MULTIMODAL_UPSCALE=8** |
| 16 | 2026-08-26 | V25 | 1.42 | -1.14 | 6 | 7 grafts but missing FP8/WBC/RE/NG |
| 17 | 2026-08-27 | V28 | 1.72 | +0.30 | 9 | Text-only ablation |
| 18 | 2026-08-29 | V33 | 1.51 | -0.21 | 10 | NOOA v2 regressed |
| 19 | 2026-08-30 | V34 | 2.24 | +0.73 | 10 | NOOA v3 recovered but below V24 |
| 20 | 2026-08-31 | V35.1 | 1.59 | -0.65 | 10 | 2443 UnboundLocalError |
| 21 | 2026-09-01 | V35.2 | 1.59 | 0.00 | 10 | Syntax fixed, modules net-negative |

**Patch legend**: T = temperature set, CW = context window set, VLLM = vLLM started, FP8 = FP8 KV cache, MM8 = MULTIMODAL_UPSCALE=8, Q38 = Qwen3.8 model, WBC = writable bundle copy, RE = reasoning effort cap, NG = noop guard verified, NOOA = NOOA modules active, V35M = V35 6 modules instantiated.

## Key Observations

### Local Mean vs LB Score

The biggest gap: V35.2 local mean 6.44 (29% above V24's local mean 4.984), but LB 1.59 (38% below V24's 2.56). Modules only help on familiar public games; hidden games penalize directive-driven behavior.

### The V24 Difference

V24 was the only submission that simultaneously had:
- FP8 KV cache (verified in vLLM launch command: `--kv-cache-dtype fp8`)
- MULTIMODAL_UPSCALE=8 (verified: `Patched MULTIMODAL_UPSCALE to 8`)
- Qwen3.8-27B-FP8 model (verified: `Qwen/Qwen3.8-27B-FP8`)
- Writable bundle copy (avoided V11's read-only filesystem error)
- Reasoning effort cap to `high`
- Noop guard verified (`hard_noop_guard = True`)
- Animation-awareness on (anim bundle)

V23 had all of the above except `MULTIMODAL_UPSCALE=8`. Result: 1.53. Single change, +1.03 LB.

### The V25 Mistake

V25 added 7 grafts (winframe, goalkeep, clockwatch, hudmask, clickmap, searchmap, lawbook) on top of V24's MM_UPSCALE=8. But the V25 implementation regressed four critical patches V24 had: FP8 KV cache, writable bundle copy, reasoning effort cap, noop guard. LB dropped 2.56 to 1.42.

The lesson: adding features is not free if the foundation regresses.

### The V31 Phantom

V31 stdout shows `v31: 7 modules installed:` and `v31: Ready. Expected: v24(2.56) + cross-game learning + efficiency = 3+`. But events.jsonl contains 0 NOOA mentions across 1348 events. A non-fatal `AttributeError: _system_prompt` silently disabled every module. Not submitted to LB.

### The V35.1 / V35.2 Lesson

V35.1 had 2443 `UnboundLocalError: _v35_last_level` exceptions in step_env (closure scope bug). LB 1.59.

V35.2 fixed the syntax (`nonlocal` declaration), 0 errors. LB 1.59. Identical score.

The 6 modules (ReasoningMemory, ExplorationTracker, HypothesisEngine, TransferMechanism, ReflectionRecovery) ran correctly in V35.2 but contributed zero net value. This is the strongest evidence that modules are net-negative for ARC-AGI-3.
