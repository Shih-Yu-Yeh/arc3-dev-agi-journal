+++

## 起點：fork Tufa

Day 0 pipeline 失敗之後，第一要務是在加任何 patch 前先建立可運作的 baseline。我 fork 了 Tufa Labs duck harness（公開 kernel `samrishb/sam-solver`，Tufa 自報 LB 1.21 milestone），用未修改的 Qwen3.6-27B-FP8 model 跑完整 9 小時。LB 0.87 落在 Tufa 自報的 0.5-1.5 範圍內，確認 baseline pipeline 運作正常。

## 為什麼選 Tufa 當 base

Day 0 的 stub 失敗之後，問題不是 V2 會不會得分，而是我的 fork 能不能重現 Tufa 的結果。Tufa Labs 自報同一份 code 的 LB 變動 0.5-1.5，所以任何落在此範圍的分數都算 baseline 成功。

選 Tufa 的原因有三個：它有完整的 5 步循環（observe、reason、act、score、update）；有正確的 submission.parquet schema；而且原作者已經測試過。

## 三個刻意的「不做」

使用未修改的 Tufa Labs source bundle（`jeroencottaar/taaf-kaggle-source-share`）。Stock Qwen3.6-27B-FP8 model（`vrfai-qwen3-6-27b-fp8-hf-snapshot`）。沒有 patch、沒有 graft、沒有 prompt addendum。

三個刻意的「不做」：不升級到 Qwen3.8（V17 才做）、不啟用 FP8 KV cache（V19 才做）、不加 MULTIMODAL_UPSCALE（V24 才做）。重點是先驗證 baseline 端到端跑通並產出真實 LB 分數。

## Stock 配置表

| 參數 | 值 | 理由 |
|---|---|---|
| model | Qwen3.6-27B-FP8 | Stock Tufa 配置 |
| temperature | 0.6 | Stock |
| context window | 32768 | Stock |
| MULTIMODAL_UPSCALE | 4（256x256） | Stock |
| n_passes | 1 | Stock |
| FP8 KV cache | disabled | Stock |

## 0.87 在 Tufa 自報範圍內

- Local mean: 未量測
- Baseline（上一版）: n/a（第一次真實 run）
- Local delta: n/a
- LB score: **0.87**

## 四個基本 marker

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## 三個觀察：wall clock、temperature、context gap

LB 0.87 落在 Tufa Labs 自報的 0.5-1.5 範圍內。Baseline pipeline 運作正常。

三個觀察：

9 小時 wall clock 很緊。Qwen3.6-27B-FP8 在單張 RTX Pro 6000 配 32k context 下，沒辦法在 9 小時內跑完所有 110 個 hidden games。Give-up 機制（每遊戲 max_runtime）在大多數遊戲上啟動，限制貢獻分數。

Local temperature 顯示是 0.0，不是我設定的 0.6。Solver 在選動作時固定用 temperature=0，跟設定的 `LOCAL_ANALYZER_TEMPERATURE` 無關——後者只影響推理階段，不影響動作選擇階段。這個區別對後續 V14 的 temperature 實驗很關鍵。

vLLM server 以 `--max-model-len 65536` 啟動成功，但 analyzer context window 只有 32768。Model 能處理 65k tokens，但 analyzer 只餵 32k。這個缺口是 V6（context budget 增加）想解決的問題。

## 目標突破 1.0

在 V2 上加 program synthesis system prompt。目標：靠鼓勵結構化 tool call 突破 1.0 LB。
