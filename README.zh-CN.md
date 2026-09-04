# Agent Pulse

Agent Pulse 将 Claude Code 与 Codex 的工作状态同步到 Windows 本机 Dashboard、悬浮窗和 ESP32 实体三色灯，帮助你在不盯着终端时也能了解 AI 编程会话的进度。

语言：[English](README.md) | 简体中文 | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Español](README.es.md)

## 状态含义

| 颜色 | 常见含义 |
|---|---|
| 绿 | 会话空闲、任务结束或可查看结果 |
| 黄 | 正在响应、调用工具、工具完成后继续处理或需要补充信息 |
| 红 | 权限请求、工具失败、阻塞或需要人工介入 |

状态按项目目录保存和展示；同一台电脑上的多个项目可同时出现在 Dashboard 中。

## 安装

### 推荐：Windows 安装包

下载并运行 `AgentPulseSetup-<版本>.exe`。普通用户不需要自行安装 Node.js、npm、Python、BLE Bridge、PyInstaller 或 Arduino 工具。

官方下载地址：

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

安装程序会为当前 Windows 用户：

- 安装 Agent Pulse daemon、内置 Node runtime、BLE Bridge 与悬浮窗；
- 安全合并 Claude Code hooks 和 Codex hooks，不覆盖已有的其他 hooks；
- 设置登录后自动启动；
- 完成后启动 Agent Pulse 并打开 Dashboard。

默认安装位置通常是：

```text
%LOCALAPPDATA%\AgentPulse
```

安装完成后，请重新启动 Claude Code/Codex 或新开一个会话，让 hooks 重新加载。

### 从源码/命令行安装

