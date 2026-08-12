# Codex Skills

這個倉庫保存可移轉的個人 Codex Skills。

## 收錄項目

### `delegate-luna`

要求母代理先完成全局判斷，再以不繼承對話、單向且一次性的自足工作單，把明確、有限、可驗收的工作交給 `gpt-5.6-luna`（Max）執行；母代理負責驗收、整合與失敗歸因。

![指派 Luna 一次性子代理協作流程](docs/delegate-luna-workflow.png)

### `ask-web-pro`

以一次性的 Luna Max 子代理操作使用者已登入的 Google Chrome，在獨立的 ChatGPT 對話中確認 `GPT-5.6 Sol` 與 `Pro` 模式、原樣送出問題，等待完整回答後帶回母代理。支援使用同一個 ChatGPT 對話網址繼續追問。

可用「問網頁 Pro」、「先問 Pro」、「交給 Pro 問」或「讓 Pro 回答後再執行」等說法觸發。

## 移轉安裝

1. 把需要的 skill 資料夾完整複製到 `~/.codex/skills/`：
   - [`delegate-luna`](delegate-luna)
   - [`ask-web-pro`](ask-web-pro)
2. 使用任一 Luna 工作流程時，把 [`delegate-luna/assets/luna-worker.toml`](delegate-luna/assets/luna-worker.toml) 複製到 `~/.codex/agents/luna-worker.toml`；若只想讓單一專案使用，改放在該專案的 `.codex/agents/luna-worker.toml`。
3. 確認 Codex 多代理功能已啟用，並完整重新啟動 Codex。
4. `ask-web-pro` 還需要可由 Codex 控制的 Google Chrome、已登入 ChatGPT 的瀏覽器狀態，以及帳號可用的 `GPT-5.6 Sol`／`Pro` 選項。

## 核心原則

- Luna thread 一律使用 `fork_turns = "none"`。
- 母代理下達完整工作單，Luna 執行、自驗並回報，之後 thread 結束。
- 不向已完成的 Luna thread 發 follow-up；需要追加提問時建立新的 Luna thread。
- Luna 的結果是外部輸入，母代理仍須依本地證據驗收後再決定是否執行。
