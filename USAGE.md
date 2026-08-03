# 指派 Luna 使用說明

`delegate-luna` 是一個 Codex Skill。它讓 Sol／母代理保留全局判斷與驗收責任，只把明確、有限、可獨立完成的工作，以一次性自足工作單交給 `luna_worker`。

## 版本要求：正式版 0.146.0

這個倉庫的可重現基準是：

```text
codex-cli 0.146.0
```

不要把 `0.146.0-alpha.*` 當成相同版本。Alpha 是預覽建置，功能、協定或桌面整合可能和正式版不同。較舊 Codex 也可能不知道如何載入這裡使用的自訂代理模型與思考強度。

安裝或固定正式版：

```text
npm install -g @openai/codex@0.146.0
codex --version
```

預期輸出必須是：

```text
codex-cli 0.146.0
```

如果使用 Codex 桌面 App，要注意 App 可能使用自己內建的 App Server runtime。npm 顯示 `0.146.0` 不代表桌面 App 一定正在使用它；請以桌面 App 實際啟動路徑、runtime metadata 或日誌中的 `Current reported app-server version` 為準。不要手動覆寫 WindowsApps 或其他受保護的 App 檔案。

版本不是正式 `0.146.0` 時，先不要宣稱 Luna Max 已設定成功。至少要另外驗證：

- 多代理工具能啟動自訂角色。
- `luna_worker` 能從正式代理目錄載入。
- 子代理 metadata 顯示 `gpt-5.6-luna`。
- reasoning effort 顯示 `max`。

## 可以直接貼給另一個 Codex 的安裝提示詞

```text
請從 https://github.com/Jason-King-Wang/skill-any-way 安裝 delegate-luna。

限制：
1. 只處理 delegate-luna，不得讀寫、搬移或上傳其他 Skill。
2. 先檢查實際 Codex CLI 與 App Server runtime；可重現目標是正式版 0.146.0，不得把 0.146.0-alpha.* 當成相同版本。
3. 若需要安裝 CLI，使用 @openai/codex@0.146.0，安裝後驗證 codex --version。
4. 把 delegate-luna 安裝到 ~/.codex/skills/delegate-luna/。
5. Skill 內的 assets/luna-worker.toml 只是可攜來源；另複製到 ~/.codex/agents/luna-worker.toml，或目前專案的 .codex/agents/luna-worker.toml。
6. 若任何目標檔已存在，先比較並回報，不得直接覆寫。
7. 確認 multi_agent_v2 = true，完整重新啟動 Codex。
8. 最後驗證 agent_role=luna_worker、model=gpt-5.6-luna、effort=max；任一項不符都不得宣稱成功。
9. 不得手動覆寫 WindowsApps 或其他受保護的桌面 App 檔案。

完成後回報：實際 CLI 版本、實際 App Server 版本、安裝路徑、feature 狀態、代理 metadata，以及所有異動檔案。
```

## 需要安裝的兩個部分

| 部分 | 來源 | Codex 載入位置 | 用途 |
| --- | --- | --- | --- |
| Skill | `delegate-luna/` | `~/.codex/skills/delegate-luna/` | 告訴母代理如何拆分、派工、驗收及失敗歸因 |
| 自訂代理 | `delegate-luna/assets/luna-worker.toml` | `~/.codex/agents/luna-worker.toml` | 固定子代理為 `gpt-5.6-luna`、`max` |

`delegate-luna/agents/openai.yaml` 是 Skill 的介面資訊，不是 `luna_worker` 的執行設定。自訂代理 TOML 必須另外放到正式代理目錄，Codex 才會載入。

## 安裝

### Windows PowerShell

以下命令只適合全新安裝；如果目標已存在，會停止，不會覆寫。

```powershell
git clone https://github.com/Jason-King-Wang/skill-any-way.git
Set-Location .\skill-any-way

$codexRoot = Join-Path $env:USERPROFILE ".codex"
$skillTarget = Join-Path $codexRoot "skills\delegate-luna"
$agentTarget = Join-Path $codexRoot "agents\luna-worker.toml"

if (Test-Path -LiteralPath $skillTarget) { throw "Skill 已存在：$skillTarget" }
if (Test-Path -LiteralPath $agentTarget) { throw "代理設定已存在：$agentTarget" }

New-Item -ItemType Directory -Path (Join-Path $codexRoot "skills") -Force | Out-Null
New-Item -ItemType Directory -Path (Join-Path $codexRoot "agents") -Force | Out-Null
Copy-Item -LiteralPath .\delegate-luna -Destination $skillTarget -Recurse
Copy-Item -LiteralPath .\delegate-luna\assets\luna-worker.toml -Destination $agentTarget
```

### macOS／Linux

