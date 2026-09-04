# Agent Pulse

Agent Pulse synchronisiert die Arbeitszustände von Claude Code und Codex mit einem lokalen Windows-Dashboard, einem schwebenden Fenster und einer physischen ESP32-Dreifarb-LED, damit du den Fortschritt deiner KI-Programmiersitzungen auch ohne ständigen Blick aufs Terminal verfolgen kannst.

Sprache: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | Deutsch | [Español](README.es.md)

## Aktuelle Version

Die aktuelle Quellcodeversion ist `0.4.1` (2026-08-14), die ESP32-Firmware bleibt bei `0.1.21+22`. Eine einzelne Leuchte behält das einfache Standardverhalten und folgt der neuesten Aufgabe. Bei mehreren Leuchten kann jede unabhängig der neuesten Aufgabe, einem Projekt oder einem Agent wie Claude Code, Codex, WorkBuddy oder CodeBuddy folgen und ein eigenes Lichtprofil verwenden.

Diese Version zeigt außerdem alle verbundenen BLE- und USB-Geräte mit Verbindungs- und Akkustatus an, verhindert doppelte BLE-Bridge-Prozesse und kennzeichnet das schwebende Fenster mit Agent und Projekt der neuesten Aufgabe. Bluetooth unterstützt weiterhin Nahbereichsverbindung und systemgekoppelte Verbindung unter Windows. Updates verwenden zuerst Gitee und wechseln bei Fehlern automatisch zu GitHub. Einzelheiten stehen im [Änderungsprotokoll](CHANGELOG.md).

## Bedeutung der Statusfarben

| Farbe | Übliche Bedeutung |
|---|---|
| Grün | Sitzung ist inaktiv, Aufgabe beendet oder Ergebnis kann angesehen werden |
| Gelb | Antwort wird erstellt, ein Tool wird aufgerufen, Verarbeitung wird nach Tool-Abschluss fortgesetzt oder weitere Informationen werden benötigt |
| Rot | Berechtigungsanfrage, Tool-Fehler, Blockierung oder menschliches Eingreifen erforderlich |

Status werden pro Projektverzeichnis gespeichert und angezeigt; mehrere Projekte auf demselben Computer können gleichzeitig im Dashboard erscheinen.

## Installation

### Empfohlen: Windows-Installationspaket

Lade `AgentPulseSetup-<版本>.exe` herunter und führe es aus. Normale Benutzer müssen Node.js, npm, Python, BLE Bridge, PyInstaller oder Arduino-Werkzeuge nicht selbst installieren.

Offizielle Download-Links:

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

Das Installationsprogramm richtet für den aktuellen Windows-Benutzer Folgendes ein:

- Agent Pulse daemon, integrierte Node-Laufzeit, BLE Bridge und schwebendes Fenster installieren;
- Claude Code-Hooks und Codex-Hooks sicher zusammenführen, ohne vorhandene andere Hooks zu überschreiben;
- automatischen Start nach der Anmeldung einrichten;
- nach Abschluss Agent Pulse starten und das Dashboard öffnen.

Der Standardinstallationsort ist gewöhnlich:

```text
%LOCALAPPDATA%\AgentPulse
```

Starte nach der Installation Claude Code/Codex neu oder öffne eine neue Sitzung, damit die Hooks neu geladen werden.

### Installation aus dem Quellcode/über die Befehlszeile

