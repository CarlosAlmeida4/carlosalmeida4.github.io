---
title: "Pajero Central Screen Mini Dashboard"
date: "2026-03-15"
slug: "PajeroCentralScreenMiniDashboard"
tags: ["Software", "Hardware", "ESP32", "Car", "Vehicle", "3dPrinting"]
categories: ["Projects"]
series: ["Pajero"]
draft: true
---

### Table of contents
1. [Introduction](#introduction)
2. [Developed Software](#developed-software)
3. [3D Design](#3d-design)
4. [Integration into the Vehicle](#integration-into-the-vehicle)
5. [Conclusion](#conclusion)

# Introduction

This page is the project-level view of my ESP32-S3 touchscreen inclinometer build.
The detailed implementation and architecture are described in the companion post:

- [ESP32 S3 Touchscreen Inclinometer - Advanced Dashboard][esp32_s3_post]

In short, this project turns the vehicle center display area into a compact, extensible HMI platform: an inclinometer first, but designed from day one to host additional telemetry and diagnostics features.


# Developed Software

The firmware is based on **ESP-IDF + LVGL + SquareLine Studio**, organized into modular components for display, sensors, OTA, and system orchestration.

## Summary based on repository changelog/release history

From the repository history and `CarTSmD-v1.0.0` release notes, the software evolution can be summarized as:

1. **Core inclinometer foundation**
   - Initial ESP32-S3 UI and sensor integration
   - Roll/pitch pipeline and live UI updates

2. **OTA workflow added and integrated in UI**
   - OTA update path added
   - UI updated to expose OTA controls and status feedback

3. **Wi-Fi connection flow maturing**
   - Dedicated Wi-Fi manager class added
   - New Wi-Fi connection screen implemented
   - State handling and callback behavior refined across several fixes

4. **Diagnostics and robustness improvements**
   - Remote diagnostics component introduced
   - I2C timeout adjustments and acceleration-cycle timing fixes
   - General bug fixes around initialization order, locking, and guard conditions

This gives a clear trajectory: **from single-purpose UI + sensor app to a more complete embedded platform** with updateability, connectivity, and serviceability.

{{< figure src="/images/PajeroProjects/ESP32TouchScreen/ScreenDesign_noBckg.png" alt="Software architecture and UI overview" caption="Current UI/software overview from the ESP32-S3 implementation" width="100%" >}}

## Example diagram: software block overview

{{< mermaid >}}
flowchart LR
	IMU[IMU QMI8658] --> SYS[System / FreeRTOS]
	UI[LVGL UI Layer] <--> DISP[Display Component]
	SYS --> UI
	WIFI[WiFi Manager] --> OTA[OTA Updater]
	UI --> OTA
	OTA --> SYS
	DIAG[Remote Diagnostics] --> SYS
{{< /mermaid >}}

## Example diagram: OTA trigger and feedback

{{< mermaid >}}
sequenceDiagram
	actor User
	participant UI as Display/UI
	participant SYS as System
	participant OTA as OTAUpdater
	participant NET as WiFi/HTTP

	User->>UI: Tap "Update"
	UI->>SYS: Request OTA start
	SYS->>OTA: Trigger update task
	OTA->>UI: Status = "Starting"
	OTA->>NET: Download firmware
	OTA->>UI: Status = "Updating"
	OTA->>SYS: Flash + validate
	OTA->>UI: Status = "Rebooting"
	SYS->>SYS: Restart device
{{< /mermaid >}}


# 3D Design

This chapter will cover the mechanical design required to mount and protect the display in the vehicle.

## Planned sections

- Packaging constraints and measurements from the dashboard area
- Bracket concept iterations (v1, v2, v3)
- Material selection (temperature resistance, vibration behavior)
- Print settings and post-processing
- Final assembly design with cable routing and service access

{{< figure src="/images/PajeroProjects/ESP32TouchScreen/BracketPrototype_placeholder.png" alt="3D bracket placeholder" caption="Bracket prototype iteration (placeholder)" width="100%" >}}

## Example diagram: 3D design iteration flow

{{< mermaid >}}
flowchart TD
	A[Measure dashboard space] --> B[Create CAD concept]
	B --> C[3D print prototype]
	C --> D[Test fit in vehicle]
	D --> E{Fit OK?}
	E -- No --> B
	E -- Yes --> F[Finalize bracket + cover]
{{< /mermaid >}}


# Integration into the Vehicle

This chapter documents how the hardware and software are integrated into the vehicle electrical and physical environment.

## Planned sections

- Power strategy (ignition-switched vs permanent supply)
- Grounding and noise considerations in automotive environment
- Mounting location and visibility/safety constraints
- Wiring loom interface and connector strategy
- Boot, reconnect, and OTA behavior in real-world usage

{{< figure src="/images/PajeroProjects/ESP32TouchScreen/VehicleIntegration_placeholder.png" alt="Vehicle integration placeholder" caption="Installed unit and wiring integration point (placeholder)" width="100%" >}}

## Example diagram: vehicle integration overview

{{< mermaid >}}
flowchart LR
	BAT[Vehicle Battery] --> REG[DC/DC + Protection]
	REG --> ESP[ESP32-S3 Touch Unit]
	ESP --> UI[Driver UI]
	ESP <--> WIFI[Garage/Home WiFi]
	ESP --> SENS[IMU + Optional Sensors]
{{< /mermaid >}}


# Conclusion

The ESP32-S3 version is the transition point from a standalone inclinometer into a reusable in-vehicle embedded platform.
With OTA updates, Wi-Fi connectivity, and diagnostics in place, the next work package is primarily mechanical and integration-focused: complete the 3D mounting solution, wire it cleanly into the vehicle, and validate long-term behavior under real driving conditions.


---

If you found this project useful, consider supporting my work:
[☕ Buy me a coffee][buymeacoffee]

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida
[esp32_s3_post]: /posts/esp32s3inclinometerwithtouchscreen/
