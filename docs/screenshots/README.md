# 教程截图清单

README 中的截图统一放在本目录（`docs/screenshots/`），用相对路径引用。

## 状态：部分已截

`README.zh-CN.md` 里已经**预留好图片引用位**（用 HTML 注释包着，不会显示破图）。

### ✅ 已补好（已嵌入 README）
| 文件名 | 内容 | 用在哪里 |
| --- | --- | --- |
| `config-events.png`（1280x900） | 配置页顶部「Agent 集成」区块（Hooks 安装卡片，中文） | 第 1 章 步骤 2 |
| `config-full.png`（1280x7200） | **配置页全貌**：Agent 集成 / 灯光方案 / 声音通知 / 自定义音乐编辑器 / 事件与灯光方案 / **WebHook 通知区块** | 第 6.2 节 |
| `floating-window.png` | 桌面悬浮窗（黄灯 solid = 进行中，左下角蓝牙已连接、电池图标） | 第 6.3 节 |

> 这两张已覆盖「Hooks 安装」「音乐编辑器」「Webhook 区块」三个教程要点，
> 因此下面清单中对应项可视为**已满足**（单独聚焦版可后续补）。
> `floating-window.png` 为人工真截图（多屏环境下脚本按 `winfo_screenwidth` 计算的坐标不可靠，故手动截取）。

### ⏸ 待补（README 暂以 Mermaid 图或全貌图替代）
| 文件名 | 截什么 | 用在章节 | 状态 |
| --- | --- | --- | --- |
| `dashboard.png` | Dashboard 首页（实时事件面板） | 6.1 Dashboard | 待补（SSE 导致截图失败，见下） |
| `lamp-breathing.png` | 实体灯的绿灯呼吸状态（等待连接） | 5.2 灯效反馈速查表 | 待补（需拍摄实体灯） |
| `config-events-section.png` | 单独截取「事件与灯光方案」区块 | 6.2 配置页 | 可选（全貌图已含） |
| `webhook-channels.png` | 单独截取 Webhook 通知区块 | 8.2 添加通知通道 | 可选（全貌图已含） |

### ⚠️ Dashboard 截图说明
Dashboard 使用 SSE 长连接推送实时事件，导致 Edge `--screenshot` 的 `load`
事件永不触发，截图始终失败。README 第 6.1 节已用 Mermaid 图代替。如需 Dashboard
真截图，建议：
- 用 Chrome DevTools 的 Capture full size screenshot 功能（手动操作）；
- 或改造 daemon 让 Dashboard 支持暂停 SSE 后截图。

## 截图建议

- **界面截图**：窗口宽度建议 1280-1500px
- **实体灯**：在光线较暗的环境拍摄，突出灯效
- **敏感信息**：Webhook 截图请**遮掉** URL 中的密钥部分（如 `?key=xxx`）与签名密钥
- **命名**：全部小写 + 连字符，与 README 中的引用保持一致
- **路径陷阱**：Edge `--screenshot=<path>` 不接受路径中含空格；先截到无空格临时目录再 `Copy-Item` 到目标

## 为什么不能用 AI 生成的界面图

AI 画的界面图与实际 UI 必然有出入（按钮位置、文案、配色），放进教程会误导用户
照着找却找不到。这里只放**真实截图**。
