# Agent Pulse

Agent Pulse mirrors the working state of Claude Code and Codex to a local Windows Dashboard, floating widget, and physical ESP32 three-color status light, so you can follow an AI coding session without watching the terminal constantly.

Languages: English | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Español](README.es.md)

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

For the current Windows user, the installer:

- installs the Agent Pulse daemon, bundled Node runtime, BLE Bridge, and floating widget;
- safely merges Claude Code and Codex hooks without replacing existing hooks;
- configures startup on sign-in;
- starts Agent Pulse and opens the Dashboard after installation.

The default installation directory is usually:

```text
%LOCALAPPDATA%\AgentPulse
```

After installation, restart Claude Code/Codex or open a new session so hooks are reloaded.

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

You can adjust notifications, the stuck-task threshold, colors/effects/brightness for each event, and sound settings. `7900` is the Dashboard; `4321` is a separate configuration page.

### Claude Code and Codex integration

The installer merges Agent Pulse global hooks into:

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

It observes session starts, user prompts, tool calls, permission requests, stops, and failures, then updates the Dashboard, floating widget, and physical status light. Your other hooks and settings are preserved.

To verify the integration, open a new Claude Code or Codex session, submit a prompt, and trigger a tool call or permission request. Watch the Dashboard's live events and status color.

> Codex Offline Sandbox can block local loopback networking. Agent Pulse continues to synchronize through local status-file watching and does not depend on that network channel.

If status does not update, reinstall the relevant hooks from the configuration page, then restart Claude Code/Codex or open a new session.

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

#### BLE connection (recommended)

1. Power on the device. If it has been disconnected for a long time, short-press the button to advertise again.
2. Open the Dashboard and wait for the BLE state to change from scanning/connecting to connected.
3. After connection, Agent Pulse automatically sends the current state to the physical light.

The Dashboard BLE icon normally means: blue for scanning/connecting, green for recent valid device interaction, gray for disconnected, and red for a connection error. Blue is only a software-icon state, not a physical LED.

If the device cannot be found, make sure it is powered on and advertising, Windows Bluetooth is available, the device is close enough, and no additional Agent Pulse/BLE Bridge instance is running.

#### USB connection, diagnostics, and recovery

USB can provide wired light control, device-information/battery reads, diagnostics, recovery, and firmware updates for compatible devices. Use a data cable rather than a charge-only cable, and confirm that Windows shows a COM port in Device Manager.

The current version filters candidate ports by USB-serial vendor ID. If several ESP32 or common USB-serial devices are connected, explicitly select the target port from the command line, for example:

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

Do not use an unrelated serial device as an Agent Pulse light-control target. A future version will send a `deviceInfo` request to candidate ports and connect automatically only after receiving a valid device response.

If no serial port appears at all, check the cable, driver, and whether the firmware enables USB CDC. USB is the preferred recovery path for first flashing, partition migration, or a failed OTA update.

## Floating widget

Click **Show floating light** or **Hide floating light** in the Dashboard. The widget shows the current status color, project name, BLE state, and device battery information when available.

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

Confirm that you are visiting `http://127.0.0.1:7900`, not configuration port `4321`. With the installed version, try restarting Agent Pulse from the Start menu. Developers can check from a terminal:

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

Do not run the source daemon and installed version at the same time. They compete for `7900`, `47801`, `7950`, and the BLE device.

### Claude Code/Codex status does not change

1. Open a new Claude Code/Codex session.
2. Reinstall the relevant hooks from the configuration page.
3. Confirm that `%USERPROFILE%\.claude\settings.json` or `%USERPROFILE%\.codex\hooks.json` still includes the Agent Pulse configuration.
4. Claude Code users can run:

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE does not connect

Check device power, Windows Bluetooth, distance, and Dashboard state. Do not manually start an additional BLE Bridge because it can occupy `47801`.

### USB device is not found

Use a data cable, check **Ports (COM & LPT)** in Device Manager, and explicitly choose the COM port if needed. If no COM port appears, check USB CDC firmware and drivers.

### Notifications are too frequent

Disable completion/error/stuck notifications or adjust the stuck-task threshold in the configuration page.

## Notes

- This document is for Windows-installer users. Before production use, verify Claude/Codex hooks, BLE, the floating widget, and update flow on the target computer and hardware.
- Multiple Agent Pulse devices should not be distinguished solely by the same BLE name. A future multi-device setup should bind a unique `deviceId`; RSSI is appropriate only for ordering devices during first discovery.
- Application updates and firmware OTA are different processes: application updates install a Windows EXE, while firmware OTA writes only a compatible device application image.
