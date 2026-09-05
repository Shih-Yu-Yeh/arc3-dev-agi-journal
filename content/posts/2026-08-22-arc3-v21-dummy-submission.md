+++

## Dummy 提交事故

V19 退步到 1.17，我假設 2-pass visible updates（每遊戲玩兩次，保留較高分）會讓 solver 在第一次 timeout 的遊戲上有第二次機會。V21 在 V19 的 9 個 patch 上加 2-pass，所有 patch 驗證 fire，預期 LB 1.5+。結果 LB 0.00——submission.parquet 是 3411 byte 的 dummy placeholder，不是真實遊戲分數。

## 2-pass 的假設

V19 退步到 1.17。我假設 2-pass visible updates（每遊戲玩兩次，保留較高分）會讓 solver 在第一次 timeout 的遊戲上有第二次機會，恢復退步。

V21 在 V19 的 9 個 patch 上加 2-pass。所有 patch 驗證 fire。預期 LB 1.5+。

結果 LB 0.00。submission.parquet 是 3411 byte，是 dummy placeholder，不是真實遊戲分數。

## 2-pass 實作的陷阱

2-pass 機制：用相同遊戲清單跑 `bm.run()` 兩次。兩次都跑完後，每遊戲取較高分寫進 submission.parquet。

2-pass 實作在 notebook source 是對的。問題不在 2-pass 邏輯，在 submission.parquet schema。

3411 byte 的 submission.parquet 是 `KAGGLE_IS_COMPETITION_RERUN=False`（commit run）時寫的 placeholder。Hidden rerun 應該用真實分數覆寫它。但 hidden rerun 也產出了 3411 byte 檔案，表示 hidden rerun 沒實際跑 solver。

## try/except 的設計缺陷

| 參數 | V19 | V21 | 理由 |
|---|---|---|---|
| n_passes | 1 | 2 | Visible updates |
| submission.parquet 大小 | ~5KB（真實） | 3411B（dummy） | Pipeline 失敗 |
| patches | 9 個 fire | 9 個 fire | 全部驗證 |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | 未變 |

## 0.00 的衝擊

- Local mean: 未量測
- Baseline（上一版）: V19 LB 1.17
- Local delta: n/a
- LB score: **0.00**

## 9 patch 都 fire 但無效

| Patch | 是否 fire? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort: 'high' |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## 3411 byte placeholder 之謎

LB 0.00。submission.parquet 是 3411 byte，是 dummy placeholder 格式。

診斷上，hidden rerun 沒實際跑 solver。commit run 寫 placeholder；hidden rerun 應該用真實分數覆寫。但 hidden rerun 也寫了 placeholder。

三個可能原因：

2-pass 邏輯可能在 hidden rerun 階段崩潰，退回 placeholder 路徑。2-pass code 把 `bm.run()` 包在 try/except，但 except block 寫 placeholder 而不是 re-raise。

Hidden rerun 環境可能跟 commit run 不同。Hidden rerun 用內部 gateway（`http://gateway:8001`），可能無法使用。commit run 用 bundled `environment_files/`。

submission.parquet schema 可能錯了。3411 byte 大小暗示是 1 列 placeholder，不是 25 列或 110 列真實 submission。

V21 resubmit（下一篇）測 0.00 是 transient 還是 persistent。

## resubmit 測 transient vs persistent

V21 resubmit：再 push 同一個 kernel，看 0.00 是 transient 還是 persistent。
