# Agent Pulse

Agent Pulse mirrors the working state of Claude Code, Codex, WorkBuddy, CodeBuddy, and remote Ubuntu Collector sessions to a local Windows or macOS Dashboard, floating widget, and physical ESP32 three-color status light, so you can follow multiple AI coding agents without watching terminals constantly.

Languages: English | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Español](README.es.md)

## Current release

The current source version is `0.4.1` (2026-08-14), with ESP32 firmware still pinned to `0.1.21+22`. This release focuses on WorkBuddy and CodeBuddy hooks plus Ubuntu server multi-agent collection/display. A single light keeps the simple default behavior and follows the latest task. When multiple lights are connected, each light can independently follow the latest task, a selected project, or a selected agent such as Claude Code, Codex, WorkBuddy, or CodeBuddy, and can use its own light profile.

This release also keeps every connected BLE or USB device visible with its connection and battery state, prevents duplicate BLE bridge processes, and labels the floating widget with the agent and project for the latest task. Ubuntu Collector can publish remote agent status through MQTT for display in the Windows Dashboard alongside local agents. Bluetooth retains both proximity auto-connect and Windows system-paired connection. Application and firmware updates prefer Gitee and automatically fall back to GitHub. See the [release notes](CHANGELOG.md) for details.

## Status meanings

| Color | Typical meaning |
|---|---|
| Green | The session is idle, a task is complete, or results are ready to review |
| Yellow | The agent is responding, calling tools, continuing work after a tool completes, or needs more information |
| Red | A permission request, tool failure, block, or other situation requires human attention |

Status is stored and displayed per project directory. Multiple projects on the same computer can appear in the Dashboard at the same time.

## Installation

### Recommended: Windows installer

Download and run `AgentPulseSetup-<version>.exe`. End users do not need to install Node.js, npm, Python, BLE Bridge, PyInstaller, or Arduino tools separately.

Official download links:

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

For the current Windows user, the installer:

- installs the Agent Pulse daemon, bundled Node runtime, BLE Bridge, and floating widget;
- safely merges Claude Code, Codex, WorkBuddy, and CodeBuddy hooks without replacing existing hooks;
- configures startup on sign-in;
- starts Agent Pulse and opens the Dashboard after installation.

The default installation directory is usually:

```text
%LOCALAPPDATA%\AgentPulse
```

After installation, restart Claude Code, Codex, WorkBuddy, or CodeBuddy, or open a new session so hooks are reloaded.

### macOS package

macOS 12 or later uses architecture-specific `x86_64` or `arm64` PKG files. Production packages are signed with Developer ID, notarized by Apple, and stapled. Do not disable Gatekeeper or SIP; the user must approve administrator, Bluetooth, and notification prompts in the visible macOS UI.

Version `0.3.0` publishes signed and notarized `x86_64` and `arm64` packages in the production update catalog, including the new device controls and Gitee-first update fallback. See the [macOS installation guide](macos-install/RELEASE_INSTALL.md).

### Install from source or the command line

