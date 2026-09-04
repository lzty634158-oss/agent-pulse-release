# Agent Pulse

Agent Pulse는 Claude Code와 Codex의 작업 상태를 Windows 로컬 Dashboard, 플로팅 창, ESP32 물리 3색 상태등에 동기화하여, 터미널을 계속 보지 않아도 AI 코딩 세션의 진행 상황을 파악할 수 있게 해 줍니다.

언어: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | 한국어 | [Français](README.fr.md) | [Deutsch](README.de.md) | [Español](README.es.md)

## 상태 의미

| 색상 | 일반적인 의미 |
|---|---|
| 녹색 | 세션 유휴 상태, 작업 완료 또는 결과 확인 가능 |
| 노란색 | 응답 중, 도구 호출 중, 도구 완료 후 계속 처리 중 또는 추가 정보 필요 |
| 빨간색 | 권한 요청, 도구 실패, 차단됨 또는 사람의 개입 필요 |

상태는 프로젝트 디렉터리별로 저장되고 표시됩니다. 한 대의 컴퓨터에서 여러 프로젝트가 Dashboard에 동시에 나타날 수 있습니다.

## 설치

### 권장: Windows 설치 패키지

`AgentPulseSetup-<版本>.exe`를 다운로드하여 실행합니다. 일반 사용자는 Node.js, npm, Python, BLE Bridge, PyInstaller 또는 Arduino 도구를 직접 설치할 필요가 없습니다.

공식 다운로드 링크:

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

설치 프로그램은 현재 Windows 사용자에 대해 다음을 수행합니다.

- Agent Pulse daemon, 내장 Node runtime, BLE Bridge 및 플로팅 창을 설치합니다.
- Claude Code hooks와 Codex hooks를 안전하게 병합하며, 기존의 다른 hooks를 덮어쓰지 않습니다.
- 로그인 후 자동 시작을 설정합니다.
- 완료 후 Agent Pulse를 시작하고 Dashboard를 엽니다.

기본 설치 위치는 일반적으로 다음과 같습니다.

```text
%LOCALAPPDATA%\AgentPulse
```

설치 후에는 Claude Code/Codex를 다시 시작하거나 새 세션을 열어 hooks가 다시 로드되도록 하세요.

### 소스/명령줄에서 설치

