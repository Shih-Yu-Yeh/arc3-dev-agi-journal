+++
title = "About"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Vincent — ocean240812

軟體工程師，專注於 AI agent 開發、大型語言模型推理、code competition pipeline。

## 聯絡

- Email: ocean240812@gmail.com
- 姓名: Vincent
- Kaggle: [ocean240812](https://www.kaggle.com/ocean240812) (ARC-AGI-3 Rank 59 / 2708 teams)
- GitHub: [@ocean240812](https://github.com/ocean240812)

## 為什麼參加 ARC Prize 2026 (ARC-AGI-3)

ARC-AGI-3 是第一個互動式 AI agent benchmark：110 個 hidden games、單 pass、9 小時 wall clock、RHAE 計分。我參加是為了驗證三個假設：

1. Qwen3.8-27B-FP8 加 vision upsampling 能在 hidden games 上達到 LB 2.0+。
2. Tufa Labs duck harness 是穩固的基礎；targeted patches 勝過 custom modules。
3. Local mean score 與 leaderboard score 在 hidden games 上無相關。

結果：V24 在 2026-08-25 透過把 `MULTIMODAL_UPSCALE` 從 4 升到 8 達到 LB 2.56。後續每個 module 加法 (V25 grafts、V33/V34 NOOA、V35 modules) 都退步到 V24 之下。

## 技術棧

| 層 | 選擇 | 理由 |
|---|------|------|
| Model | Qwen3.8-27B-FP8 | V17 從 Qwen3.6-27B-FP8 升級；flash-attention 相容，FP8 權重 |
| Serving | vLLM 0.19.0 + FP8 KV cache + prefix caching | 單張 RTX Pro 6000 跑 65k context window 必需 |
| Framework | TAAF (Tufa Labs duck harness) — forked | 5 步循環 (observe、reason、act、score、update)；穩固 baseline |
| Vision | MULTIMODAL_UPSCALE=8 (512x512 grid rendering) | 決定性因素：V23 (upscale=4) = 1.53，V24 (upscale=8) = 2.56 |
| Pipeline | Kaggle code competition rerun | submission.parquet 透過內部 gateway 觸發 hidden games |
| Instrumentation | level probe graft (Tufa-style per-game JSONL) | 唯一能事後學習 hidden game 行為的方法 |
| Validation | 14-marker patch 驗證 (kaggle kernel_log.json API) | 抓「聲稱有但從未 fire 的 patch」(V14、V31) |

## 5 個最大教訓

1. **Local mean 不是 LB score。** V35.2 local mean 6.44，LB 1.59。Modules 只對 6 個熟悉的公開遊戲有幫助，對 85 個不熟悉的 hidden games 有害。
2. **每次 module 加法都退步 LB。** V25 grafts、V33/V34 NOOA、V35 6-modules。印證 Tufa Labs 經驗上「hand-crafted modules degrade model performance」的宣稱。
3. **V31「7 modules installed」是假的。** Wiring 壞了 (`AttributeError: _system_prompt`)。1348 個 events 中 0 個提到 NOOA。「non-fatal」例外靜默地讓 modules 從未執行。
4. **V35.1 UnboundLocalError x 2443。** Closure scope bug (`_v35_last_level` 缺 `nonlocal` 宣告) 讓 step_env 每次 call 都崩潰。比邏輯 bug 更難抓，因為 kernel 仍完成。
5. **`MULTIMODAL_UPSCALE=8` 是決定性一擊。** V23 到 V24：一個改動，+1.03 LB。對 ARC-AGI-3 來說，vision resolution 比 reasoning effort 更重要。

## 開發憲法 v2.0 (16 條)

完整 16 條在 [Lessons Learned](/lessons-learned/)。重點：

- 本地 dry-run 是每次 push 前的強制步驟。
- 一個版本只改一個變數。V6 違反這條，退步 -0.31。
- 永遠不寫 bare `except Exception: pass`。V31 付出代價。
- 驗證 event 內容，不只驗 event count。V31 有 1348 個 events，但 0 個有用。
