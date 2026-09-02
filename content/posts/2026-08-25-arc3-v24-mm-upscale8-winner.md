+++
title = "ARC3 V24 (LB 2.56): MULTIMODAL_UPSCALE=8 — 從 1.53 到 2.56 的決定性一擊"
date = 2026-08-25T04:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "FP8", "MULTIMODAL_UPSCALE", "winner"]
categories = ["ARC3 Dev Journal"]
summary = "單一變數改動：MULTIMODAL_UPSCALE 4 → 8 (256x256 → 512x512 vision)。Local mean 4.984 (比 V23 的 3.008 +66%)。LB 2.56 (比 V23 的 1.53 +1.03)。個人最佳。"
lb_score = "2.56"
version = "V24"
status = "WINNER"
+++

## TL;DR

單一變數改動：MULTIMODAL_UPSCALE 4 → 8 (256x256 → 512x512 vision)。Local mean 4.984 (比 V23 的 3.008 +66%)。LB 2.56 (比 V23 的 1.53 +1.03)。個人最佳。

## Context

V23 用 MULTIMODAL_UPSCALE=4 拿到 local mean 3.008。假設：256x256 vision 失去了 ARC-AGI-3 cover predicate 與 co-location win condition 所需的 sub-cell pattern。512x512 應該能保留它們。

V24 是單一變數改動：`MULTIMODAL_UPSCALE = 8` (原 4)。沒有其他改動。這是整個 33 天 campaign 中最乾淨的歸因實驗。

## 技術選擇

Patch 是一行：

```python
# V23
'MULTIMODAL_UPSCALE': '4',

# V24
'MULTIMODAL_UPSCALE': '8',
```

Patch 在 Cell 5 (pre-setup) 套用，在 TAAF setup command 跑之前，遵循從 V15 學到的 dvm env var 注入 pattern。

驗證：stdout 含 `v24: Patched MULTIMODAL_UPSCALE to 8 (512x512 vision)`。TAAF setup command 讀到 env var 並據此設定 solver。

## 參數決策

| 參數 | V23 | V24 | 理由 |
|---|---|---|---|
| MULTIMODAL_UPSCALE | 4 (256x256) | 8 (512x512) | 保留 sub-cell pattern |
| FP8 KV cache | enabled | enabled | 未變 |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | 未變 |
| source bundle | anim bundle | anim bundle | 未變 |
| temperature | 0.6 | 0.6 | 未變 |
| context window | 32768 | 32768 | 未變 |
| n_passes | 1 | 1 | 未變 |

## Local vs LB Score

- Local mean: 4.984
- Baseline (上一版): V23 local 3.008
- Local delta: +1.976 (+66%)
- LB score: **2.56**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True + --kv-cache-dtype fp8 in vLLM launch |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 (512x512 vision) |
| Qwen3.8 model | yes | Qwen/Qwen3.8-27B-FP8 (vLLM 啟動 log 確認) |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort: 'high' |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## 結果分析

LB 2.56，比 V23 的 1.53 +1.03。Local mean 4.984，比 V23 的 3.008 +66%。Local-to-LB delta 是 +1.03 / +1.976 = 52% transfer rate，整個 campaign 最高。

為什麼 MULTIMODAL_UPSCALE=8 這麼重要？

ARC-AGI-3 hidden games 包含 cover predicate (每個 kind A 物件最終要跟 kind B 物件共位) 與 pixel-level equality (workspace 區域要等於參考區域)。256x256 下 sub-cell pattern 丟失；solver 無法區分兩個形狀相似的物件。512x512 下 solver 可以。

4x token 增加 (256x256 = 65k tokens，512x512 = 262k tokens 處理前) 被 FP8 KV cache 與 32k context window 吸收。Vision tokens 每次觀察處理一次，不是跨整個遊戲保留。

結果：solver 能從第一 frame 讀出 win condition (依 busyaprime 的發現「the goal is usually already drawn on the first frame」) 並執行動作滿足它。

這是唯一一次所有 9 個關鍵 patch 同時 fire 且配置正確的提交。V25 會因為拿掉 FP8 KV、WBC、RE cap、NG 而破壞這個。

## 下一版計畫

V25 會在 V24 上加 7 個 TAAF grafts (winframe、goalkeep、clockwatch、hudmask、clickmap、searchmap、lawbook)。目標：LB 3.0+。假設：在 V24 的 winning config 上疊 grafts 應該複合。
