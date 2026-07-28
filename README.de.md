# Agent Pulse

Agent Pulse synchronisiert die Arbeitszustände von Claude Code und Codex mit einem lokalen Windows-Dashboard, einem schwebenden Fenster und einer physischen ESP32-Dreifarb-LED, damit du den Fortschritt deiner KI-Programmiersitzungen auch ohne ständigen Blick aufs Terminal verfolgen kannst.

Sprache: [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Français](README.fr.md) | Deutsch | [Español](README.es.md)

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

#### BLE-Verbindung (empfohlen)

1. Versorge das Gerät mit Strom; wenn es längere Zeit nicht verbunden war, drücke die Taste kurz, damit es erneut Werbung sendet.
2. Öffne das Dashboard und warte, bis der BLE-Status von Scannen/Verbinden auf Verbunden wechselt.
3. Nach erfolgreicher Verbindung synchronisiert Agent Pulse automatisch den aktuellen Status mit dem physischen Licht.

Das BLE-Symbol im Dashboard bedeutet im Allgemeinen: Blau für Scannen/Verbinden, Grün für kürzlich empfangene gültige Geräteinteraktionen, Grau für nicht verbunden und Rot für Verbindungsfehler. Blau ist nur der Status eines Softwaresymbols, nicht der einer physischen LED.

Wenn das Gerät nicht gefunden wird, stelle sicher, dass es eingeschaltet ist und Werbung sendet, Windows-Bluetooth verfügbar ist, das Gerät nahe genug ist und nicht mehrere Agent-Pulse-/BLE-Bridge-Instanzen gleichzeitig laufen.

#### USB-Verbindung, Diagnose und Wiederherstellung

USB kann für kabelgebundene Lichtsteuerung, das Auslesen von Geräteinformationen/Akkustand, Diagnose, Wiederherstellung sowie Firmware-Upgrades kompatibler Geräte verwendet werden. Verwende ein Datenkabel statt eines reinen Ladekabels und bestätige im Geräte-Manager, dass ein COM-Port erscheint.

Die aktuelle Version filtert Kandidatengeräte anhand der Herstellerkennung des USB-Seriellanschlusses. Wenn mehrere ESP32- oder gängige USB-Seriellgeräte verbunden sind, gib den Zielport in der Befehlszeile ausdrücklich an, zum Beispiel:

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

Verwende keine nicht zugehörigen seriellen Geräte als Ziel für die Agent-Pulse-Lichtsteuerung. Künftige Versionen werden zuerst eine `deviceInfo`-Anfrage an Kandidatenports senden und nur dann automatisch verbinden, wenn eine gültige Geräteantwort empfangen wird.

Wenn das Gerät überhaupt keinen seriellen Port hat, überprüfe Kabel, Treiber und ob die Firmware USB CDC aktiviert hat. USB ist die bevorzugte Wiederherstellungsmethode nach dem ersten Flashen, einer Partitionsmigration oder einem fehlgeschlagenen OTA.

## Schwebendes Fenster

Klicke im Dashboard auf „Schwebendes Licht öffnen“ oder „Schwebendes Licht schließen“. Das schwebende Fenster zeigt die aktuelle Statusfarbe, den Projektnamen, den BLE-Status und, sofern verfügbar, den Geräteakkustand.

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
- Mehrere Agent-Pulse-Geräte sollten derzeit nicht allein anhand desselben BLE-Namens automatisch unterschieden werden; künftige Mehrgeräteszenarien sollten eine eindeutige `deviceId`-Bindung verwenden, und RSSI eignet sich nur als Sortiergrundlage bei der ersten Erkennung.
- Programmupdates und Firmware OTA sind unterschiedliche Abläufe: Programmupdates installieren die Windows-EXE; Firmware OTA schreibt nur das Anwendungsimage auf kompatible Geräte.