Dies ist der Weg für Entwickler und kein erforderlicher Schritt für Benutzer des Windows-Installationspakets. Siehe den [Entwickleranhang](#开发者附录) am Ende dieses Dokuments.

## Tägliche Nutzung

### Dashboard

Öffne:

```text
http://127.0.0.1:7900
```

Du kannst es auch über **Open Dashboard** im Startmenü öffnen. Das Dashboard ist der Einstiegspunkt für den täglichen Betrieb und zeigt:

- aktuelle Projekte, Lichtfarben und Echtzeitereignisse;
- BLE-Verbindungsstatus und Geräteakkustand (wenn vom Gerät unterstützt);
- schwebendes Fenster ein-/ausblenden;
- Programm- und Firmware-Updates;
- Zugang zur Konfiguration.

Das Dashboard lauscht nur an der lokalen Loopback-Adresse und wird nicht im LAN veröffentlicht.

### Konfigurationsseite

Klicke im Dashboard auf „Konfiguration“, um die Konfigurationsseite zu öffnen. Ihre Standardadresse lautet:

```text
http://127.0.0.1:4321/?lang=zh
```

Du kannst Benachrichtigungen, die Zeit bis zur Erkennung eines festgefahrenen Zustands, Farbe/Blinken/Atmungseffekte für verschiedene Ereignisse, Helligkeit, Ton und weitere Optionen anpassen. `7900` ist das Dashboard, `4321` die eigenständige Konfigurationsseite; ihre Zwecke unterscheiden sich.

### Integration von Claude Code und Codex

Das Installationsprogramm führt die globalen Agent-Pulse-Hooks zusammen mit:

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

Es erkennt Ereignisse wie Sitzungsstart, Benutzereingabe, vor und nach Tool-Aufrufen, Berechtigungsanfragen, Stopps und Fehler und aktualisiert Dashboard, schwebendes Fenster und Hardwarelicht. Deine sonstigen Hooks und Einstellungen bleiben erhalten.

Überprüfung: Öffne eine neue Claude Code- oder Codex-Sitzung, sende eine Anfrage und löse einen Tool-Aufruf oder eine Berechtigungsanfrage aus. Beobachte anschließend die Echtzeitereignisse und Statusfarben im Dashboard.

> Codex Offline Sandbox kann den lokalen Loopback-Netzwerkzugriff blockieren; Agent Pulse synchronisiert den Status weiterhin durch Überwachung lokaler Statusdateien und ist nicht von diesem Netzwerkkanal abhängig.

#### Vertrauen und Konfiguration von Codex-Hooks

Codex muss die Ausführung externer Command-Hooks erlauben, damit Agent Pulse Codex-Ereignisse empfangen kann. Wähle bei der ersten Installation oder wenn Codex eine Sicherheitsbestätigung für Hooks anzeigt, **Agent-Pulse-Hooks vertrauen/zulassen**; wenn du sie ablehnst oder ihnen nicht vertraust, führt Codex diese Befehle nicht aus und das Dashboard sowie die physische Leuchte ändern sich nicht entsprechend dem Codex-Status.

Konfigurationsschritte:

1. Öffne das Agent-Pulse-Dashboard und klicke auf „Konfiguration“.
2. Klicke auf der Konfigurationsseite auf „Codex-Hooks installieren“.
3. Vergewissere dich, dass `%USERPROFILE%\.codex\hooks.json` die Agent-Pulse-Hooks enthält; bei der Installation bleiben andere vorhandene Hooks erhalten.
4. Starte Codex neu oder öffne eine neue Sitzung.
5. Wenn Codex eine Bestätigung zum Vertrauen in bzw. Ausführen von Hooks anzeigt, wähle Vertrauen oder Zulassen.
6. Sende eine Anfrage und löse einen Tool-Aufruf aus, um zu bestätigen, dass die Echtzeitereignisse im Dashboard den Codex-Status anzeigen.

Wenn der Status nicht aktualisiert wird, überprüfe zuerst den Vertrauensstatus der Codex-Hooks, installiere dann die betreffenden Hooks auf der Konfigurationsseite erneut und starte Codex neu oder öffne eine neue Sitzung. Mehrfache Installationen häufen Agent-Pulse-Hooks nicht an; wenn du eine ältere Version verwendet hast und deutliche Verzögerungen feststellst, installiere die Hooks einmal erneut, um die Bereinigungsmigration abzuschließen.

## Hardware-Statuslicht

Das aktuelle physische Gerät HW v2 / ESP32-C3-next verwendet **drei separate rote, gelbe und grüne LEDs**; es besitzt keine blaue LED. Die Zustände des physischen Lichts sind:

- **Grünes Licht**: Aufgabe abgeschlossen, Sitzung inaktiv oder Ergebnis kann angesehen werden.
- **Gelbes Licht**: Antwort wird erstellt, ein Tool wird aufgerufen oder Verarbeitung läuft.
- **Rotes Licht**: Berechtigungsanfrage, Tool-Fehler, Blockierung oder menschliches Eingreifen erforderlich.

Dauerlicht, Blinken und Atmungseffekte lassen sich auf der Konfigurationsseite frei einstellen.

> Das BLE-Symbol im Dashboard kann blau angezeigt werden. Es bedeutet nur, dass der Computer Bluetooth scannt oder eine Verbindung herstellt, und **nicht, dass das Gerät blau leuchtet**.

### Einschalten, Ausschalten und Taste

- **Einschalten**: Halte bei ausgeschaltetem Gerät die Taste etwa 2 Sekunden gedrückt. Das Gerät verriegelt die Stromversorgung und startet.
- **Einschalt-Rückmeldung**: Das Gerät zeigt nacheinander Rot → Gelb → Grün und wechselt dann in den standardmäßigen grün blinkenden Zustand mit BLE-Werbung; wenn Ton aktiviert ist, wird ein Startton abgespielt.
- **Ausschalten**: Halte nach dem Einschalten die Taste erneut etwa 2 Sekunden gedrückt. Das Gerät schaltet die LEDs aus und deaktiviert die Stromversorgungshaltefunktion; wenn Ton aktiviert ist, wird ein Ausschaltsignalton abgespielt.
- **Kurz drücken**: Zeigt den Akkustand etwa 2 Sekunden lang an; wenn noch keine BLE-Verbindung besteht, wird gleichzeitig die Werbung erneut gestartet/aktiviert.
- **Während eines Upgrades**: Tastenbetätigungen werden während der OTA-Übertragung ignoriert, damit das Upgrade nicht versehentlich unterbrochen wird.

### Physisches Licht und Erkennungsrückmeldung

- **Sendet Werbung und wartet auf Verbindung**: Grünes Licht atmet.
- **BLE verbunden**: Grünes Licht leuchtet dauerhaft; anschließend wird der aktuelle Agent-Status wiederhergestellt/empfangen.
- **Verbindung getrennt**: Das Gerät sendet erneut Werbung und kehrt zum atmenden grünen Licht zurück.
- **Nach etwa 60 Sekunden noch nicht verbunden**: Werbung wird beendet und das rote Licht blinkt; durch kurzes Drücken kann die Werbung erneut gestartet werden.
- **Gerät identifizieren**: Wenn die Identifikation über das Dashboard ausgeführt wird, zeigt das Gerät schnell Rot → Gelb → Grün → Aus, wiederholt dies einige Male und stellt anschließend den vorherigen Zustand wieder her.
- **Verbindungsanimation**: Das Gerät signalisiert den Verbindungsprozess in der Reihenfolge Rot, Gelb, Grün; nach Abschluss der Verbindung sendet der Host den aktuellen Arbeitsstatus.

### Akkustand, Laden und Ton

Ein kurzer Tastendruck zeigt den Akkustand etwa 2 Sekunden lang an:

| Geschätzte Spannung | Lichteffekt |
|---|---|
| Etwa ≥ 4.0 V | Rot, Gelb und Grün leuchten alle |
| Etwa 3.7–4.0 V | Rot und Gelb leuchten |
| Etwa < 3.7 V | Rotes Licht leuchtet |

Wenn verbunden und vom Gerät unterstützt, zeigen Dashboard und schwebendes Fenster die geschätzte Spannung, den Prozentsatz und den Ladestatus an. Die Werte dienen der täglichen Einschätzung und sind kein präzises Akkumessgerät.

Der Schalter „Ton“ auf der Konfigurationsseite steuert die Pieptöne des Summers und ist standardmäßig deaktiviert. Die Einstellungen für Dreifarbhelligkeit und Ton werden auf dem Gerät gespeichert und bleiben auch nach einem Stromausfall erhalten.

### Verbindungsarten

#### BLE-Nahbereichsverbindung (empfohlen)

1. Trenne das USB-Datenkabel der Statusleuchte und schalte sie ein. Falls keine Werbung mehr gesendet wird, starte sie mit einem kurzen Tastendruck erneut.
2. Öffne das Dashboard. Ohne gebundenes Gerät scannt Agent Pulse fortlaufend, bis automatisch oder manuell gebunden beziehungsweise der Scan angehalten wird.
3. Bringe die gewünschte Leuchte in die Nähe des Bluetooth-Adapters dieses Computers. Die Liste zeigt Name, MAC/Gerätekennung, RSSI und Anzahl der Messwerte in Echtzeit.
4. Automatische Bindung erfordert mindestens 3 Messwerte, ein RSSI von `-45 dBm` oder stärker und mindestens `8 dB` Vorsprung vor dem zweitstärksten Kandidaten. Falls unterschiedliche Empfangsstärken dies verhindern, wähle die Leuchte aus und klicke auf „Binden“.

Nach der Bindung endet der Scan und die Kennung wird lokal gespeichert. Künftige Starts verbinden nur diese Leuchte und wechseln nicht anhand der Signalstärke zu anderen Geräten. Zum Wechseln „Gerät vergessen“ auswählen und die Nahbereichs- oder manuelle Bindung erneut durchführen.

Das BLE-Symbol ist blau beim Scannen/Verbinden, grün nach kürzlich gültiger Kommunikation, grau ohne Verbindung und rot bei Fehlern. Nach der Verbindung wird nur der aktuelle gültige Zustand synchronisiert; abgelaufene alte Lichtereignisse werden nicht erneut gesendet. Windows zeigt normalerweise die Bluetooth-MAC. macOS kann wegen der CoreBluetooth-Datenschutzregeln eine vom System vergebene UUID anzeigen; sie eignet sich für die lokale Bindung, ist aber nicht die Hardware-MAC.

Wenn kein Gerät erscheint, prüfe Stromversorgung und Werbung, System-Bluetooth, dass USB getrennt ist und keine weitere Agent-Pulse-/BLE-Bridge-Instanz läuft. Unter macOS muss beim ersten Start außerdem der Bluetooth-Zugriff erlaubt werden.

#### USB-Seriellverbindung, Auswahl und Wiederherstellung

1. Verwende ein USB-Kabel mit Datenübertragung. Ein reines Ladekabel erzeugt keinen seriellen Port.
2. Öffne im Dashboard den Bereich „USB seriell“. Windows verwendet `COM*`, macOS gewöhnlich `/dev/cu.usbmodem*` oder `/dev/cu.usbserial*`.
3. Die automatische Standardauswahl verbindet nur das treiberlose AgentPulse-Gerät `VID:PID 303A:1001`. CH340, CP210x, FTDI und andere Adapter werden nicht automatisch geöffnet.
4. Sind mehrere `303A:1001`-Geräte angeschlossen oder wird ein anderer kompatibler Adapter benötigt, prüfe Port und VID/PID in der Liste und wähle den Zielport manuell.

Die manuelle Auswahl wird lokal gespeichert. Fehlt der gewählte Port, bleibt Agent Pulse offline und öffnet nicht unbemerkt einen anderen Port. Die automatische Auswahl kann jederzeit wieder aktiviert werden. Die Liste markiert Standard-, ausgewählte, verbundene und fehlende Ports; bei Firmwareunterstützung erscheinen Akku- und Ladestatus in der Kopfzeile.

USB hat Vorrang vor BLE: Eine USB-Verbindung pausiert BLE-Scan und -Verbindung; nach dem Trennen werden Scan oder Wiederverbindung zum gebundenen Gerät fortgesetzt. Wenn kein Port erscheint, prüfe Kabel, Treiber und USB-CDC-Firmware. USB ist der bevorzugte Wiederherstellungsweg für Erst-Flashen, Partitionsmigration oder nach einem OTA-Fehler.

## Schwebendes Fenster

Klicke im Dashboard auf „Schwebendes Licht öffnen“ oder „Schwebendes Licht schließen“. Das schwebende Fenster zeigt die aktuelle Statusfarbe, den Projektnamen, den BLE-Status und, sofern verfügbar, den Geräteakkustand.

![Desktop-Schwebefenster (gelbes Licht = in Bearbeitung)](docs/screenshots/floating-window.png)

Es wird vom daemon der Installationsversion verwaltet; selbst wenn das schwebende Fenster nicht gestartet werden kann, funktionieren Dashboard und Statussynchronisierung weiterhin.

## Programmupdate

Im Bereich **AgentPulse Programmupdate** des Dashboards:

1. Klicke auf „Nach Programmupdates suchen“.
2. Wenn eine neue Version gefunden wurde, klicke auf „Installation bestätigen“.
3. Agent Pulse lädt die signierte Update-Manifestdatei, den Installationspaketnamen, die Größe und SHA-256 herunter und prüft sie.
4. Nach erfolgreicher Prüfung öffnet sich der Windows-Explorer und markiert das geprüfte Installationspaket.
5. Doppelklicke diese EXE manuell und schließe die Installation im sichtbaren Inno-Setup-Assistenten ab.

Agent Pulse führt das Installationspaket nicht unbeaufsichtigt aus und schließt den Installationsassistenten nicht für dich ab. Bei einer Überinstallierung beendet der Inno-Assistent den Agent-Pulse-daemon, das schwebende Fenster und die BLE Bridge im aktuellen Installationsverzeichnis, um belegte Dateien freizugeben; nicht zugehörige Programme werden nicht pauschal beendet.

Heruntergeladene Installationspakete werden standardmäßig hier zwischengespeichert:

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## Firmware-Update

Hardwarefunktionen:

- **ESP32-C3-next mit aktualisierbarer Firmware**: Geräteinformationen müssen die folgende Hardwarekennung und OTA-Fähigkeit melden, damit BLE/USB OTA über das Dashboard verwendet werden kann:

  ```text
  agentpulse-esp32c3-next
  ```

Stelle vor dem Upgrade sicher, dass Geräteinformationen und Zielfirmware stimmen. OTA akzeptiert nur Arduino-**Anwendungsimages** `.ino.bin`; lade keinen Bootloader, keine Partitionstabelle, keine `merged.bin` und keine anderen vollständigen Dateien für das erste Flashen hoch.

### Wichtige Einschränkungen

Das aktuelle OTA ist weiterhin eine Laborfunktion: Die Firmware implementiert noch keine Überprüfung der Imagesignatur, Secure Boot, Flash Encryption, Gesundheitsbestätigung oder automatisches Rollback. Trenne während des Upgrades nicht die Stromversorgung oder das Kabel, schalte Bluetooth nicht ab und beende den daemon nicht; stelle bei einem Fehler vorrangig über USB wieder her.

***Es wird empfohlen, während des Upgrades eine Stromversorgung anzuschließen, damit ein plötzlicher Stromausfall den Upgrade-Vorgang nicht fehlschlagen lässt und die Nutzung beeinträchtigt.***

Das OTA-Partitionslayout alter Geräte kann nicht durch ein normales BLE/USB application OTA migriert werden. Zur Migration des Partitionslayouts oder beim ersten Flashen müssen bootloader, Partitionstabelle, OTA boot selector und factory app vollständig über den USB-Download-/Bootloader-Modus geflasht werden.

## Daten und Datenschutz

Standardmäßige Laufzeitdaten werden lokal gespeichert:

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

Agent Pulse lädt standardmäßig keinen Code, keine Prompts, Terminalausgaben oder Projektdateien hoch. Das alte `.agent-pulse` im Projektstamm dient nur der Kompatibilität/Migration; neue Versionen schreiben keine neuen Laufzeitdaten mehr in Projektverzeichnisse.

## Häufig gestellte Fragen

### Dashboard lässt sich nicht öffnen

Stelle sicher, dass `http://127.0.0.1:7900` aufgerufen wird und nicht der Konfigurationsseitenport `4321`. In der Installationsversion kannst du versuchen, Agent Pulse über das Startmenü neu zu starten; Entwickler können in der Befehlszeile Folgendes prüfen:

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

Führe Quellcode-daemon und Installationsversion nicht gleichzeitig aus, da sie um `7900`, `47801`, `7950` und das BLE-Gerät konkurrieren.

### Claude Code-/Codex-Status ändert sich nicht

1. Öffne eine neue Claude Code-/Codex-Sitzung.
2. Installiere auf der Konfigurationsseite die entsprechenden Hooks erneut.
3. Stelle sicher, dass `%USERPROFILE%\.claude\settings.json` oder `%USERPROFILE%\.codex\hooks.json` weiterhin die Agent-Pulse-Konfiguration enthält.
4. Claude Code-Benutzer können Folgendes ausführen:

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### BLE kann keine Verbindung herstellen

Überprüfe Stromversorgung des Geräts, Windows-Bluetooth, Entfernung und Dashboard-Status; starte keine zusätzliche BLE Bridge manuell, da sonst `47801` belegt werden kann.

### USB-Gerät nicht gefunden

Verwende ein Datenkabel, prüfe „Anschlüsse (COM und LPT)“ im Geräte-Manager und wähle bei Bedarf einen eindeutigen COM-Port. Wenn kein COM-Port vorhanden ist, überprüfe USB-CDC-Firmware und Treiber.

### Zu viele Benachrichtigungen

Deaktiviere auf der Konfigurationsseite Benachrichtigungen für Abschluss/Fehler/festgefahrene Zustände oder passe die Zeit zur Erkennung eines „festgefahrenen“ Zustands an.

## Hinweise

- Dieses Dokument richtet sich an Benutzer des Windows-Installationspakets. Überprüfe vor dem produktiven Einsatz auf dem Zielcomputer und mit der Hardware Claude-/Codex-Hooks, BLE, schwebendes Fenster und Updateabläufe.
- Umgebungen mit mehreren Geräten verwenden eine dauerhaft gebundene eindeutige Gerätekennung. RSSI dient nur der ersten Nahbereichsauswahl und wechselt niemals eine bereits gebundene Leuchte. Unter Windows ist die Kennung normalerweise eine MAC, unter macOS möglicherweise eine CoreBluetooth-UUID.
- Programmupdates und Firmware OTA sind unterschiedliche Abläufe: Programmupdates installieren die Windows-EXE; Firmware OTA schreibt nur das Anwendungsimage auf kompatible Geräte.
