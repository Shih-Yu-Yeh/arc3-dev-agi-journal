+++

## Patch 沒套上的那 4 層

V14 想測試較低 temperature 對 hidden games 的影響。Notebook source 裡有 patch code，commit run 應該把 `LOCAL_ANALYZER_TEMPERATURE: '0.3'` 寫進環境。實際上沒有——stdout 顯示 temperature 還是 0.6。Kernel 還因為其他原因 ERROR。整個 temperature 實驗等於白做。

## 溫度 0.3 的假設

判斷是：ARC-AGI-3 hidden games 有嚴格 win condition（例如 cover predicate、co-location）。較高 temperature（0.6）引入動作變動，可能破壞這些條件。較低 temperature（0.3）應該產生更確定、可重現的動作。

V14 設計來測這個。Notebook source 含 patch code。Commit run 應該把 `LOCAL_ANALYZER_TEMPERATURE: '0.3'` 寫進環境。

它沒有。

## 為什麼選 0.3 而不是 0.4

Patch 應該在 TAAF setup command 跑之前設定 `LOCAL_ANALYZER_TEMPERATURE='0.3'`。TAAF setup 讀這個 env var 並寫進 `taaf_setup_env.json`，solver 在 runtime 讀取。

Patch code 語法正確。問題在執行順序：env var 在 TAAF setup command 已經讀過預設值（0.6）並寫進 `taaf_setup_env.json` 之後才被設定。Patch 太晚了。

## 聲稱 vs 實際

| 參數 | V13 | V14（聲稱） | V14（實際） |
|---|---|---|---|
| LOCAL_ANALYZER_TEMPERATURE | 0.6 | 0.3 | 0.6 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | 未變 |
| context window | 32768 | 32768 | 未變 |
| MULTIMODAL_UPSCALE | 4 | 4 | 未變 |

## kernel ERROR

- Local mean: n/a（kernel 在 solver 跑之前 ERROR）
- Baseline（上一版）: V13 LB 0.87
- Local delta: n/a
- LB score: **ERROR**

## 4 層檢查的結果

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature 0.3 | NO | log 顯示 LOCAL_ANALYZER_TEMPERATURE: '0.6' |
| temperature set（任何） | yes | temperature': 0.0（action selection） |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## 為什麼 4 層驗證是必要的

LB ERROR。Kernel 在 commit run 階段失敗，solver 沒機會跑。Temperature patch 聲稱有但從未 fire。

![4 層 patch 驗證流程](/images/4layer-validation.png)

依 4 層檢查（§7）：

第 1 層 Syntax：notebook source 含 patch code。PASS。
第 2 層 Semantic：stdout 應該含確認 `LOCAL_ANALYZER_TEMPERATURE='0.3'` 的 marker print。FAIL。實際 stdout 顯示 `LOCAL_ANALYZER_TEMPERATURE: '0.6'`。
第 3 層 Behavioral：vLLM 應該以 temperature 0.3 啟動。NOT REACHED（kernel ERROR）。
第 4 層 Result：events.jsonl 應該顯示 solver config 中 temperature 0.3。NOT REACHED。

4 層檢查抓到 worklog 一開始漏掉的事。V14 的 worklog 條目寫「v9 + low temperature」，但實際 run 從未套用 patch。

根因：env var 設定在錯的 cell。Cell 5（pre-setup）應該設它，但 cell 反而在 Cell 6（TAAF setup）已經擷取預設值之後才設。修法是把 env var 指派移到 Cell 5 頂部，在 TAAF setup command 跑之前。

這個修復從未做。V15 之後 temperature 維持 0.6。Temperature 實驗仍未完成。

關鍵收穫：通過 syntax check（第 1 層）的 patch 仍可能在 semantic check（第 2 層）失敗。4 層驗證是必要的，不是選配。

## Temperature 實驗延後

V15 fork `dvm` 的 kernel 來觀察另一隊怎麼處理 vLLM 啟動。Temperature 實驗延後。