이것은 개발자 경로이며 Windows 설치 패키지 사용자에게 필요한 단계는 아닙니다. 문서 끝의 [개발자 부록](#개발자부록)을 참조하세요.

## 일상 사용

### Dashboard

열기:

```text
http://127.0.0.1:7900
```

시작 메뉴의 **Open Dashboard**에서도 열 수 있습니다. Dashboard는 일상 작업의 진입점이며, 다음을 확인할 수 있습니다.

- 현재 프로젝트, 표시등 색상 및 실시간 이벤트
- BLE 연결 상태, 장치 배터리 잔량(장치가 지원하는 경우)
- 플로팅 창 표시/숨기기
- 프로그램 및 펌웨어 업데이트
- 구성 진입점

Dashboard는 로컬 loopback 주소에서만 수신 대기하며 LAN에 공개되지 않습니다.

### 구성 페이지

Dashboard에서 “구성”을 클릭하여 구성 페이지를 엽니다. 구성 페이지의 기본 주소는 다음과 같습니다.

```text
http://127.0.0.1:4321/?lang=zh
```

알림, 멈춤 판정 시간, 이벤트별 색상/점멸/호흡 효과, 밝기 및 소리 등을 조정할 수 있습니다. `7900`은 Dashboard이고 `4321`은 독립 구성 페이지이므로 용도가 다릅니다.

### Claude Code 및 Codex 통합

설치 프로그램은 Agent Pulse의 전역 hooks를 다음 위치에 병합합니다.

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

세션 시작, 사용자 제출, 도구 호출 전후, 권한 요청, 중지 및 실패 등의 이벤트를 감지하고 Dashboard, 플로팅 창 및 하드웨어 표시등을 업데이트합니다. 다른 hooks와 설정은 유지됩니다.

검증 방법: 새 Claude Code 또는 Codex 세션을 열고 요청을 한 번 제출한 뒤 도구 호출 또는 권한 요청을 발생시켜 Dashboard의 실시간 이벤트와 상태 색상을 확인합니다.

> Codex Offline Sandbox는 로컬 loopback 네트워크를 차단할 수 있습니다. Agent Pulse는 로컬 상태 파일 감시를 통해 계속 상태를 동기화하며 이 네트워크 채널에 의존하지 않습니다.

#### Codex hooks 신뢰 및 구성

Agent Pulse가 Codex 이벤트를 수신하려면 Codex가 외부 command hooks의 실행을 허용해야 합니다. 처음 설치할 때 또는 Codex가 hook 보안 확인을 표시할 때는 **Agent Pulse hooks 신뢰/허용**을 선택하세요. 거부하거나 신뢰하지 않으면 Codex는 이 명령을 실행하지 않으므로 Dashboard와 물리 상태등이 Codex 상태 변경을 반영하지 않습니다.

구성 단계:

1. Agent Pulse Dashboard를 열고 "구성"을 클릭합니다.
2. 구성 페이지에서 "Codex Hooks 설치"를 클릭합니다.
3. `%USERPROFILE%\.codex\hooks.json`에 Agent Pulse hooks가 포함되어 있는지 확인합니다. 설치 과정에서는 기존의 다른 hooks가 유지됩니다.
4. Codex를 재시작하거나 새 세션을 엽니다.
5. Codex가 hook 신뢰/실행 확인을 표시하면 신뢰 또는 허용을 선택합니다.
6. 요청을 한 번 제출하고 도구 호출을 발생시켜 Dashboard의 실시간 이벤트에 Codex 상태가 표시되는지 확인합니다.

상태가 업데이트되지 않으면 먼저 Codex의 hook 신뢰 상태를 확인하고, 구성 페이지에서 해당 hooks를 다시 설치한 후 Codex를 재시작하거나 새 세션을 여세요. 반복 설치해도 Agent Pulse hooks가 누적되지는 않습니다. 이전 버전을 사용했고 눈에 띄는 지연이 발생한다면 hooks를 한 번 다시 설치하여 정리 마이그레이션을 완료할 수 있습니다.

## 하드웨어 상태등

현재 HW v2 / ESP32-C3-next 물리 장치는 **빨간색, 노란색, 녹색의 독립 LED 세 개**를 사용하며 파란색 LED는 없습니다. 물리 표시등 상태는 다음과 같습니다.

- **녹색 LED**: 작업 완료, 세션 유휴 상태 또는 결과 확인 가능.
- **노란색 LED**: 응답 중, 도구 호출 중 또는 처리 중.
- **빨간색 LED**: 권한 요청, 도구 실패, 차단됨 또는 사람의 개입 필요.

상시 점등, 점멸 및 호흡 효과는 구성 페이지에서 자유롭게 조정할 수 있습니다.

> Dashboard의 BLE 아이콘은 파란색으로 표시될 수 있으며, 이는 컴퓨터에서 Bluetooth를 검색하거나 연결하고 있음을 나타낼 뿐 **장치가 파란색으로 켜진다는 뜻은 아닙니다**.

### 전원 켜기, 끄기 및 버튼

- **전원 켜기**: 전원이 꺼진 상태에서 버튼을 약 2초간 길게 누르면 장치가 전원 공급을 유지하고 시작합니다.
- **전원 켜기 피드백**: 장치는 빨간색 → 노란색 → 녹색을 차례로 표시한 뒤 기본 녹색 점멸 및 BLE 광고 상태로 진입합니다. 소리가 활성화되어 있으면 시작 알림음이 재생됩니다.
- **전원 끄기**: 전원을 켠 후 다시 약 2초간 길게 누르면 장치의 불이 꺼지고 전원 유지가 해제됩니다. 소리가 활성화되어 있으면 종료 알림음이 재생됩니다.
- **짧게 누르기**: 약 2초간 배터리 잔량을 표시합니다. 아직 BLE에 연결되지 않았다면 동시에 광고를 다시 시작/깨웁니다.
- **업그레이드 중**: OTA 전송 중에는 의도치 않은 업그레이드 중단을 방지하기 위해 버튼 조작이 무시됩니다.

### 물리 표시등 및 식별 피드백

- **광고 중, 연결 대기 중**: 녹색 LED 호흡.
- **BLE 연결됨**: 녹색 LED 상시 점등 후 현재 Agent 상태를 복원/수신합니다.
- **연결 해제됨**: 장치가 다시 광고하고 녹색 LED 호흡 상태로 돌아갑니다.
- **약 60초 동안 연결되지 않음**: 광고를 중지하고 빨간색 LED가 점멸합니다. 짧게 눌러 광고를 다시 시작할 수 있습니다.
- **장치 식별**: Dashboard에서 식별을 실행하면 장치가 빨간색 → 노란색 → 녹색 → 꺼짐을 빠르게 표시하고, 몇 차례 반복한 뒤 원래 상태로 돌아갑니다.
- **연결 애니메이션**: 장치는 빨간색, 노란색, 녹색 순서로 연결 과정을 알립니다. 연결이 완료되면 호스트가 현재 작업 상태를 전송합니다.

### 배터리, 충전 및 소리

장치를 짧게 누르면 약 2초간 배터리 잔량이 표시됩니다.

| 전압 추정 | 조명 효과 |
|---|---|
| 약 ≥ 4.0 V | 빨간색, 노란색, 녹색 모두 켜짐 |
| 약 3.7–4.0 V | 빨간색, 노란색 켜짐 |
| 약 < 3.7 V | 빨간색 LED 켜짐 |

연결되어 있고 장치가 지원하는 경우 Dashboard와 플로팅 창에 추정 전압, 백분율 및 충전 상태가 표시됩니다. 이 값은 일상적인 판단용이며 정밀 배터리 측정기로 사용해서는 안 됩니다.

구성 페이지의 “소리” 스위치는 부저 알림음을 제어하며 기본값은 꺼짐입니다. 3색 밝기와 소리 설정은 장치에 저장되어 전원이 꺼진 뒤에도 유지됩니다.

### 연결 방법

#### BLE 연결(권장)

1. 장치에 전원을 공급합니다. 오랫동안 연결되지 않았다면 짧게 한 번 눌러 다시 광고하게 합니다.
2. Dashboard를 열고 BLE 상태가 검색/연결 중에서 연결됨으로 바뀔 때까지 기다립니다.
3. 연결에 성공하면 Agent Pulse가 현재 상태를 물리 표시등에 자동으로 동기화합니다.

Dashboard BLE 아이콘은 일반적으로 다음을 의미합니다. 파란색은 검색/연결 중, 녹색은 최근 유효한 장치 상호작용 수신, 회색은 연결되지 않음, 빨간색은 연결 오류입니다. 파란색은 소프트웨어 아이콘 상태일 뿐 물리 LED가 아닙니다.

장치를 찾을 수 없으면 장치 전원이 켜져 있고 광고 중인지, Windows Bluetooth를 사용할 수 있는지, 장치가 충분히 가까운지 확인하고 여러 Agent Pulse/BLE Bridge 인스턴스를 동시에 실행하지 마세요.

#### USB 연결, 진단 및 복구

USB는 유선 표시등 제어, 장치 정보/배터리 잔량 읽기, 진단, 복구 및 호환 장치의 펌웨어 업그레이드에 사용할 수 있습니다. 충전 전용 케이블이 아닌 데이터 케이블을 사용하고 장치 관리자에서 COM 포트가 표시되는지 확인하세요.

현재 버전은 USB 직렬 포트의 제조업체 식별자를 기준으로 후보 장치를 필터링합니다. 여러 ESP32 또는 일반 USB 직렬 장치가 연결되어 있다면 명령줄에서 대상 포트를 명시적으로 지정하세요. 예:

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

관련 없는 직렬 포트 장치를 Agent Pulse 표시등 제어 대상으로 사용하지 마세요. 이후 버전에서는 후보 직렬 포트에 먼저 `deviceInfo` 요청을 보내고 유효한 장치 응답을 받은 경우에만 자동으로 연결합니다.

장치에 직렬 포트가 전혀 없으면 케이블, 드라이버 및 펌웨어에서 USB CDC가 활성화되어 있는지 확인하세요. USB는 초기 플래싱, 파티션 마이그레이션 또는 OTA 실패 후 우선 복구 방법입니다.

## 플로팅 창

Dashboard에서 “플로팅 표시등 열기” 또는 “플로팅 표시등 닫기”를 클릭하세요. 플로팅 창에는 현재 상태 색상, 프로젝트 이름, BLE 상태 및 사용할 수 있을 때 장치 배터리 잔량이 표시됩니다.

![데스크톱 플로팅 창 (노란색 표시등 = 진행 중)](docs/screenshots/floating-window.png)

이 창은 설치 버전의 daemon이 관리합니다. 플로팅 창 시작에 실패하더라도 Dashboard와 상태 동기화는 계속 작동할 수 있습니다.

## 프로그램 업데이트

Dashboard의 **AgentPulse 프로그램 업데이트** 영역에서 다음을 수행합니다.

1. “프로그램 업데이트 확인”을 클릭합니다.
2. 새 버전이 발견되면 “설치 확인”을 클릭합니다.
3. Agent Pulse가 서명된 업데이트 매니페스트, 설치 패키지 이름, 크기 및 SHA-256을 다운로드하고 검증합니다.
4. 검증이 통과하면 Windows 탐색기가 열리고 검증된 설치 패키지가 선택됩니다.
5. 해당 EXE를 직접 두 번 클릭하고 표시되는 Inno Setup 마법사에서 설치를 완료하세요.

Agent Pulse는 설치 패키지를 무인 실행하지 않으며 설치 마법사를 대신 완료하지도 않습니다. 덮어쓰기 설치 시 Inno 마법사는 사용 중인 파일을 해제하기 위해 현재 설치 디렉터리의 Agent Pulse daemon, 플로팅 창 및 BLE Bridge를 종료합니다. 관련 없는 프로그램을 광범위하게 종료하지는 않습니다.

다운로드된 설치 패키지는 기본적으로 다음 위치에 캐시됩니다.

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## 펌웨어 업데이트

하드웨어 기능:

- **ESP32-C3-next 업그레이드 가능 펌웨어**: Dashboard의 BLE/USB OTA를 사용하려면 장치 정보가 다음 하드웨어 식별자와 OTA 기능을 보고해야 합니다.

  ```text
  agentpulse-esp32c3-next
  ```

업그레이드 전에 장치 정보와 대상 펌웨어를 확인하세요. OTA는 Arduino **애플리케이션 이미지** `.ino.bin`만 허용합니다. bootloader, 파티션 테이블, `merged.bin` 또는 기타 전체 최초 플래싱 파일을 업로드하지 마세요.

### 중요 제한 사항

현재 OTA는 여전히 실험실 기능입니다. 펌웨어 측은 아직 이미지 서명 검증, Secure Boot, Flash Encryption, 상태 확인 및 자동 롤백을 구현하지 않았습니다. 업그레이드 중에는 전원을 끄거나 케이블을 뽑거나 Bluetooth를 끄거나 daemon을 종료하지 마세요. 실패한 경우 우선 USB로 복구하세요.

***업그레이드 중 갑작스러운 전원 차단으로 인해 업그레이드가 실패하고 사용에 영향을 주지 않도록 업그레이드 시 전원을 연결하는 것이 좋습니다.***

기존 장치의 OTA 파티션 레이아웃은 일반 BLE/USB application OTA로 마이그레이션할 수 없습니다. 파티션 레이아웃을 마이그레이션하거나 최초 플래싱할 때는 USB download/bootloader 모드로 bootloader, 파티션 테이블, OTA boot selector 및 factory app을 완전히 플래싱해야 합니다.

## 데이터 및 개인정보

기본 런타임 데이터는 로컬에 저장됩니다.

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

Agent Pulse는 기본적으로 코드, 프롬프트, 터미널 출력 또는 프로젝트 파일을 업로드하지 않습니다. 프로젝트 루트 디렉터리의 기존 `.agent-pulse`는 호환성/마이그레이션에만 사용되며 새 버전은 더 이상 프로젝트 디렉터리에 새 런타임 데이터를 쓰지 않습니다.

## 자주 묻는 질문

### Dashboard를 열 수 없음

구성 페이지 포트 `4321`이 아니라 `http://127.0.0.1:7900`에 접속하고 있는지 확인하세요. 설치 버전에서는 시작 메뉴에서 Agent Pulse를 다시 시작해 볼 수 있고, 개발자는 명령줄에서 다음을 확인할 수 있습니다.

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

소스 daemon과 설치 버전을 동시에 실행하지 마세요. `7900`, `47801`, `7950` 및 BLE 장치를 서로 차지하게 됩니다.

### Claude Code/Codex 상태가 변경되지 않음

1. 새 Claude Code/Codex 세션을 엽니다.
2. 구성 페이지에서 해당 hooks를 다시 설치합니다.
3. `%USERPROFILE%\.claude\settings.json` 또는 `%USERPROFILE%\.codex\hooks.json`에 여전히 Agent Pulse 구성이 포함되어 있는지 확인합니다.
4. Claude Code 사용자는 다음을 실행할 수 있습니다.

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE를 연결할 수 없음

장치 전원, Windows Bluetooth, 거리 및 Dashboard 상태를 확인하세요. `47801`을 점유하지 않도록 추가 BLE Bridge를 수동으로 시작하지 마세요.

### USB 장치를 찾을 수 없음

데이터 케이블을 사용하고 장치 관리자의 “포트(COM 및 LPT)”를 확인하며, 필요한 경우 명확한 COM 포트를 선택하세요. COM 포트가 없으면 USB CDC 펌웨어와 드라이버를 확인하세요.

### 알림이 너무 잦음

구성 페이지에서 완료/오류/멈춤 알림을 끄거나 “멈춤” 판정 시간을 조정하세요.

## 참고 사항

- 이 문서는 Windows 설치 패키지 사용자를 대상으로 합니다. 운영 환경에서 사용하기 전에 대상 컴퓨터와 하드웨어에서 Claude/Codex hooks, BLE, 플로팅 창 및 업데이트 절차를 검증해야 합니다.
- 여러 Agent Pulse 장치는 현재 동일한 BLE 이름만으로 자동 구분해서는 안 됩니다. 향후 다중 장치 환경에서는 고유한 `deviceId` 바인딩을 사용해야 하며 RSSI는 최초 검색 시 정렬 기준으로만 적합합니다.
- 프로그램 업데이트와 펌웨어 OTA는 서로 다른 절차입니다. 프로그램 업데이트는 Windows EXE를 설치하며 펌웨어 OTA는 호환 장치의 애플리케이션 이미지만 씁니다.
