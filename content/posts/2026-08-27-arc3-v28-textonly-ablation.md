+++

## Text vs Image

V24 用 image modality（MULTIMODAL_CONTEXT=current_grid 加 MULTIMODAL_UPSCALE=8）。我假設 text-only modality 會更快（無 vision token overhead），而且如果 solver 能從 text 表示讀出盤面，分數可能接近。V28 是 controlled ablation：V24 完全相同 config 加一個改動，把 MULTIMODAL_CONTEXT 從 `'current_grid'` 改成 `'text'`。LB 1.72，比 V24 -0.84。Image modality 顯著重要。

## text-only 會更快的假設

V24 用 image modality（MULTIMODAL_CONTEXT=current_grid 加 MULTIMODAL_UPSCALE=8）。我假設 text-only modality（MULTIMODAL_CONTEXT=text）會更快（無 vision token overhead），而且如果 solver 能從 text 表示讀出盤面，分數可能接近。

V28 是 controlled ablation：V24 完全相同 config 加一個改動，`MULTIMODAL_CONTEXT = 'text'`（原 `'current_grid'`）。如果 LB 顯著下降，image modality 重要。如果 LB 留在 2.56 附近，text 就夠了。

## controlled ablation 設計

單一變數改動：`MULTIMODAL_CONTEXT = 'text'`（原 `'current_grid'`）。其他 patch（FP8 KV、MM_UPSCALE=8、WBC、RE cap、NG）從 V24 保留。

Text 表示是 64x64 ASCII grid 帶 color code。Solver 把它當字串讀，不是當 image 讀。沒有產生 vision tokens。

## 單一變數：MULTIMODAL_CONTEXT

| 參數 | V24 | V28 | 理由 |
|---|---|---|---|
| MULTIMODAL_CONTEXT | current_grid (image) | text | Ablation |
| MULTIMODAL_UPSCALE | 8 | 8 | 未變（text 不適用） |
| FP8 KV cache | enabled | enabled | 未變 |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | 未變 |
| source bundle | anim bundle | anim bundle | 未變 |

## -0.84 的差距

- Local mean: 未量測
- Baseline（上一版）: V24 LB 2.56
- Local delta: n/a
- LB score: **1.72**

## 所有 patch 保留

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

## 空間關係為什麼重要

LB 1.72，比 V24 的 2.56 -0.84。Image modality 顯著重要。

0.84 的差距是 image modality 相對 text modality 的價值，其他條件都一樣。Image modality 保留了 text 表示丟失的空間關係。對 ARC-AGI-3 的 cover predicate 與 co-location win condition 來說，空間推理是必要的。

V28 也從 V25 的 1.42 恢復到 1.72，確認 V24 patches（FP8 KV、WBC、RE cap、NG）是必要的，而且 thtennant fork 掉這些 patch 造成 V25 的退步。

V24 config（image modality）是局部最佳點。Text modality 是 vision tokens 太貴時的替代方案，但對 ARC-AGI-3 來說，image 贏。

## V31 的 NOOA 計畫

V30 是 synthesis AVO 實驗。V31 會在 V24 config 上加 NOOA modules。目標是透過 cross-game learning 達 LB 3.0+。
