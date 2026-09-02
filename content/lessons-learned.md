+++
title = "Lessons Learned"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Lessons Learned — 33 Days, 21 Submissions, 33 Private Versions

## Development Constitution v2.0 (16 Rules)

A living document, updated after each submission. The rules that prevented regressions are bolded; the rules learned from regressions are marked.

### Process Rules

**§1.** Local dry-run is a mandatory pre-step before every push. No exceptions, even for "one-line fixes."

**§2.** One variable per version. V6 broke this rule (context budget + offline env_dir fix) and regressed -0.31 LB. The cause was unattributable.

**§3.** Never write bare `except Exception: pass`. V31's `system prompt injection failed (non-fatal)` silently disabled every module for the entire 9-hour run.

**§4.** Verify event contents, not just event counts. V31 had 1348 events. None mentioned NOOA. The "modules installed" log was a lie.

**§5.** Every patch must emit a marker print. A patch that silently applies cannot be debugged.

**§6.** Run `kaggle kernels output` AND fetch `kernel_log.json` via `/api/v1/kernels/output` API. The two cover different artifacts.

### Validation Rules

**§7.** Patch verification is a 4-layer check:
1. Syntax: notebook source contains the patch code.
2. Semantic: stdout contains the expected marker print.
3. Behavioral: vLLM / solver launched with the patched config.
4. Result: events.jsonl shows the patch had effect.

V14 passed layers 1-2 but failed layer 3 (temperature never written). V31 passed layers 1-2 but failed layer 4 (NOOA never executed).

**§8.** Local mean is not LB score. Always submit to LB before drawing conclusions.

**§9.** Local mean improvement on public games does not transfer to hidden games. V35.2 local +29% vs V24, LB -38% vs V24.

### Technical Rules

**§10.** Use `frame[-1]`, not `frame[0]`. The first frame of a film strip is not the final board state. On `ls20`, they differ by 4012 of 4096 cells. (Source: busyaprime kernel)

**§11.** Do not trust `available_actions` as a liveness signal. The engine returns an empty frame after game over, but `available_actions` stays full. 2364 of 5000 random steps were blind. (Source: busyaprime kernel)

**§12.** Always pass `x` and `y` to `ACTION6`. Without coordinates, 5 of 25 games raise `KeyError` inside game code. (Source: busyaprime kernel)

**§13.** Do not assume level order equals difficulty order. `baseline_actions` peaks at the last level in only 7 of 25 games. (Source: busyaprime kernel)

**§14.** Use writable bundle copy. `/kaggle/input/` is read-only. V11 died at `OSError: [Errno 30] Read-only file system`.

### Resource Rules

**§15.** Weekly GPU quota planning. 30h/week. V35 cycle burned 325h wallclock (vs V24's 60h) due to modules causing infinite loops.

**§16.** Hidden games are air-gapped. No public API exposes them. The only way to learn hidden game behavior is to instrument your own submissions with a level-probe graft.

---

## Five Largest Technical Lessons

### 1. Local Mean is Not LB Score

V35.2 local mean: 6.44 (+29% vs V24 local 4.984).
V35.2 LB score: 1.59 (-38% vs V24 LB 2.56).

Modules that improve performance on the 6 familiar public games hurt performance on the 85 unfamiliar hidden games. The hidden game distribution is adversarial to directive-driven behavior.

### 2. Every Module Addition Regressed LB

| Version | Module Type | Local Δ | LB Δ vs V24 |
|---------|-------------|---------|--------------|
| V25 | 7 TAAF grafts | n/a | -1.14 |
| V33 | NOOA v2 | n/a | -1.05 |
| V34 | NOOA v3 | n/a | -0.32 |
| V35.1 | 6 custom modules | n/a | -0.97 |
| V35.2 | 6 custom modules (fixed) | +29% | -0.97 |

Tufa Labs' empirical claim that "hand-crafted tools degrade model performance" holds for ARC-AGI-3.

### 3. V31 Phantom Modules

V31's stdout claimed `7 modules installed`. Reality: 0 modules executed.

Root cause: `solver._system_prompt` attribute did not exist on `HarnessSolver`. The system prompt injection code crashed with `AttributeError`, which was caught by a bare `except Exception: pass`. Modules loaded but had no injection point.

Detection: grep events.jsonl for "NOOA" — V31 had 0 mentions, V32 (fixed) had 469.

### 4. V35.1 UnboundLocalError x 2443

```python
def step_env(...):
    if some_condition:
        _v35_last_level = current_level  # assignment makes it local
    # ...later...
    if _v35_last_level != current_level:  # UnboundLocalError if branch not taken
        ...
```

Fix: `nonlocal _v35_last_level` at the top of `step_env`.

This bug is harder to catch than logic bugs because:
- The kernel still completed (exceptions caught).
- stdout was 624 KB (vs 134 KB for V24) — full of error noise.
- LB returned a real score (1.59), so the submission looked "successful."

Detection: grep stdout for `UnboundLocalError`. V35.1: 2443 hits. V35.2: 0 hits.

### 5. MULTIMODAL_UPSCALE=8 was the Decisive Move

V23 (LB 1.53) and V24 (LB 2.56) differ by exactly one change:

```python
# V23
'MULTIMODAL_UPSCALE': '4',

# V24
'MULTIMODAL_UPSCALE': '8',
```

Local mean: V23 3.008, V24 4.984 (+66%). LB: V23 1.53, V24 2.56 (+67%).

Vision resolution matters more than reasoning effort, context window size, or module sophistication for ARC-AGI-3. The 512x512 grid rendering preserves sub-cell patterns that 256x256 loses.

---

## What Did Not Work

| Approach | Version | Outcome | Diagnosis |
|----------|---------|---------|-----------|
| Context budget 32768 → 49152 | V6 | -0.31 LB | Larger context dilutes attention on small boards |
| Temperature 0.6 → 0.3 | V14 | ERROR | Patch never wrote the env var; kernel ERROR'd on unrelated issue |
| 2-pass visible updates | V21 | 0.00 LB | submission.parquet was 3411-byte dummy; pipeline issue |
| 7 TAAF grafts on top of V24 | V25 | -1.14 LB | Lost FP8 KV, WBC, RE, NG vs V24 |
| Text-only ablation | V28 | -0.84 LB | Hidden games prefer image modality |
| NOOA v2 (Memory + Supervisor) | V33 | -1.05 LB | Hysteresis logic redirects solver too aggressively |
| NOOA v3 (V31 done right) | V34 | -0.32 LB | Wiring fixed; modules still net-negative |
| 6 custom modules + closure bug | V35.1 | -0.97 LB | 2443 UnboundLocalError in step_env |
| 6 custom modules + syntax fix | V35.2 | -0.97 LB | Modules run correctly; still net-negative |

## What Worked

| Approach | Version | Outcome | Why |
|----------|---------|---------|-----|
| Tufa Labs duck harness fork | V2 | 0.87 LB | Solid base; 5-step cycle is sound |
| Program synthesis prompt | V4 | +0.19 LB | Encourages structured tool calls |
| Qwen3.8-27B-FP8 upgrade | V17 | +0.63 LB | Better tool-call accuracy than Qwen3.6 |
| Anim bundle + wheelhouse fix | V23 | +1.53 LB | Animation-awareness matters |
| MULTIMODAL_UPSCALE 4 → 8 | V24 | +1.03 LB | Vision resolution is the bottleneck |
