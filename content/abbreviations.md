+++
title = "Abbreviations"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# 縮寫對照表

本站常用縮寫的全名與簡短說明。

## 比賽與平台

| 縮寫 | 全名 | 說明 |
|------|------|------|
| ARC-AGI-3 | ARC Prize 2026 - ARC-AGI-3 | François Chollet 主辦的互動式 AI agent benchmark，110 hidden games、9 小時 wall clock、RHAE 計分 |
| LB | Leaderboard | 公開排行榜分數 |
| RHAE | Relative Human Action Efficiency | 相對人類 baseline 的動作效率比，ARC-AGI-3 的計分公式 |
| TAAF | Tufa Labs Agent Framework | Tufa Labs 團隊的 duck harness solver，本計畫 fork 自 `jeroencottaar/taaf-kaggle-source-share` |

## 模型與 serving

| 縮寫 | 全名 | 說明 |
|------|------|------|
| Qwen3.6 / Qwen3.8 | Qwen3.6-27B-FP8 / Qwen3.8-27B-FP8 | 通义千問 27B 模型的 FP8 量化版本，V17 從 3.6 升級到 3.8 |
| vLLM | vLLM serving engine | 高吞吐量 LLM serving 框架，本計畫使用 v0.19.0 |
| FP8 KV cache | FP8 量化 KV cache | 把 KV cache 從 BF16 降到 FP8，2x cache 容量換約 1% accuracy loss |

## 視覺與 multimodal

| 縮寫 | 全名 | 說明 |
|------|------|------|
| MULTIMODAL_UPSCALE | 多模態放大倍率 | 把遊戲盤面從 64x64 放大成 256x256 (UPSCALE=4) 或 512x512 (UPSCALE=8)。V24 的決定性改動是把 4 升到 8 |
| MULTIMODAL_CONTEXT | 多模態上下文格式 | `current_grid` (image) 或 `text` (ASCII grid)，V28 測試 text-only |
| anim bundle | animation-aware bundle | `jakobbrggen/taaf-kaggle-source-anim-20260807-anim`，含 animation-awareness |

## Patches

| 縮寫 | 全名 | 說明 |
|------|------|------|
| FP8 | FP8 KV cache patch | 在 vLLM 啟動指令加 `--kv-cache-dtype fp8` |
| MM8 | MULTIMODAL_UPSCALE=8 patch | 把盤面渲染從 256x256 升到 512x512 |
| Q38 | Qwen3.8 model patch | 把 model 從 Qwen3.6 升級到 Qwen3.8 |
| WBC | Writable Bundle Copy patch | 複製 bundle 到 `/kaggle/working/` 避開 read-only filesystem |
| RE cap | Reasoning Effort cap patch | 把 reasoning_effort 從 max 降到 high |
| NG | Noop Guard verified | 驗證 `hard_noop_guard = True` 已啟用 |
| T | Temperature set | 設定 `LOCAL_ANALYZER_TEMPERATURE` |
| CW | Context Window set | 設定 `ANALYZER_CONTEXT_WINDOW` |
| VLLM | vLLM server started | vLLM OpenAI server 啟動成功 marker |
| NOOA | NOOA modules active | NOOA instrumentation 在 events.jsonl 中有實際觸發 |

## Pipeline 與提交

| 縮寫 | 全名 | 說明 |
|------|------|------|
| commit run | Commit 階段 | Kaggle kernel push 後跑通 notebook 的階段，在公開 25 個遊戲上執行 |
| hidden rerun | Hidden 階段 | Kaggle 用內部 gateway 提供 110 個 hidden games 重新執行計分 |
| wall clock | 實際運行時間 | 從啟動到完成的總時間，ARC-AGI-3 限制 9 小時 |
| submission.parquet | 提交檔 | Kaggle code competition 要求的最終產出，含 `row_id`、`game_id`、`end_of_game`、`score` 四欄 |
| GPU quota | GPU 額度 | Kaggle 每週 30 小時免費 GPU |

## 程式與檔案

| 縮寫 | 全名 | 說明 |
|------|------|------|
| kernel | Kaggle Notebook | Kaggle 上的可執行 notebook |
| bundle | TAAF source bundle | Tufa Labs 的 source code dataset，掛載在 `/kaggle/input/datasets/...` |
| wheelhouse | vLLM wheelhouse | 離線安裝 vLLM 與依賴的 wheel 集合，`driessmit1/arc3-vllm-h100-wheelhouse-v3` 是目前 canonical 版本 |
| events.jsonl | 事件記錄 | 每 game 每_pass 的 per-step 軌跡，含 board、action、reward、reasoning |

## 開發憲法條號

lessons-learned.md 中的 §1 ~ §16 是開發憲法 v2.0 的 16 條規則編號，例如 §2 是「一個版本只改一個變數」、§7 是「4 層 patch 驗證」、§9 是「local mean 不會轉移到 hidden games」。

## 版本對照

完整版本對照表見 [LB Progress](/lb-progress/) 頁面的 21 次提交表格。
