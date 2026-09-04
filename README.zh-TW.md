# Agent Pulse

Agent Pulse 將 Claude Code 與 Codex 的工作狀態同步至 Windows 或 macOS 本機 Dashboard、懸浮視窗和 ESP32 實體三色燈，幫助您即使不盯著終端機，也能掌握 AI 程式設計工作階段的進度。

語言：[English](README.md) | [简体中文](README.zh-CN.md) | 繁體中文 | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Español](README.es.md)

## 目前版本

目前原始碼版本為 `0.4.1`（2026-08-14），ESP32 韌體版本仍為 `0.1.21+22`。單燈維持簡單的預設體驗並自動跟隨最新工作；連接多盞燈後，每盞燈可獨立跟隨最新工作、指定專案或 Claude Code/Codex/WorkBuddy/CodeBuddy 等指定 Agent，並可使用獨立燈光方案。

本版本也會持續顯示每個已連接的藍牙或 USB 裝置及其連線、電量與充電狀態，避免重複 BLE Bridge 程序爭用裝置，並在懸浮視窗顯示最新工作所屬的 Agent 與專案。藍牙繼續支援近場自動連線與 Windows 系統配對連線。應用程式與韌體更新仍優先使用 Gitee，失敗後自動切換 GitHub。完整內容請參閱 [版本更新記錄](CHANGELOG.md)。

## 狀態含義

| 顏色 | 常見含義 |
|---|---|
| 綠 | 工作階段閒置、工作結束或可檢視結果 |
| 黃 | 正在回應、呼叫工具、工具完成後繼續處理或需要補充資訊 |
| 紅 | 權限請求、工具失敗、受阻或需要人工介入 |

狀態依專案目錄儲存和顯示；同一台電腦上的多個專案可同時出現在 Dashboard 中。

## 安裝

### 建議：Windows 安裝程式

下載並執行 `AgentPulseSetup-<版本>.exe`。一般使用者無須自行安裝 Node.js、npm、Python、BLE Bridge、PyInstaller 或 Arduino 工具。

官方下載連結：

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

安裝程式會為目前 Windows 使用者：

- 安裝 Agent Pulse daemon、內建 Node runtime、BLE Bridge 與懸浮視窗；
- 安全合併 Claude Code hooks 和 Codex hooks，不覆寫既有的其他 hooks；
- 設定登入後自動啟動；
- 完成後啟動 Agent Pulse 並開啟 Dashboard。

預設安裝位置通常為：

```text
%LOCALAPPDATA%\AgentPulse
```

安裝完成後，請重新啟動 Claude Code/Codex 或開啟新的工作階段，讓 hooks 重新載入。

### macOS 安裝套件

macOS 12 或更新版本使用依 CPU 架構區分的 `x86_64` 或 `arm64` PKG。正式套件使用 Developer ID 簽署、Apple 公證及票據裝訂；不要關閉 Gatekeeper 或 SIP。`0.3.0` 已正式提供重新簽署、公證的兩種架構專用 PKG，並包含新的裝置連線與更新來源邏輯。完整步驟請參閱 [macOS 安裝說明](macos-install/RELEASE_INSTALL.md)。

### 從原始碼/命令列安裝

