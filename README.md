# Heiman HS1SA-E Smoke Detector with Zigbee2MQTT and Node-RED

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node-RED](https://img.shields.io/badge/Node--RED-3.0+-red.svg)](https://nodered.org/)

Integrate the **Heiman HS1SA-E Zigbee smoke detector** into your smart home using Zigbee2MQTT and Node-RED. This project provides a complete Node-RED flow for monitoring the detector, sending alerts via **Mail**, **ntfy**, and **Pushover**, and triggering a visual alarm (e.g., a blinking Zigbee bulb) when smoke is detected.

> **Note on the Heiman HS1SA-E:** This model does **not** support a test alarm via Zigbee, only the physical button. The successor model might include this feature.

## Features

-   **Periodic Status Reports**: Daily (or on-demand) reports on battery, signal strength, test mode, and device status.
-   **Multi-Channel Alerts**:
    -   **Email** (HTML formatted status table)
    -   **Pushover** (High-priority push notifications with alarm sound)
    -   **ntfy.sh** (Self-hostable push notifications)
-   **Visual Alarm**: Blinks a Zigbee smart bulb (e.g., a candle bulb) in red for 30 seconds upon alarm.
-   **Node-RED Dashboard**: UI elements to show battery level, smoke status, signal quality, and more.
-   **Smart Battery Monitoring**: Automatic low-battery warnings (<20%).

## Prerequisites

-   **Hardware**:
    -   Raspberry Pi (or any server running Node-RED)
    -   Zigbee Coordinator (e.g., Sonoff Zigbee 3.0 USB Dongle Plus)
    -   Heiman HS1SA-E Smoke Detector
    -   (Optional) Zigbee bulb for visual alarm
-   **Software**:
    -   [Node-RED](https://nodered.org/) (version 3.0 or later)
    -   [Zigbee2MQTT](https://www.zigbee2mqtt.io/) (with a working MQTT broker)
    -   Node-RED Palettes:
        -   `node-red-contrib-zigbee2mqtt`
        -   `node-red-node-email`
        -   `node-red-contrib-pushover`
        -   `@flowfuse/node-red-dashboard`
    -   Accounts (optional): SMTP server, [Pushover](https://pushover.net/), [ntfy](https://ntfy.sh/) (or self-hosted)

## Pairing the Heiman HS1SA-E with Zigbee2MQTT

The detector is power-sensitive and goes to sleep quickly. To successfully pair it:

1.  Put your Zigbee2MQTT adapter into pairing mode (permit join).
2.  Press and hold the reset button on the smoke detector for about 5 seconds until the LED flashes.
3.  **Crucial step:** During the interview process, press the reset button **every 2-3 seconds** to prevent the device from going back to sleep. You may need to repeat this several times until all data clusters are read successfully.

## Node-RED Flow

The flow consists of three main parts:

1.  **Status Reporting (Polling & Push)**: Polls the detector daily or on demand, formats a report, and sends it via email, Pushover, and ntfy.
2.  **Alarm Handling (Event-driven)**: Listens for the `smoke` attribute from Zigbee2MQTT, triggers push notifications, and starts the visual alarm.
3.  **Visual Alarm (Blinking Bulb)**: Controls a Zigbee bulb to blink red for 30 seconds when an alarm is triggered, then restores the previous state.

### Import the Flow

Copy the entire JSON array from [`flows/heiman-hs1sa-e-flow.json`](./flows/heiman-hs1sa-e-flow.json) and paste it into the Node-RED editor (Menu → Import).

> ⚠️ **Important:** You must configure the following nodes with your own credentials/settings before the flow will work:
> -   **Zigbee2MQTT Server**: All `zigbee2mqtt-get` and `zigbee2mqtt-out` nodes.
> -   **E-mail**: SMTP server, port, credentials.
> -   **Pushover**: Application keys and user keys.
> -   **ntfy**: Topic URL (e.g., `https://ntfy.sh/your-secret-topic`).
> -   **Device IDs & Friendly Names**: Replace the example IDs/names with those from your own Zigbee2MQTT devices.

## Flow Structure (Simplified)

```text
[Manual Test] / [Daily Cron]
    │
    ▼
[zigbee2mqtt-get] ──► [Parse Zigbee Data]
                           │
                           ├──► [Node-RED Dashboard (Gauges/Text)]
                           │
                           ▼
              [Format Status Report (Email, Pushover, ntfy)]
                           │
                           ▼
                  (Send via SMTP/Pushover/HTTP)

[zigbee2mqtt-in] (smoke attribute)
    │
    ▼
[Alarm Switch] (if smoke == true)
    ├──► [Pushover High-Priority Alert]
    └──► [30s Blink Trigger] ──► [Blink Candle Bulb (Red)]
