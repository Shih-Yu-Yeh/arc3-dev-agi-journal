+++

## 從 0.00 恢復

V21 的 2-pass 實驗兩次都 0.00，2-pass code 被放棄。V23 退回 1-pass，做兩個改動：切到 `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle（內建 animation-awareness），修 wheelhouse owner 從 `jeroencottaar` 改成 `driessmit1`。LB 1.53，從 V21 的 0.00 恢復並創新個人最佳（超過 V17 的 1.43）。

## 2-pass 失敗後的退路

V21 的 2-pass 實驗兩次都 0.00。2-pass code 被放棄。V23 退回 1-pass。

兩個改動：切到 `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle（內建 animation-awareness）；修 wheelhouse owner 從 `jeroencottaar` 改成 `driessmit1`，原本造成靜默 dependency resolution 失敗。

## 為什麼 anim bundle 有效

Anim bundle：`jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle 包含：

- Animation-awareness（等遊戲盤面穩定才讀）
- Hard noop guard（防止無限迴圈）
- Qwen3.8 model patch（自動解析到 `jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`）

Wheelhouse 修復：把 `WHEELHOUSE_OWNER = 'jeroencottaar'` 改成 `WHEELHOUSE_OWNER = 'driessmit1'`。`jeroencottaar` wheelhouse 在 8 月中已 deprecated；`driessmit1/arc3-vllm-h100-wheelhouse-v3` 是目前 canonical wheelhouse。

V23 沒有 MULTIMODAL_UPSCALE patch。Anim bundle 預設 MULTIMODAL_UPSCALE=4。

## wheelhouse deprecation 的歷史

| 參數 | V21 | V23 | 理由 |
|---|---|---|---|
| source bundle | V19 預設 bundle | jakobbrggen anim bundle | Animation-awareness |
| wheelhouse owner | jeroencottaar | driessmit1 | 修 deprecation |
| n_passes | 2 | 1 | 從 V21 退回 |
| MULTIMODAL_UPSCALE | 8 | 4（stock） | 退回 stock |
| FP8 KV cache | enabled | enabled | 保留 |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | 未變 |

## 1.53 新個人最佳

- Local mean: 3.008
- Baseline（上一版）: V21 LB 0.00
- Local delta: n/a（V21 是 0.00）
- LB score: **1.53**

## 8 個 marker

| Patch | 是否 fire? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## animation-awareness 為什麼關鍵

LB 1.53，從 V21 的 0.00 恢復並創新個人最佳（超過 V17 的 1.43）。

歸因上，anim bundle 的 animation-awareness 最可能是主要貢獻者。沒有它，solver 可能讀到動畫中的盤面發錯動作。有了它，solver 會等盤面動畫結束、畫面穩定後才讀取。

Wheelhouse 修復是必要但不充分。沒有正確 wheelhouse，vLLM 無法安裝。有了它，vLLM 安裝但不必然改善 LB。

MULTIMODAL_UPSCALE=4（stock）是瓶頸。V23 local mean 3.008。下一版（V24）會升級到 MULTIMODAL_UPSCALE=8，目標 local mean 5.0+。

## MULTIMODAL_UPSCALE 的目標

V24：把 MULTIMODAL_UPSCALE 從 4 升到 8。單一變數改動。目標 LB 2.0+。
