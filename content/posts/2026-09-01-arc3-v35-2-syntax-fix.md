+++
title = "ARC3 V35.2 (LB 1.59): 修對了 Syntax，LB 沒變 — Modules 是 Net-Negative"
date = 2026-09-01T03:38:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "modules", "net-negative"]
categories = ["ARC3 Dev Journal"]
summary = "修了 UnboundLocalError（0 個錯誤）。6 個 modules 正確跑（10 個 experiment events）。LB 1.59，跟 V35.1 broken 分數相同。Modules 確認是淨負面。"
lb_score = "1.59"
version = "V35.2"
status = "CONFIRMED_NET_NEGATIVE"
+++


## 修對了語法，分數沒變

V35.1 有 2443 個 UnboundLocalError。修復是語法上的：在 `step_env` 頂部加 `nonlocal _v35_last_level`。V35.2 套用這個修復，用相同 config 重跑。我假設如果 V35.1 的 1.59 是 bug 造成的（modules 從未執行），那 V35.2 修完後應該得分更高。如果 V35.2 跟 V35.1 同分，那 modules 即使能跑也是淨負面。結果 LB 1.59，跟 V35.1 同分。

## V35.1 bug 修復的歸因實驗

V35.1 有 2443 個 UnboundLocalError。修復是語法上的：在 `step_env` 頂部加 `nonlocal _v35_last_level`。V35.2 套用這個修復，用相同 config 重跑。

假設上，如果 V35.1 的 1.59 是 bug 造成的（modules 從未執行），那 V35.2 修完後應該得分更高（modules 現在執行）。如果 V35.2 跟 V35.1 同分，那 modules 即使能跑也是淨負面。

這是 module 價值最乾淨的歸因實驗。

## 一行 nonlocal 修復

一行修復：

```python
# V35.1 (broken)
def step_env(...):
    if some_condition:
        _v35_last_level = current_level  # local

# V35.2 (fixed)
def step_env(...):
    nonlocal _v35_last_level  # closure 變數
    if some_condition:
        _v35_last_level = current_level
```

其他 config 跟 V35.1 完全相同。

## 其他 config 完全相同

| 參數 | V35.1 | V35.2 | 理由 |
|---|---|---|---|
| nonlocal 宣告 | MISSING | present | 修 closure scope |
| UnboundLocalError 計數 | 2443 | 0 | 修復 |
| modules | 6（instantiated，從未執行） | 6（執行） | 現在跑了 |
| experiment events | 0 | 10 | HypothesisEngine firing |
| MULTIMODAL_UPSCALE | 8 | 8 | 未變 |
| FP8 KV cache | enabled | enabled | 未變 |

## 1.59 跟 V35.1 同分

- Local mean: 6.44
- Baseline（上一版）: V35.1 LB 1.59
- Local delta: n/a
- LB score: **1.59**

## 0 errors + 10 experiment events

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
| HypothesisEngine firing | yes | HYPOTHESIS: Is your win hypothesis still valid? |
| UnboundLocalError | 0（修復） | stdout 中無 UnboundLocalError |

## modules 淨負面最強證據

LB 1.59，跟 V35.1 broken 分數相同。Local mean 6.44（比 V24 local 4.984 +29%）。

![V35.2 vs V24 Per-game 對比](/arc3-dev-agi-journal/images/v35-2-vs-v24-per-game.png)

這是 modules 對 ARC-AGI-3 是淨負面最強的證據：

- V35.1：6 個 modules instantiated，0 個執行（2443 錯誤）。LB 1.59。
- V35.2：6 個 modules instantiated，全部執行（0 錯誤，10 個 experiment events）。LB 1.59。

分數沒變。這代表 modules 的執行本身不影響分數——它們加了 overhead 但沒加價值。

Local mean 改善了（比 V24 +29%），但這改善是在 6 個熟悉的公開遊戲上。Hidden games 懲罰了指令式驅動（directive-driven）行為——modules 主動告訴 solver 該做什麼，這在陌生環境反而干擾了 model 本身的判斷。Local mean 不是 LB score。

Tufa Labs 的經驗也顯示「hand-crafted tools degrade model performance」。每次 module 加法（V25 grafts、V33/V34 NOOA、V35 modules）都退步 LB。

33 天開發循環的結論：V24 config（9 patches，無 modules）在 LB 2.56 是局部最佳。前進方向不是更多 modules，是更好的 base model 或不同的 benchmark。

## Campaign 暫停

開發循環先暫停。V36 會複製 V24 完全相同的 9-patch config + busyaprime 的 4 個 engine findings（frame[-1]、不信 available_actions、ACTION6 必帶 x/y、level order 不等於 difficulty）。無 modules。目標是透過 engine-level 修復達 LB 2.8+。
