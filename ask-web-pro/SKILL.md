---
name: ask-web-pro
description: 以固定的 Luna Max 子 Agent 操作使用者已登入的 Microsoft Edge，在獨立的 GPT-5.6 Sol Pro 對話原樣送出問題、等待完整回答並帶回母 Agent；支援母 Agent 在同一條對話繼續追問。當使用者說「問網頁 Pro」、「先問 Pro」、「交給 Pro 問」、「讓 Pro 回答後再執行」或要求透過 ChatGPT Pro 網頁取得第二意見時使用。
---

# 問網頁 GPT-5.6 Sol Pro

## 固定執行配置

以母 Agent 負責決策與執行，以固定的 `luna_worker` 子 Agent 負責瀏覽器傳話。每次首次提問與每次追加提問都必須建立全新的、一次性的 Luna thread；工作單必須自足，不得繼承母 Agent 的完整對話。

建立子 Agent 時必須使用：

```text
task_name: web_pro_messenger_<unique_suffix>
fork_turns: none
agent_type: luna_worker
```

`<unique_suffix>` 必須由母 Agent 在每次首次提問或追加提問建立 thread 前重新產生，且只能包含 lowercase letters、digits、underscores；每次都必須是全新的唯一值，不得重用任何既有 canonical task path。若 `spawn_agent` 因 task path 已存在而拒絕，視為名稱生成缺陷，立即改用另一個全新的唯一 `task_name` 重建一次；此重試僅處理名稱碰撞，不得改用舊 thread、`followup_task` 或其他模型，也不得擴張為其他失敗的重試。

不得在建立子 Agent 時直接指定 `model` 或 `reasoning_effort` override；正式角色設定必須由 `luna_worker` runtime 提供。

建立後立即核對 runtime metadata。若 `spawn_agent` 拒絕 `luna_worker`，或實際 `agent_role`、`model`、`effort` 不是分別為 `luna_worker`、`gpt-5.6-luna`、`max`，立即停止並回報實際值；不得改用 Sol、default 或任何其他代理／模型，也不得以別名掩飾不符的 runtime 設定。

## Edge 啟動授權

使用者已永久允許本 Skill 在 Microsoft Edge 未執行時，依 `chrome:control-chrome` Skill 的官方流程直接啟動 Edge，不必再次詢問。啟動後等待兩秒，再重試 Edge 連線一次。

此授權僅限為本 Skill 啟動 Edge；不包含安裝或重裝外掛、修改瀏覽器設定、切換其他瀏覽器，或讀取 cookies、儲存區與登入資料。若重試仍失敗，依 Edge 疑難排解規則停止並回報。

## 首次提問

1. 母 Agent 先產生本輪全新的 lowercase/digits/underscore `<unique_suffix>`，再以 `task_name: web_pro_messenger_<unique_suffix>`、`fork_turns: none`、`agent_type: luna_worker` 建立全新的子 Agent，並在工作單中指示它使用 `chrome:control-chrome` Skill，以穩定選擇器 `edge` 操作使用者已登入的 Microsoft Edge。Edge 是硬性限制，不得改用 Chrome、內建瀏覽器或其他瀏覽器。不得重用任何既有 canonical task path；若僅因 task path 已存在而被拒絕，依固定配置只用另一個全新的唯一 task_name 重建一次。
2. 子 Agent 在任何瀏覽器操作前完整讀取該 Skill，並遵守其瀏覽器選擇、文件載入、登入與禁止讀取 cookies／儲存區的規則。
3. 前往 ChatGPT，建立全新的獨立對話。
4. 在送出問題前，分別操作並確認模型選擇器的實際選取值為 `GPT-5.6 Sol`，模式／推理強度的實際選取值為 `Pro`。若需要切換，先選擇 `GPT-5.6 Sol`，再選擇 `Pro`，最後重新讀取兩個控制項的可見選取狀態。
5. 帳號或方案標籤顯示 `Pro` 不算模式確認；不得把帳號方案、頁面標題或推測值當成模型／模式證據。若畫面只有 `ChatGPT`、`Auto`、`Instant`、`Thinking` 或任何非指定值，或任一控制項無法明確確認，不得送出問題，立即回報母 Agent。
6. 將母 Agent 提供的問題原樣貼上，不得自行摘要、改寫或去敏。若更高優先級的安全、隱私或工作區規則禁止傳送，停止並明確指出被禁止的資料類別，不得暗中改寫。
7. 送出後持續觀察生成狀態。不要只固定等待一段時間後盲目擷取。
8. 每次等待不得阻塞超過 60 秒；定期檢查回答是否仍在生成。總等待上限預設 20 分鐘。
9. 僅在生成已停止且最新回答內容穩定後擷取答案。保留原始段落、清單、Markdown 與程式碼區塊。
10. 子 Agent 不執行答案中的建議，只負責傳話與狀態回報。

## 回傳格式

子 Agent 回傳母 Agent：

```text
status: completed | blocked | timed_out | failed
messenger_role: 建立後 runtime 實際角色，完成時必須是 luna_worker
messenger_model: 建立後 runtime 實際模型，完成時必須是 gpt-5.6-luna
messenger_effort: 建立後 runtime 實際推理強度，完成時必須是 max
model_label: 送出前實際看到的模型選取標籤，完成時必須是 GPT-5.6 Sol
mode_label: 送出前實際看到的模式／推理強度標籤，完成時必須是 Pro
conversation_url: 可重新開啟的對話網址；無法取得時說明
question_sent: 實際送出的完整問題
answer: GPT-5.6 Sol Pro 的完整回答
issues: 登入、額度、介面、逾時或擷取問題；沒有則寫 none
```

只有 `messenger_role: luna_worker`、`messenger_model: gpt-5.6-luna`、`messenger_effort: max`，且 `model_label: GPT-5.6 Sol`、`mode_label: Pro` 同時成立，才能回報 `status: completed`。不得宣稱未在 runtime 或畫面上確認的信使角色、模型、推理強度、網頁模型、模式或完成狀態。

## 母 Agent 追加提問

保留上一輪的 `conversation_url` 作為追加提問的對話位置，但每次追加提問都必須先產生新的 lowercase/digits/underscore `<unique_suffix>`，再以全新的 `task_name: web_pro_messenger_<unique_suffix>` 建立全新的、一次性的 `luna_worker` thread（`fork_turns: none`），並把必要上下文與追加問題完整放入自足工作單。不得重用任何既有 canonical task path，不得對已完成的 Luna thread 使用 `followup_task`，也不得重用上一輪的子 Agent；若僅因 task path 已存在而被拒絕，只能以另一個全新的唯一 task_name 重建一次。

新的 Luna thread 應回到同一條 ChatGPT 對話：優先使用仍存在的分頁，否則開啟保存的 `conversation_url`。確認 GPT-5.6 Sol 與 Pro 的可見選取值後，原樣送出追加問題，依首次提問相同的完成判定擷取答案，再回傳母 Agent；若 runtime 三欄或網頁兩欄任一不符，立即停止並回報實際值。

預設最多往返五輪。若仍有關鍵歧義，母 Agent 整理未決事項並向使用者說明，不要無限互問。

## 母 Agent 收尾

母 Agent 將 Pro 回答視為外部建議，先核對任務條件與本地證據，再決定是否執行。若使用者要求實作，母 Agent 負責修改、測試與最終驗收；子 Agent 不直接修改專案。