這是開發人員路徑，並非 Windows 安裝程式使用者的必要步驟。請參閱文末的[開發人員附錄](#開發人員附錄)。

## 日常使用

### Dashboard

開啟：

```text
http://127.0.0.1:7900
```

也可以從開始功能表的 **Open Dashboard** 開啟。Dashboard 是日常操作入口，可檢視：

- 目前專案、燈色和即時事件；
- BLE 連線狀態、裝置電量（裝置支援時）；
- 懸浮視窗顯示/隱藏；
- 程式與韌體更新；
- 設定入口。

Dashboard 僅監聽本機迴環位址，不會公開至區域網路。

### 設定頁面

在 Dashboard 點擊「設定」開啟設定頁面。設定頁面預設位址為：

```text
http://127.0.0.1:4321/?lang=zh
```

可調整通知、卡住判定時間、各類事件的顏色/閃爍/呼吸效果、亮度和聲音等。「安裝 Claude Code Hooks」與「安裝 Codex Hooks」會自動識別 Windows 或 macOS 並使用對應安裝程式。`7900` 是 Dashboard，`4321` 是獨立設定頁面，兩者用途不同。

### Claude Code 與 Codex 整合

安裝程式及設定頁面的 Hooks 按鈕會將 Agent Pulse 的全域 hooks 合併至：

```text
Windows:
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json

macOS:
~/.claude/settings.json
~/.codex/hooks.json
```

會感知工作階段開始、使用者提交、工具呼叫前後、權限請求、停止和失敗等事件，並更新 Dashboard、懸浮視窗與硬體燈。修改前會建立時間戳記備份，既有 hooks 會保留，重複安裝不會累積 Agent Pulse hooks。

驗證方式：開啟新的 Claude Code 或 Codex 工作階段，提交一次請求並觸發工具呼叫或權限請求，觀察 Dashboard 的即時事件和狀態顏色。

> Codex Offline Sandbox 可能封鎖本機 loopback 網路；Agent Pulse 會透過本機狀態檔案監聽繼續同步狀態，不依賴該網路通道。

#### Codex hooks 信任與設定

Codex 必須允許執行外部 command hooks，Agent Pulse 才能收到 Codex 事件。首次安裝或 Codex 顯示 hook 安全確認時，請選擇**信任/允許 Agent Pulse hooks**；如果拒絕或未信任，Codex 不會執行這些命令，Dashboard 和實體燈也不會隨 Codex 狀態變化。

設定步驟：

1. 開啟 Agent Pulse Dashboard，點選「設定」。
2. 在設定頁面點選「安裝 Codex Hooks」。
3. 確認 `%USERPROFILE%\.codex\hooks.json` 已包含 Agent Pulse hooks；安裝程序會保留其他現有 hooks。
4. 重新啟動 Codex 或開啟新的工作階段。
5. 當 Codex 彈出 hook 信任/執行確認時，選擇信任或允許。
6. 提交一次請求並觸發工具呼叫，確認 Dashboard 的即時事件中出現 Codex 狀態。

如果狀態未更新，請先確認 Codex 的 hook 信任狀態，再在設定頁面重新安裝對應的 hooks，然後重新啟動 Codex 或開啟新的工作階段。重複安裝不會累積 Agent Pulse hooks；若曾使用舊版本且發現明顯卡頓，可重新安裝一次 hooks 以完成清理遷移。

## 硬體狀態燈

目前 HW v2 / ESP32-C3-next 實體裝置使用**紅、黃、綠三顆獨立 LED**，沒有藍色 LED。實體燈狀態如下：

- **綠燈**：工作完成、工作階段閒置或可檢視結果。
- **黃燈**：正在回應、呼叫工具或處理中。
- **紅燈**：權限請求、工具失敗、受阻或需要人工介入。

恆亮、閃爍和呼吸效果可在設定頁面自由調整。

> Dashboard 的 BLE 圖示可能顯示藍色，它只表示電腦端正在掃描或連線藍牙，**不表示裝置會亮藍燈**。

### 開機、關機與按鍵

- **開機**：關機狀態下長按按鍵約 2 秒，裝置會鎖定供電並啟動。
- **開機回饋**：裝置依序顯示紅 → 黃 → 綠，然後進入預設的綠燈閃爍和 BLE 廣播狀態；如已啟用聲音，會播放啟動提示音。
- **關機**：開機後再次長按約 2 秒，裝置熄燈並關閉供電保持；如已啟用聲音，會播放關機提示音。
- **短按**：顯示約 2 秒電量；尚未連線 BLE 時，同時重新開始/喚醒廣播。
- **升級期間**：OTA 傳輸過程中按鍵操作會被忽略，避免意外中斷升級。

### 實體燈與識別回饋

- **正在廣播、等待連線**：綠燈呼吸。
- **已連線 BLE**：綠燈恆亮，隨後會恢復/接收目前 Agent 狀態。
- **連線中斷**：裝置重新廣播，並回到綠燈呼吸。
- **約 60 秒仍未連線**：停止廣播並紅燈閃爍；短按可再次開始廣播。
- **識別裝置**：從 Dashboard 執行識別時，裝置會快速顯示紅 → 黃 → 綠 → 熄滅，循環數次後恢復原狀態。
- **連線動畫**：裝置會用紅、黃、綠的順序回饋連線過程；連線完成後主機再下發目前工作狀態。

### 電量、充電與聲音

短按裝置會顯示電量約 2 秒：

| 電壓估算 | 燈效 |
|---|---|
| 約 ≥ 4.0 V | 紅、黃、綠全亮 |
| 約 3.7–4.0 V | 紅、黃亮 |
| 約 < 3.7 V | 紅燈亮 |

已連線且裝置支援時，Dashboard 和懸浮視窗會顯示估算電壓、百分比與充電狀態。數值用於日常判斷，不應作為精密電量計。

設定頁面的「聲音」開關控制蜂鳴器提示音；預設關閉。三色亮度和聲音設定會儲存在裝置中，斷電後仍保持。

### 連線方式

#### 藍牙近場連線（建議）

1. 中斷狀態燈的 USB 資料線並開啟裝置；若已停止廣播，短按按鍵重新開始。
2. 開啟 Dashboard。尚未繫結裝置時會持續掃描，直到自動繫結、手動繫結或使用者停止掃描。
3. 將目標狀態燈靠近目前電腦的藍牙介面卡。清單會即時顯示名稱、MAC/裝置識別碼、RSSI 和取樣次數。
4. 自動繫結要求至少 3 次取樣、RSSI 達到 `-45 dBm`，且比第二強候選裝置至少高 `8 dB`。若不同電腦的接收強度差異較大，可直接點擊「繫結」。

繫結成功後會停止掃描並將識別碼儲存在本機設定中；後續只會自動連線這盞燈，不會依訊號強度切換其他裝置。更換狀態燈時請點擊「忘記裝置」，再重新近場或手動繫結。

BLE 圖示藍色代表掃描/連線中、綠色代表近期有效互動、灰色代表未連線、紅色代表錯誤。連線後只同步目前有效狀態，不補送已過期的歷史燈光事件。Windows 通常顯示藍牙 MAC；受 CoreBluetooth 隱私機制限制，macOS 可能顯示系統 UUID，該 UUID 可供本機穩定繫結，但不是硬體 MAC。

找不到裝置時，請確認裝置已開機並在廣播、系統藍牙可用、USB 未連線，且沒有同時執行其他 Agent Pulse/BLE Bridge。macOS 首次使用時還需核准藍牙權限。

#### USB 序列埠連線、選擇與復原

1. 使用支援資料傳輸的 USB 線連接狀態燈；僅充電線不會產生序列埠。
2. 開啟 Dashboard 的「USB 序列埠」面板。Windows 顯示 `COM*`，macOS 通常顯示 `/dev/cu.usbmodem*` 或 `/dev/cu.usbserial*`。
3. 預設「自動選擇」只連線 AgentPulse 免驅裝置 `VID:PID 303A:1001`，不會自動開啟 CH340、CP210x、FTDI 等其他序列埠。
4. 同時連接多台 `303A:1001` 或需要其他相容介面卡時，請依序列埠和 VID/PID 在清單中手動選擇。

手動選擇會儲存在本機設定中。指定序列埠不存在時會保持離線，不會靜默切換其他裝置；選擇「自動選擇」可恢復預設模式。清單會標示預設、已選擇、已連線和未連線狀態，韌體支援時頁面頂部也會顯示電量與充電狀態。

USB 優先於 BLE：USB 連線成功後會暫停 BLE 掃描與連線，USB 中斷後再恢復掃描或連線已繫結裝置。若清單中完全沒有裝置，請檢查資料線、驅動程式和 USB CDC 韌體；USB 也是首次燒錄、分割區遷移或 OTA 失敗後的優先復原方式。

## 懸浮視窗

在 Dashboard 點擊「開啟懸浮燈」或「關閉懸浮燈」。懸浮視窗會顯示目前狀態顏色、專案名稱、BLE 狀態及可用時的裝置電量。

![桌面懸浮視窗（黃燈 = 進行中）](docs/screenshots/floating-window.png)

它由安裝版 daemon 管理；即使懸浮視窗啟動失敗，Dashboard 和狀態同步仍可繼續運作。

## 程式更新

在 Dashboard 的 **AgentPulse 程式更新** 區域：

1. 點擊「檢查程式更新」。
2. 發現新版本後，點擊「確認安裝」。
3. Agent Pulse 下載並驗證已簽署更新清單、安裝程式名稱、大小和 SHA-256。
4. 驗證通過後，Windows 檔案總管會開啟並選取已驗證的安裝程式。
5. 請手動雙擊該 EXE，並在可見的 Inno Setup 精靈中完成安裝。

Agent Pulse 不會靜默執行安裝程式，也不會代您完成安裝精靈。覆蓋安裝時，Inno 精靈會關閉目前安裝目錄下的 Agent Pulse daemon、懸浮視窗和 BLE Bridge，以釋放被占用的檔案；不會對無關程式執行廣泛關閉。

已下載的安裝程式預設快取於：

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## 韌體更新

硬體能力：

- **ESP32-C3-next 可升級韌體**：裝置資訊必須回報以下硬體識別和 OTA 能力，才可使用 Dashboard 的 BLE/USB OTA：

  ```text
  agentpulse-esp32c3-next
  ```

升級前請確認裝置資訊和目標韌體。OTA 只接受 Arduino **應用程式映像** `.ino.bin`；不要上傳 bootloader、分割區表、`merged.bin` 或其他完整首次燒錄檔案。

### 重要限制

目前 OTA 仍是實驗室功能：韌體端尚未實作映像簽名驗證、Secure Boot、Flash Encryption、健康確認和自動回滾。升級時請勿斷電、拔線、關閉 Bluetooth 或結束 daemon；失敗時請優先透過 USB 復原。

***升級建議插上電源，防止升級過程中突然斷電導致升級失敗影響使用。***

舊裝置的 OTA 分割區配置無法透過一般 BLE/USB application OTA 遷移。需要遷移分割區配置或首次燒錄時，必須透過 USB download/bootloader 模式完整燒錄 bootloader、分割區表、OTA boot selector 與 factory app。

## 資料與隱私

預設執行資料儲存在本機：

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

Agent Pulse 預設不會上傳程式碼、提示詞、終端機輸出或專案檔案。專案根目錄中的舊 `.agent-pulse` 僅用於相容/遷移；新版本不再向專案目錄寫入新的執行階段資料。

## 常見問題

### Dashboard 無法開啟

確認存取的是 `http://127.0.0.1:7900`，而不是設定頁面連接埠 `4321`。安裝版可嘗試從開始功能表重新啟動 Agent Pulse；開發人員可在命令列檢查：

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

不要同時執行原始碼 daemon 和安裝版，它們會爭用 `7900`、`47801`、`7950` 與 BLE 裝置。

### Claude Code/Codex 狀態沒有變化

1. 開啟新的 Claude Code/Codex 工作階段。
2. 在設定頁面重新安裝對應 hooks。
3. 確認 `%USERPROFILE%\.claude\settings.json` 或 `%USERPROFILE%\.codex\hooks.json` 仍包含 Agent Pulse 設定。
4. Claude Code 使用者可執行：

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE 無法連線

檢查裝置供電、Windows 藍牙、距離與 Dashboard 狀態；不要手動啟動額外的 BLE Bridge，以免占用 `47801`。

### USB 裝置未找到

使用資料線，檢視裝置管理員的「連接埠（COM 和 LPT）」，必要時選擇明確 COM 連接埠。若沒有 COM 連接埠，請檢查 USB CDC 韌體和驅動程式。

### 通知過於頻繁

在設定頁面關閉完成/錯誤/卡住提醒，或調整「卡住」判定時間。

## 注意事項

- Windows 與 macOS 共用 daemon、Dashboard 和裝置協定；安裝程式、Hooks 腳本、BLE Sidecar、懸浮視窗和程式更新驗證依平台實作，正式使用前應在目標系統與硬體驗證完整流程。
- 多裝置環境使用唯一裝置識別碼持久化繫結；RSSI 只參與首次近場選擇，不會切換已繫結的狀態燈。Windows 通常使用 MAC，macOS 可能使用 CoreBluetooth UUID。
- 程式更新與韌體 OTA 是不同流程：Windows 使用 EXE，macOS 使用符合 CPU 架構且通過簽署、公證驗證的 PKG；韌體 OTA 只寫入相容裝置的應用程式映像。