```bash
git clone https://github.com/Jason-King-Wang/skill-any-way.git
cd skill-any-way

CODEX_ROOT="${CODEX_HOME:-$HOME/.codex}"
test ! -e "$CODEX_ROOT/skills/delegate-luna" || { echo "Skill 已存在"; exit 1; }
test ! -e "$CODEX_ROOT/agents/luna-worker.toml" || { echo "代理設定已存在"; exit 1; }

mkdir -p "$CODEX_ROOT/skills" "$CODEX_ROOT/agents"
cp -R ./delegate-luna "$CODEX_ROOT/skills/delegate-luna"
cp ./delegate-luna/assets/luna-worker.toml "$CODEX_ROOT/agents/luna-worker.toml"
```

### 專案限定安裝

若只想在單一專案使用，將代理設定放在該專案的：

```text
.codex/agents/luna-worker.toml
```

Skill 仍可安裝在個人的 `~/.codex/skills/delegate-luna/`；也可依 Codex 支援的專案 Skill 方式放入專案範圍。

## 確認 Codex 版本與多代理功能

這份設定已在正式版 Codex CLI `0.146.0` 驗證。先檢查：

```text
codex --version
codex features list
```

如果 `multi_agent_v2` 已顯示 `true`，不需要修改。若顯示 `false`，可執行：

```text
codex features enable multi_agent_v2
```

安裝或調整後，請完整退出並重新啟動 Codex。桌面版若仍留在系統匣，也要一併退出。

重新啟動後，需同時通過以下檢查，才算安裝完成：

```text
App Server version = 0.146.0
multi_agent_v2 = true
agent_role = luna_worker
model = gpt-5.6-luna
effort = max
```

## 怎麼觸發

直接在任務中使用下列說法：

```text
指派 Luna 幫我處理這個任務。
```

```text
指派路那，把這個問題拆成適合 Luna Max 的一次性子任務。
```

```text
只用 Luna 子代理執行，母代理負責拆分與驗收。
```

也可以明確指定限制：

```text
指派 Luna 修改這個模組。不要改其他檔案；完成後跑指定測試，由母代理驗收。
```

## 正常流程

1. 母代理先理解目標、固定決策並排除歧義。
2. 母代理建立一份不依賴先前對話的完整工作單。
3. 以 `fork_turns = "none"` 啟動 `luna_worker`。
4. Luna 在同一個 thread 內執行、自驗並回報 `PASS` 或 `BLOCKED`。
5. 母代理獨立查證結果，再整合交付。
6. 若驗收失敗，母代理先判斷是工作單、任務切分、Luna、環境或整合問題；修正工作單後開新的 Luna thread，不向舊 thread 發 follow-up。

## 如何確認真的用了 Luna Max

第一個子代理啟動後，檢查介面或 runtime metadata：

```text
agent_role = luna_worker
model = gpt-5.6-luna
effort = max
```

任何一項不符都不算成功；Skill 規定母代理必須停止並回報實際值，不得用其他模型偷偷代做。

也可以直接檢查安裝後的代理設定：

```toml
name = "luna_worker"
model = "gpt-5.6-luna"
model_reasoning_effort = "max"
```

## 適合與不適合的任務

適合交給 Luna：

- 目標與邊界明確的局部實作。
- 固定格式的整理、轉換或批次處理。
- 有清楚通過條件的測試、檢查與修復。
- 不需重新決定產品方向的局部研究。

應由母代理先處理：

- 開放式需求探索與全局架構決策。
- 跨系統取捨或高度耦合的大範圍改動。
- 仍有多種合理解讀、需要使用者選擇的需求。
- 需要長期依賴整段對話才能理解的工作。

## 常見問題

### 找不到 `luna_worker`

確認檔案位於 `~/.codex/agents/luna-worker.toml` 或目前專案的 `.codex/agents/luna-worker.toml`，然後完整重啟 Codex。只把 TOML 留在 Skill 的 `assets/` 不會自動生效。

### Skill 沒有觸發

確認目錄為 `~/.codex/skills/delegate-luna/SKILL.md`，再建立新任務並明確使用「指派 Luna」或「指派路那」。

### 模型或思考強度不正確

比較正式代理目錄中的 TOML 與倉庫的 [`luna-worker.toml`](delegate-luna/assets/luna-worker.toml)，確認 `model` 與 `model_reasoning_effort`，再重啟 Codex。

### Luna 回報 `BLOCKED`

這不代表一定是 Luna 做錯。母代理應先檢查工作單是否缺少路徑、輸入、固定決策、禁止事項或驗收標準，再建立全新的 Luna thread。

## 更新時的安全原則

不要直接覆寫現有安裝。先比較下列兩組內容：

- 倉庫的 `delegate-luna/` 與本機的 `~/.codex/skills/delegate-luna/`
- 倉庫的 `delegate-luna/assets/luna-worker.toml` 與本機的 `~/.codex/agents/luna-worker.toml`

確認沒有自己的客製修改後再更新，並於更新後重新啟動 Codex、檢查 runtime metadata。