这是开发者路径，不是 Windows 安装包用户的必需步骤。请参见文末的[开发者附录](#开发者附录)。

## 日常使用

### Dashboard

打开：

```text
http://127.0.0.1:7900
```

也可以从开始菜单的 **Open Dashboard** 打开。Dashboard 是日常操作入口，可查看：

- 当前项目、灯色和实时事件；
- BLE 连接状态、设备电量（设备支持时）；
- 悬浮窗显示/隐藏；
- 程序与固件更新；
- 配置入口。

Dashboard 只监听本机回环地址，不会公开到局域网。

### 配置页

在 Dashboard 点击“配置”打开配置页。配置页默认地址为：

```text
http://127.0.0.1:4321/?lang=zh
```

可调整通知、卡住判定时间、各类事件的颜色/闪烁/呼吸效果、亮度和声音等。`7900` 是 Dashboard，`4321` 是独立配置页，两者用途不同。

### Claude Code 与 Codex 集成

安装器会把 Agent Pulse 的全局 hooks 合并到：

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

会感知会话开始、用户提交、工具调用前后、权限请求、停止和失败等事件，并更新 Dashboard、悬浮窗与硬件灯。你的其他 hooks 和设置会保留。

验证方式：新开 Claude Code 或 Codex 会话，提交一次请求并触发工具调用或权限请求，观察 Dashboard 的实时事件和状态颜色。

> Codex Offline Sandbox 可能阻断本机 loopback 网络；Agent Pulse 会通过本地状态文件监听继续同步状态，不依赖该网络通道。

#### Codex hooks 信任与配置

Codex 必须允许执行外部 command hooks，Agent Pulse 才能收到 Codex 事件。首次安装或 Codex 提示 hook 安全确认时，请选择**信任/允许 Agent Pulse hooks**；如果拒绝或未信任，Codex 不会执行这些命令，Dashboard 和实体灯也不会随 Codex 状态变化。

配置步骤：

1. 打开 Agent Pulse Dashboard，点击“配置”。
2. 在配置页点击“安装 Codex Hooks”。
3. 确认 `%USERPROFILE%\.codex\hooks.json` 已包含 Agent Pulse hooks；安装过程会保留其他已有 hooks。
4. 重启 Codex 或新开一个会话。
5. 当 Codex 弹出 hook 信任/执行确认时，选择信任或允许。
6. 提交一次请求并触发工具调用，确认 Dashboard 的实时事件出现 Codex 状态。

如果状态不更新，请先确认 Codex 的 hook 信任状态，再在配置页重新安装对应的 hooks，然后重启 Codex 或新开会话。重复安装不会累积 Agent Pulse hooks；如曾使用旧版本且发现明显卡顿，可重新安装一次 hooks 以完成清理迁移。

## 硬件状态灯

当前 HW v2 / ESP32-C3-next 实体设备使用**红、黄、绿三颗独立 LED**，没有蓝色 LED。实体灯状态如下：

- **绿灯**：任务完成、会话空闲或可查看结果。
- **黄灯**：正在响应、调用工具或处理中。
- **红灯**：权限请求、工具失败、阻塞或需要人工介入。

常亮、闪烁和呼吸效果可在配置页自由调整。

> Dashboard 的 BLE 图标可能显示蓝色，它只表示电脑端正在扫描或连接蓝牙，**不表示设备会亮蓝灯**。

### 开机、关机与按键

- **开机**：关机状态下长按按键约 2 秒，设备会锁定供电并启动。
- **开机反馈**：设备依次显示红 → 黄 → 绿，然后进入默认的绿灯闪烁和 BLE 广播状态；如已启用声音，会播放启动提示音。
- **关机**：开机后再次长按约 2 秒，设备熄灯并关闭供电保持；如已启用声音，会播放关机提示音。
- **短按**：显示约 2 秒电量；尚未连接 BLE 时，同时重新开始/唤醒广播。
- **升级期间**：OTA 传输过程中按键操作会被忽略，避免意外中断升级。

### 实体灯与识别反馈

- **正在广播、等待连接**：绿灯呼吸。
- **已连接 BLE**：绿灯常亮，随后会恢复/接收当前 Agent 状态。
- **连接断开**：设备重新广播，并回到绿灯呼吸。
- **约 60 秒仍未连接**：停止广播并红灯闪烁；短按可再次开始广播。
- **识别设备**：从 Dashboard 执行识别时，设备会快速显示红 → 黄 → 绿 → 熄灭，循环数次后恢复原状态。
- **连接动画**：设备会用红、黄、绿的顺序反馈连接过程；连接完成后主机再下发当前工作状态。

### 电量、充电与声音

短按设备会显示电量约 2 秒：

| 电压估算 | 灯效 |
|---|---|
| 约 ≥ 4.0 V | 红、黄、绿全亮 |
| 约 3.7–4.0 V | 红、黄亮 |
| 约 < 3.7 V | 红灯亮 |

已连接且设备支持时，Dashboard 和悬浮窗会显示估算电压、百分比与充电状态。数值用于日常判断，不应作为精密电量计。

配置页的“声音”开关控制蜂鸣器提示音；默认关闭。三色亮度和声音设置会保存在设备中，断电后仍保持。

### 连接方式

#### BLE 连接（推荐）

1. 给设备上电；若长时间未连接，短按一次让它重新广播。
2. 打开 Dashboard，等待 BLE 状态从扫描/连接中变为已连接。
3. 连接成功后，Agent Pulse 会自动把当前状态同步到实体灯。

Dashboard BLE 图标一般表示：蓝色为扫描/连接中，绿色为近期收到有效设备交互，灰色为未连接，红色为连接错误。蓝色仅是软件图标状态，不是实体 LED。

找不到设备时，请确认设备已上电并在广播、Windows 蓝牙可用、设备距离足够近，并避免同时运行多个 Agent Pulse/BLE Bridge 实例。

#### USB 连接、诊断与恢复

USB 可用于有线灯控、读取设备信息/电量、诊断、恢复，以及兼容设备的固件升级。请使用数据线而非仅充电线，并在设备管理器确认出现 COM 端口。

当前版本按 USB 串口的厂商标识筛选候选设备；如果连接了多个 ESP32 或常见 USB 串口设备，请在命令行明确指定目标端口，例如：

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

不要把无关的串口设备作为 Agent Pulse 灯控目标。后续版本会改为先向候选串口发送 `deviceInfo` 请求，只有收到合法设备响应后才自动连接。

如果设备完全没有串口，请检查线材、驱动以及固件是否启用了 USB CDC。USB 是首次烧录、分区迁移或 OTA 失败后的优先恢复方式。

## 悬浮窗

在 Dashboard 点击“打开悬浮灯”或“关闭悬浮灯”。悬浮窗会显示当前状态颜色、项目名、BLE 状态及可用时的设备电量。

![桌面悬浮窗（黄灯 = 进行中）](docs/screenshots/floating-window.png)

它由安装版 daemon 管理；即使悬浮窗启动失败，Dashboard 和状态同步仍可继续工作。

## 程序更新

在 Dashboard 的 **AgentPulse 程序更新** 区域：

1. 点击“检查程序更新”。
2. 发现新版本后，点击“确认安装”。
3. Agent Pulse 下载并校验已签名更新清单、安装包名称、大小和 SHA-256。
4. 校验通过后，Windows 资源管理器会打开并选中已验证的安装包。
5. 请手动双击该 EXE，并在可见的 Inno Setup 向导中完成安装。

Agent Pulse 不会静默运行安装包，也不会替你完成安装向导。覆盖安装时，Inno 向导会关闭当前安装目录下的 Agent Pulse daemon、悬浮窗和 BLE Bridge，以释放被占用的文件；不会针对无关程序执行广泛关闭。

已下载的安装包默认缓存于：

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## 固件更新

硬件能力：

- **ESP32-C3-next 可升级固件**：设备信息必须上报以下硬件标识和 OTA 能力，才可使用 Dashboard 的 BLE/USB OTA：

  ```text
  agentpulse-esp32c3-next
  ```

升级前请确认设备信息和目标固件。OTA 只接受 Arduino **应用镜像** `.ino.bin`；不要上传 bootloader、分区表、`merged.bin` 或其他完整首刷文件。

### 重要限制

当前 OTA 仍是实验室功能：固件端尚未实现镜像签名验证、Secure Boot、Flash Encryption、健康确认和自动回滚。升级时请勿断电、拔线、关闭 Bluetooth 或退出 daemon；失败时请优先通过 USB 恢复。

***升级建议插上电源，防止升级过程中突然断电导致升级失败影响使用。***

旧设备的 OTA 分区布局不能通过普通 BLE/USB application OTA 迁移。需要迁移分区布局或首次烧录时，必须通过 USB download/bootloader 模式完整烧录 bootloader、分区表、OTA boot selector 与 factory app。

## 数据与隐私

默认运行数据保存在本机：

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

Agent Pulse 默认不上传代码、提示词、终端输出或项目文件。项目根目录中的旧 `.agent-pulse` 仅用于兼容/迁移；新版本不再向项目目录写入新的运行时数据。

## 常见问题

### Dashboard 无法打开

确认访问的是 `http://127.0.0.1:7900`，而不是配置页端口 `4321`。安装版可尝试从开始菜单重新启动 Agent Pulse；开发者可在命令行检查：

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

不要同时运行源码 daemon 和安装版，它们会争用 `7900`、`47801`、`7950` 与 BLE 设备。

### Claude Code/Codex 状态没有变化

1. 新开 Claude Code/Codex 会话。
2. 在配置页重新安装对应 hooks。
3. 确认 `%USERPROFILE%\.claude\settings.json` 或 `%USERPROFILE%\.codex\hooks.json` 仍包含 Agent Pulse 配置。
4. Claude Code 用户可运行：

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE 无法连接

检查设备上电、Windows 蓝牙、距离与 Dashboard 状态；不要手动启动额外的 BLE Bridge，以免占用 `47801`。

### USB 设备未找到

使用数据线，查看设备管理器的“端口（COM 和 LPT）”，必要时选择明确 COM 端口。若没有 COM 端口，请检查 USB CDC 固件和驱动。

### 通知过于频繁

在配置页关闭完成/错误/卡住提醒，或调整“卡住”判定时间。

## 注意事项

- 本文面向 Windows 安装包用户。生产使用前应先在目标电脑和硬件上验证 Claude/Codex hooks、BLE、悬浮窗和更新流程。
- 多台 Agent Pulse 设备目前不应仅靠相同 BLE 名称自动区分；未来多设备场景应使用唯一 `deviceId` 绑定，RSSI 只适合作为首次发现时的排序依据。
- 程序更新与固件 OTA 是不同流程：程序更新安装 Windows EXE；固件 OTA 只写入兼容设备的应用镜像。


