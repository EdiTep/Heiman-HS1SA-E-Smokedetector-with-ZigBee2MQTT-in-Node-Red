<p align="center">
  <img src="pictures/Edis-Blog-IoT-Logo.png" alt="Logo edi.teppert.com Blog" width="600"/>
</p>

# Heiman HS1SA-E Rauchmelder mit Zigbee2MQTT und Node-RED

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node-RED](https://img.shields.io/badge/Node--RED-3.0+-red.svg)](https://nodered.org/)
[![Powered by ntfy](https://img.shields.io/badge/Powered_by-ntfy-25B864.svg?logo=rocket&logoColor=white)](https://ntfy.sh/)
[![Powered by Zigbee2MQTT](https://img.shields.io/badge/Powered_by-Zigbee2MQTT-EE6C4D.svg?logo=zigbee&logoColor=white)](https://zigbee2mqtt.io/)
[![Powered by Pushover](https://img.shields.io/badge/Powered_by-Pushover-0088FF.svg?logo=android&logoColor=white)](https://pushover.net/)

<p align="center">
  <img src="pictures/2026-04-19-edi-teppert-com-Heimann-Rauchmelder-Titel-2048x820.jpg" alt="Rauchmelder Titel" width="800"/>
</p>

## 🌍 Language Selector / Sprachauswahl

| Language | Documentation |
|----------|---------------|
| 🇬🇧 English | [README.md](./README.md) |
| 🇩🇪 Deutsch | [README.de.md](./README.de.md) |

Integriere den **Heiman HS1SA-E Zigbee Rauchmelder** mit Zigbee2MQTT und Node-RED in dein Smart Home. Dieses Projekt bietet einen vollständigen Node-RED Flow zur Überwachung des Melders, zum Versenden von Alarmen per **E-Mail**, **ntfy** und **Pushover** sowie zur Auslösung eines visuellen Alarms (z. B. eine blinkende Zigbee-Lampe) bei Rauchentwicklung.

> **Hinweis zum Heiman HS1SA-E:** Dieses Modell unterstützt **keinen** Testalarm via Zigbee, nur den physischen Taster. Das Nachfolgemodell könnte diese Funktion enthalten.

## Funktionen

- **Regelmäßige Statusberichte**: Tägliche (oder bedarfsgesteuerte) Berichte zu Batterie, Signalstärke, Testmodus und Gerätestatus.
- **Mehrkanal-Alarme**:
  - **E-Mail** (HTML formatierte Statustabelle)
  - **Pushover** (Push-Benachrichtigungen mit hoher Priorität und Alarmton)
  - **ntfy.sh** (Selbst hostbare Push-Benachrichtigungen)
- **Visueller Alarm**: Lässt eine Zigbee-Lampe (z. B. eine Kerzenbirne) bei Alarm 30 Sekunden lang rot blinken.
- **Node-RED Dashboard**: UI-Elemente zur Anzeige von Batteriestand, Rauchstatus, Signalqualität und mehr.
- **Intelligente Batterieüberwachung**: Automatische Warnungen bei schwacher Batterie (<20%).

## Voraussetzungen

- **Hardware**:
  - Raspberry Pi (oder ein beliebiger Server mit Node-RED)
  - Zigbee Coordinator (z. B. Sonoff Zigbee 3.0 USB Dongle Plus)
  - Heiman HS1SA-E Rauchmelder
  - (Optional) Zigbee-Lampe für visuellen Alarm
- **Software**:
  - [Node-RED](https://nodered.org/) (Version 3.0 oder höher)
  - [Zigbee2MQTT](https://www.zigbee2mqtt.io/) (mit einem funktionierenden MQTT Broker)
  - Node-RED Paletten:
    - `node-red-contrib-zigbee2mqtt`
    - `node-red-node-email`
    - `node-red-contrib-pushover`
    - `@flowfuse/node-red-dashboard`
  - Konten (optional): SMTP-Server, [Pushover](https://pushover.net/), [ntfy](https://ntfy.sh/) (oder selbst gehostet)

<p align="center">
  <img src="pictures/2026-04-19-edi-teppert-com-Node-Red-ZigBee2MQTT-Rauchmelder-1024x811.jpg" alt="Node-Red Dashboard 2.0 Rauchmelder" width="400"/>
  <img src="pictures/2026-04-19-edi-teppert-com-Node-Red-Dashboard-2.0-Rauchmelder-1-785x1024.jpg" alt="Node-Red Dashboard 2.0 Rauchmelder" width="400"/>
</p>

## Kopplung des Heiman HS1SA-E mit Zigbee2MQTT

Der Melder ist stromsparend und geht schnell in den Schlafmodus. Für eine erfolgreiche Kopplung:

1. Versetze deinen Zigbee2MQTT-Adapter in den Pairing-Modus (Permit Join).
2. Halte die Reset-Taste am Rauchmelder etwa 5 Sekunden lang gedrückt, bis die LED blinkt.
3. **Wichtiger Schritt:** Drücke während des Interview-Prozesses die Reset-Taste **alle 2-3 Sekunden**, damit das Gerät nicht wieder in den Schlafmodus fällt. Wiederhole dies ggf. mehrmals, bis alle Datencluster erfolgreich ausgelesen wurden.

## Node-RED Flow

Der Flow besteht aus drei Hauptteilen:

1. **Statusberichterstattung (Polling & Push)**: Fragt den Melder täglich oder auf Abruf ab, formatiert einen Bericht und sendet ihn per E-Mail, Pushover und ntfy.
2. **Alarmbehandlung (Ereignisgesteuert)**: Hört auf das `smoke`-Attribut von Zigbee2MQTT, löst Push-Benachrichtigungen aus und startet den visuellen Alarm.
3. **Visueller Alarm (Blinkende Lampe)**: Steuert eine Zigbee-Lampe so, dass sie bei Alarm 30 Sekunden lang rot blinkt, und stellt dann den vorherigen Zustand wieder her.

### Flow importieren

Kopiere das gesamte JSON-Array aus [`flows/heiman-hs1sa-e-flow.json`](./flows/heiman-hs1sa-e-flow.json) und füge es in den Node-RED Editor ein (Menü → Importieren).

> ⚠️ **Wichtig:** Du musst die folgenden Nodes mit deinen eigenen Zugangsdaten/Einstellungen konfigurieren, bevor der Flow funktioniert:
> - **Zigbee2MQTT Server**: Alle `zigbee2mqtt-get` und `zigbee2mqtt-out` Nodes.
> - **E-Mail**: SMTP-Server, Port, Zugangsdaten.
> - **Pushover**: Application Keys und User Keys.
> - **ntfy**: Themen-URL (z. B. `https://ntfy.sh/dein-geheimer-topic`).
> - **Geräte-IDs & Friendly Names**: Ersetze die Beispiel-IDs/Namen durch die deiner eigenen Zigbee2MQTT-Geräte.

## Konfigurationsschritte

### 1. Zigbee2MQTT Nodes konfigurieren

- Bearbeite den `zigbee2mqtt-server` Konfigurationsnode. Setze deine MQTT-Broker-Adresse, den Port und die Zugangsdaten.
- Ersetze den `friendly_name` und die `device_id` in den `zigbee2mqtt-get` und `zigbee2mqtt-in` Nodes durch die **deines** Heiman Melders.
- Führe dasselbe für den `zigbee2mqtt-out` Node deiner Zigbee-Lampe durch.

### 2. Benachrichtigungen einrichten

- **E-Mail**: Bearbeite den `e-mail` Node. Gib deine SMTP-Serverdetails, Authentifizierung und Empfängeradresse ein.
- **Pushover**: Doppelklicke auf die `pushover api` Nodes. Wähle oder erstelle eine `pushover-keys` Konfiguration. Füge deinen Pushover Application Key und User Key hinzu.
- **ntfy**: Ersetze im Funktionsnode `Statusbericht formatieren` die URL `https://ntfy.sh/your-secret-theme-in-ntfy` durch deine eigene ntfy-Topic-URL.

### 3. Anpassungen (Optional)

- **Dashboard**: Der Flow enthält eine Node-RED Dashboard Gruppe (`Keller Werkstatt`). Du kannst die UI-Elemente (Messinstrumente, Text-Nodes) anpassen oder entfernen, falls nicht benötigt.
- **Alarmdauer**: Ändere im `30s Blinksteuerung` (Trigger) Node die `duration` (aktuell 30 Sekunden).
- **Lampenverhalten**: Die blinkende Lampe leuchtet rot. Du kannst die Farbe oder Helligkeit in den Funktionsnodes `Blink-Modus vorbereiten` und `Toggle-Blinken` ändern.

## Beispiel eines Statusberichts (E-Mail)

Der E-Mail-Bericht enthält eine übersichtliche HTML-Tabelle mit Batteriestatus, Rauchstatus, Testmodus und Signalqualität. Bei schwacher Batterie (<20%) wird eine rote Warnung hinzugefügt.

## Fehlersuche

- **Gerät koppelt nicht / unvollständige Daten**: Wie erwähnt, drücke während des Zigbee2MQTT-Interviews alle 2-3 Sekunden die Reset-Taste.
- **Keine Updates vom Melder**: Das Gerät wacht nur periodisch auf (z. B. für Batteriemeldungen) oder bei Alarm. Verwende bei Bedarf einen manuellen Test-Button (`zigbee2mqtt-get`), um einen sofortigen Status zu erhalten.
- **Alarm wird nicht ausgelöst**: Überprüfe den `zigbee2mqtt-in` Node. Das Attribut heißt `smoke` (boolean). Stelle sicher, dass die MQTT-Nachricht deines Melders dieses Feld enthält.
- **Lampe stellt vorherigen Zustand nicht wieder her**: Der Flow speichert einen simulierten vorherigen Zustand. Für eine dynamische Lösung müsstest du den aktuellen Zustand der Lampe vor dem Alarm per `zigbee2mqtt-get` abfragen.

## Dateien in diesem Repository

- `README.de.md` - Diese Dokumentation (Deutsch)
- `flows/heiman-hs1sa-e-flow.json` - Der exportierte Node-RED Flow (das zu importierende JSON-Array)
- `pictures/2026-04-19-edi-teppert-com-Heimann-Rauchmelder-Titel-2048x820.jpg` - Bild in diesem Repo
- `pictures/2026-04-19-edi-teppert-com-Node-Red-Dashboard-2.0-Rauchmelder-1-785x1024.jpg` - Bild in diesem Repo
- `pictures/2026-04-19-edi-teppert-com-Node-Red-ZigBee2MQTT-Rauchmelder-1024x811.jpg` - Bild in diesem Repo
- `pictures/Edis-Blog-IoT-Logo.png` - Bild in diesem Repo

## Flow-Struktur (vereinfacht)

```text
[Manueller Test] / [Täglicher Cron]
    │
    ▼
[zigbee2mqtt-get] ──► [Zigbee-Daten parsen]
                           │
                           ├──► [Node-RED Dashboard (Messinstrumente/Text)]
                           │
                           ▼
              [Statusbericht formatieren (E-Mail, Pushover, ntfy)]
                           │
                           ▼
                  (Senden via SMTP/Pushover/HTTP)

[zigbee2mqtt-in] (smoke Attribut)
    │
    ▼
[Alarm-Status prüfen] (wenn smoke == true)
    ├──► [Pushover Hochprioritäts-Alarm]
    └──► [30s Blinkauslöser] ──► [Kerzenbirne blinken lassen (Rot)]
```

## 📫 Kontakt & Mitwirken

I'm excited about **exchange, feedback, and collaboration** – especially on topics like:

- Energy management (PV, battery SOC, heat pumps)
- Node-RED flow optimization
- Integration of proprietary systems (e.g., Tahoma)

You can best reach me via **email** (provided on my profile).

---

> 💡 *Most of my projects are "Version 0.8" – I continuously improve them. Feel free to open issues or submit pull requests!*
## 

Du hast einen Fehler gefunden oder eine Idee zur Verbesserung? Erstelle gerne ein Issue oder reiche einen Pull Request ein. Der Flow ist Version 0.8 – es gibt Raum für Verbesserungen (z. B. Integration eines echten akustischen Alarms, Unterstützung mehrerer Melder).

## Lizenz

Dieses Projekt ist Open-Source und steht unter der MIT-Lizenz zur Verfügung.

Original-Blogbeitrag (Deutsch): Heiman HS1SA-E Rauchmelder mit ZigBee2MQTT in Node-Red

Autor: Edi Teppert (C)

---


## 📈 Letzte Aktivität

![Edi's GitHub stats](https://github-readme-stats.vercel.app/api?username=EdiTep&show_icons=true&theme=default)