This is a developer path, not a requirement for Windows-installer users. See the [Developer appendix](#developer-appendix).

## Everyday use

### Dashboard

Open:

```text
http://127.0.0.1:7900
```

You can also use **Open Dashboard** from the Start menu. The Dashboard is the main daily control surface. It shows:

- current projects, light colors, and live events;
- BLE connection state and device battery details when supported;
- floating-widget show/hide controls;
- application and firmware updates;
- a link to configuration.

The Dashboard listens only on the local loopback address and is not exposed to the LAN.

### Configuration page

Click **Configuration** in the Dashboard to open the configuration page. Its default URL is:

```text
http://127.0.0.1:4321/?lang=zh
```

You can adjust notifications, the stuck-task threshold, colors/effects/brightness for each event, and sound settings. The **Install Claude Code Hooks**, **Install Codex Hooks**, **Install WorkBuddy Hooks**, and **Install CodeBuddy Hooks** buttons detect Windows or macOS and run the matching platform installer. `7900` is the Dashboard; `4321` is a separate configuration page.

### Claude Code, Codex, WorkBuddy, and CodeBuddy integration

The installer and the configuration-page hook buttons merge Agent Pulse global hooks into:

```text
Windows:
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
%USERPROFILE%\.workbuddy\settings.json
%USERPROFILE%\.codebuddy\settings.json

macOS:
~/.claude/settings.json
~/.codex/hooks.json
~/.workbuddy/settings.json
~/.codebuddy/settings.json
```

It observes session starts, user prompts, tool calls, permission requests, stops, and failures, then updates the Dashboard, floating widget, and physical status light. A timestamped backup is created before changes, existing hooks are preserved, and repeated installation does not accumulate Agent Pulse entries.

To verify the integration, open a new Claude Code, Codex, WorkBuddy, or CodeBuddy session, submit a prompt, and trigger a tool call or permission request. Watch the Dashboard's live events and status color.

> Codex Offline Sandbox can block local loopback networking. Agent Pulse continues to synchronize through local status-file watching and does not depend on that network channel.

#### Codex hook trust and configuration

Codex must be allowed to run external command hooks for Agent Pulse to receive Codex events. During the initial installation, or when Codex shows a hook security confirmation, choose to **trust/allow Agent Pulse hooks**. If you decline or do not trust them, Codex will not run these commands, and the Dashboard and physical light will not change with Codex status.

Configuration steps:

1. Open the Agent Pulse Dashboard and click "Configuration."
2. On the configuration page, click "Install Codex Hooks."
3. Confirm that `%USERPROFILE%\.codex\hooks.json` on Windows or `~/.codex/hooks.json` on macOS contains Agent Pulse hooks; installation preserves any other existing hooks.
4. Restart Codex or open a new session.
5. When Codex displays a hook trust/execution confirmation, choose Trust or Allow.
6. Submit a request and trigger a tool call, then confirm that Codex status appears in the Dashboard's live events.

If status does not update, first check Codex's hook trust status, then reinstall the relevant hooks from the configuration page and restart Codex or open a new session. Reinstalling does not accumulate Agent Pulse hooks. If you used an older version and notice significant lag, reinstall the hooks once to complete the cleanup migration.

## Physical status light

Current HW v2 / ESP32-C3-next devices use **three separate physical LEDs: red, yellow, and green**. There is no physical blue LED. The light states are:

- **Green:** a task is complete, the session is idle, or results are ready to review.
- **Yellow:** the agent is responding, calling tools, or processing work.
- **Red:** a permission request, tool failure, block, or other human intervention is required.

Solid, blinking, and breathing effects can be adjusted on the configuration page.

> The Dashboard BLE icon can be blue. It means the desktop software is scanning or connecting over Bluetooth; **it does not mean the device has or will illuminate a blue LED**.

### Power, shutdown, and button operation

- **Power on:** when off, hold the button for about two seconds. The device latches its power supply and starts.
- **Power-on feedback:** the device displays red → yellow → green, then enters its default blinking-green BLE advertising state. If sound is enabled, it plays a startup tone.
- **Power off:** while on, hold the button again for about two seconds. The LEDs turn off and the power latch is released. If sound is enabled, it plays a shutdown tone.
- **Short press:** shows battery level for about two seconds. If BLE is not connected, it also starts or wakes advertising.
- **During updates:** button operations are ignored while OTA transfer is active to prevent accidental interruption.

### Physical light and identification feedback

- **Advertising and waiting for a connection:** green breathing.
- **BLE connected:** solid green, then the current Agent Pulse state is restored or received.
- **Connection lost:** the device starts advertising again and returns to green breathing.
- **No connection for about 60 seconds:** advertising stops and the red LED blinks. Short-press the button to advertise again.
- **Identify device:** when identification is started from the Dashboard, the device quickly displays red → yellow → green → off, repeats the sequence several times, and then restores its previous state.
- **Connection animation:** the device uses a red, yellow, green sequence to acknowledge the connection process. The host sends the current work state afterward.

### Battery, charging, and sound

A short press displays battery level for about two seconds:

| Estimated voltage | Light effect |
|---|---|
| About ≥ 4.0 V | Red, yellow, and green all on |
| About 3.7–4.0 V | Red and yellow on |
| About < 3.7 V | Red on |

When connected and supported by the device, the Dashboard and floating widget show estimated voltage, percentage, and charging state. These values are for day-to-day guidance and are not precision battery measurements.

The **Sound** switch on the configuration page controls buzzer notifications and is off by default. Three-color brightness and sound settings are stored on the device and survive power loss.

### Connection methods

#### Proximity BLE connection (recommended)

Use this flow when binding a light for the first time:

1. Disconnect the light's USB data cable and power on the device. If advertising has stopped, short-press the button to start it again.
2. Open the Dashboard. When no device is bound, Agent Pulse scans continuously until it binds automatically, you bind manually, or you click **Stop scanning**.
3. Move the intended light close to this computer's Bluetooth adapter. The device list shows its name, MAC/device identifier, RSSI, and sample count in real time.
4. Automatic binding requires at least 3 samples, an RSSI of `-45 dBm` or stronger, and an `8 dB` lead over the second-strongest candidate. These checks reduce cross-connection risk when several computers and lights share one room.
5. Bluetooth receivers differ. If the automatic criteria are not met, select the intended row and click **Bind**.

After binding, scanning stops and the identifier is stored in the local configuration. Future launches connect only to that light and do not switch devices based on signal strength. To replace it, click **Forget device**, then repeat proximity binding or bind a scan result manually.

The Dashboard BLE icon normally means: blue for scanning/connecting, green for recent valid device interaction, gray for disconnected, and red for a connection error. After connecting, Agent Pulse synchronizes only the current valid state and does not replay expired historical light events. Blue is only a software-icon state, not a physical LED.

Windows normally exposes the Bluetooth MAC address. Because of Apple CoreBluetooth privacy behavior, macOS may show an OS-assigned UUID in the same field. That UUID can still provide stable local binding, but it is not the hardware MAC printed on the device or observed by another computer.

If no device appears, verify that it is powered on and advertising, system Bluetooth is available, USB is disconnected, and no additional Agent Pulse/BLE Bridge instance is running. On macOS, approve Bluetooth access in the visible system prompt.

#### USB serial connection, selection, and recovery

USB provides wired light control, device-information and battery reads, diagnostics, recovery, and firmware updates for compatible devices:

1. Connect the light with a data-capable USB cable. A charge-only cable does not create a serial port.
2. Open the Dashboard's **USB serial** panel. Windows uses `COM*`; macOS normally uses `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`.
3. The default **Automatic selection** mode connects only to the driverless AgentPulse USB device with `VID:PID 303A:1001`. It does not automatically open CH340, CP210x, FTDI, or other serial adapters.
4. If several `303A:1001` devices are attached, or another compatible adapter is required, inspect the port and VID/PID in the list and click **Select** beside the intended port.
5. A manual selection is stored locally. If that port disappears, Agent Pulse remains offline instead of silently opening another serial device. Select **Automatic selection** to return to the default behavior.

The list marks default, selected, connected, and missing ports. Its footer can connect/disconnect USB or refresh enumeration. After connection, the current port appears in the page header; battery and charging details also appear when supported by the firmware.

USB takes priority over BLE. A successful USB connection pauses BLE scanning and connection so one light is not controlled over two transports. Disconnecting USB resumes scanning or reconnection to the bound BLE device.

If the device does not appear in the list, check the cable, operating-system serial devices, drivers, and whether the firmware enables USB CDC. USB is the preferred recovery path for first flashing, partition migration, or a failed OTA update.

## Floating widget

Click **Show floating light** or **Hide floating light** in the Dashboard. The widget shows the current status color, project name, BLE state, and device battery information when available.

![Desktop floating widget (yellow = in progress)](docs/screenshots/floating-window.png)

The installed daemon manages the widget. If the widget cannot start, the Dashboard and status synchronization can still continue.

## Application updates

In the **AgentPulse application update** area of the Dashboard:

1. Click **Check for application updates**.
2. If a new version is found, click **Confirm installation**.
3. Agent Pulse downloads and validates the signed update manifest, installer name, size, and SHA-256.
4. After validation, Windows Explorer opens with the verified installer selected.
5. Manually double-click that EXE and complete the visible Inno Setup wizard.

Agent Pulse does not silently run the installer or complete the installation wizard for you. During an in-place update, the Inno wizard closes the Agent Pulse daemon, floating widget, and BLE Bridge within the current installation directory to release locked files. It does not broadly close unrelated applications.

Downloaded installers are cached by default in:

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## Firmware updates

Hardware capabilities:

- **ESP32-C3-next updatable firmware:** it must report the following hardware ID and OTA capability before Dashboard BLE/USB OTA can be used:

  ```text
  agentpulse-esp32c3-next
  ```

Confirm device information and the target firmware before updating. OTA accepts only an Arduino **application image** (`.ino.bin`). Do not upload a bootloader, partition table, `merged.bin`, or any other complete first-flash image.

### Important limitations

OTA is currently a laboratory feature. Firmware does not yet implement image-signature verification, Secure Boot, Flash Encryption, health confirmation, or automatic rollback. Do not remove power or the cable, disable Bluetooth, or exit the daemon during an update. Prefer USB recovery if an update fails.

For safer updates, connect external power so an unexpected power loss does not interrupt the upgrade.

Older devices cannot migrate their OTA partition layout through normal BLE/USB application OTA. Partition-layout migration and first flashing require complete flashing in USB download/bootloader mode: bootloader, partition table, OTA boot selector, and factory app.

## Data and privacy

Runtime data is stored locally by default:

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

By default, Agent Pulse does not upload source code, prompts, terminal output, or project files. The legacy `.agent-pulse` directory in a project root is used only for compatibility/migration; current versions do not write new runtime data there.

## Troubleshooting

### Dashboard does not open

Confirm that you are visiting `http://127.0.0.1:7900`, not configuration port `4321`. On Windows, try restarting Agent Pulse from the Start menu. On macOS, check the user LaunchAgent:

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

```bash
launchctl print "gui/$(id -u)/com.agentpulse.daemon"
```

Do not run the source daemon and installed version at the same time. They compete for `7900`, `47801`, `7950`, and the BLE device.

### Claude Code/Codex status does not change

1. Open a new Claude Code/Codex session.
2. Reinstall the relevant hooks from the configuration page.
3. Confirm that `~/.claude/settings.json` or `~/.codex/hooks.json` still includes the Agent Pulse configuration (`~` maps to `%USERPROFILE%` on Windows).
4. Claude Code users can run:

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE does not connect

Check device power, system Bluetooth, distance, and Dashboard state. On macOS, approve Bluetooth access in the visible system prompt. Do not manually start an additional BLE Bridge because it can occupy `47801`.

### USB device is not found

Use a data cable. On Windows, check **Ports (COM & LPT)** in Device Manager. On macOS, check `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`. If no serial device appears, check the cable, USB CDC firmware, and drivers.

### Notifications are too frequent

Disable completion/error/stuck notifications or adjust the stuck-task threshold in the configuration page.

## Notes

- Windows and macOS share the daemon, Dashboard, and device protocol. Installers, hook scripts, BLE sidecars, widgets, and desktop-update validation remain platform-specific and must be verified on the target OS and hardware.
- Multi-device environments use a persistent unique device identifier. RSSI participates only in first-time proximity selection and never switches an already bound light. The identifier is normally a MAC address on Windows and may be a CoreBluetooth UUID on macOS.
- Application updates and firmware OTA are separate: Windows desktop updates use an EXE, macOS updates use an architecture-matched signed and notarized PKG, and firmware OTA writes only a compatible device application image.
