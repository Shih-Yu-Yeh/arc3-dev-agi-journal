+++
title = "ARC3 V35.1 (LB 1.59): 6 Modules + 2443 UnboundLocalError — Closure Scope Bug"
date = 2026-08-31T01:24:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "modules", "closure-bug", "UnboundLocalError"]
categories = ["ARC3 Dev Journal"]
summary = "加 6 個 custom modules (ReasoningMemory、ExplorationTracker、HypothesisEngine、TransferMechanism、ReflectionRecovery)。step_env 因缺少 nonlocal 宣告產生 2443 個 UnboundLocalError。LB 1.59，比 V34 -0.65。"
lb_score = "1.59"
version = "V35.1"
status = "BUG"
+++

## TL;DR

加 6 個 custom modules (ReasoningMemory、ExplorationTracker、HypothesisEngine、TransferMechanism、ReflectionRecovery)。step_env 因缺少 nonlocal 宣告產生 2443 個 UnboundLocalError。LB 1.59，比 V34 -0.65。

## Context

V34 的 NOOA v3 拿到 2.24 但仍低於 V24 的 2.56。假設：更通用的 module 架構 (6 個 modules 涵蓋 explore/hypothesize/verify/transfer/reflect) 會勝過 NOOA 的狹窄 memory+supervisor 設計。

V35.1 加了 6 個 modules：
- M1. ReasoningMemory：儲存學習過程，不是答案
- M2. ExplorationTracker：系統化動作測試
- M3. HypothesisEngine：form + verify + 3-miss abandon
- M4. TransferMechanism：level N → N+1 mechanic transfer
- M5. ReflectionRecovery：stuck → reflect on 4 capabilities

預期：LB 3.0+。結果：LB 1.59，比 V34 -0.65。

## 技術選擇

6 個 modules 接進 `step_env`，包 solver 的 `step` 函式。每個 module hook 進 observation-reasoning-action loop：

```python
def step_env(...):
    if some_condition:
        _v35_last_level = current_level  # 指派讓它變 local
    # ...後面...
    if _v35_last_level != current_level:  # branch 沒走時 UnboundLocalError
        ...
```

Bug：`_v35_last_level` 在 `if` branch 內被指派。Python 把它當 local 變數。如果 branch 沒走，變數從未指派，後面的存取 raise `UnboundLocalError`。

修法：在 `step_env` 頂部加 `nonlocal _v35_last_level`。但 V35.1 沒有這個修復。

## 參數決策

| 參數 | V34 | V35.1 | 理由 |
|---|---|---|---|
| modules | NOOA v3 | 6 個 custom modules | General intelligence |
| step_env wrap | NOOA wrap | V35 wrap (有 bug) | Closure scope bug |
| MULTIMODAL_UPSCALE | 8 | 8 | 未變 |
| FP8 KV cache | enabled | enabled | 未變 |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | 未變 |

## Local vs LB Score

- Local mean: 未量測
- Baseline (上一版): V34 LB 2.24
- Local delta: n/a
- LB score: **1.59**

## Patch 驗證

| Patch | 是否 fire? | Marker |
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
| step_env wrap | BROKEN | 2443 UnboundLocalError in step_env |

## 結果分析

LB 1.59，比 V34 的 2.24 -0.65。Kernel 完成 (LB 回真實分數)，但每次 step_env call 都崩潰，raise `UnboundLocalError: cannot access local variable '_v35_last_level'`。

各遊戲的錯誤計數：
- sb26：1226 個錯誤
- bp35：481 個錯誤
- ft09：238 個錯誤
- g50t：209 個錯誤
- sk48：170 個錯誤
- r11l：115 個錯誤

總計：2443 個 `UnboundLocalError` 在 step_env。每個例外都被 try/except 接住，所以 kernel 沒崩潰。但每個 step 都 fallback 到預設 solver 行為，繞過 6 個 modules。

這是最陰險的失敗模式：kernel 完成、提交、回真實 LB 分數。沒有 patch 驗證的話，我會結論 6 個 modules 造成退步。實際上 6 個 modules 從未執行任何 step。

這個 bug 難抓因為：
1. Kernel 沒崩潰 (例外被接住)。
2. stdout 是 624 KB (V24 的 134 KB)，滿是錯誤雜訊。
3. LB 回真實分數 (1.59)，所以提交看起來「成功」。

偵測：grep stdout 找 `UnboundLocalError`。V35.1：2443 hits。

## 下一版計畫

V35.2：用 `nonlocal _v35_last_level` 修 closure scope bug。用相同 config 重跑，隔離 bug 的影響。
