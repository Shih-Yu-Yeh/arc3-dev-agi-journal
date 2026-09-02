+++
title = "ARC3 Day 0 (ERROR): Stub Baseline — Kaggle Code Competition Rerun Pipeline 初體驗"
date = 2026-07-31T08:03:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "pipeline"]
categories = ["ARC3 Dev Journal"]
summary = "首次提交。Stub baseline 帶 file_name 參數。回傳 ERROR。學會 commit run 與 submission rerun 的差別。"
lb_score = "ERROR"
version = "Stub"
status = "ERROR"
+++

## TL;DR

首次提交。Stub baseline 帶 file_name 參數。回傳 ERROR。學會 commit run 與 submission rerun 的差別。

## Context

ARC-AGI-3 是 Kaggle code competition，不是 research competition。提交流程分兩階段：(1) commit run 在公開 25 個遊戲上跑通 notebook，產出 submission.parquet；(2) hidden rerun 由 Kaggle 內部 gateway (`http://gateway:8001/api/games`) 提供 110 個 hidden games 重新執行計分。

這個 stub 用來測試 pipeline 機制：kernel push、commit run、submission、LB 顯示。預期會失敗，問題是失敗在哪。

## 技術選擇

使用最簡單的 submission.parquet 格式：一列資料帶 `file_name` 參數。沒有 solver、沒有 vLLM、沒有 model。只是一個 Python cell 寫出一列 parquet。

刻意選最簡配置：把 pipeline 問題與 solver 問題隔開來看。

## 參數決策

| 參數 | 值 | 理由 |
|---|---|---|
| submission 格式 | 1 列, file_name 參數 | 最小測試 |
| solver | 無 | 隔離 pipeline 與 solver |
| model | 無 | 跳過 vLLM 啟動 |
| GPU | T4 (預設) | 最便宜 |

## Local vs LB Score

- Local mean: n/a (無 solver)
- Baseline (上一版): n/a
- Local delta: n/a
- LB score: **ERROR**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| (無 - stub) | n/a | n/a |

## 結果分析

提交回傳 ERROR。根因：`file_name` 參數不是 ARC-AGI-3 合法的 submission 欄位。競賽要求 `row_id`、`game_id`、`end_of_game`、`score` 四個欄位。

兩個學習：

第一，Kaggle code competition rerun pipeline 對 submission.parquet schema 很嚴格。研究型競賽能用的 1 列 stub 在 code competition with hidden rerun 不能用。

第二，commit run 與 submission rerun 是兩個獨立的失敗點。commit run 成功 (notebook 跑通並寫出 submission.parquet)，但 submission rerun 因 schema 錯誤而失敗。需要分開檢查。

下一版 (V2) 會改用正確 schema 並加上真實 solver。

## 下一版計畫

改用 Tufa Labs duck harness fork。使用正確的 submission.parquet schema (`row_id`、`game_id`、`end_of_game`、`score`)。預期 LB 0.5-1.5（依 Tufa Labs 自報的變動範圍）。
