---
title: "ESP32 S3 Touchscreen Inclinometer - Advanced Dashboard"
date: "2026-02-10"
slug: "ESP32S3InclinometerwithTouchscreen"
tags: ["LVGL", "Software","ESP32","Car","Vehicle","OTA",]
categories: ["Software"]
series: ["Pajero"]
draft: false
---
# The Next Generation - ESP32-S3 Touchscreen Inclinometer

While I built my earlier [Car Inclinometer][CarInclinometer] project with the RP2040, I ran into a couple of setbacks.
The previous version was great, but I wanted something more stable and capable. If you look at that repo, you will find that I actually updated the project to use a RP2350.

Even with this update I was ending in some weird states which were making developing the software quite challenging.
I was having random deadlock situations caused by the i2c bus that I was unable to debug, since the board I was using did not provide a way to connect the [Raspberry Pi debug probe][RPi debug Probe]. 

It was one of those ***"It can be running for three days without an issue, but if I look at it funny, it will break"***. That can be demoralizing. My last ditch attempt with that board was implementing a watchdog callback. But thats not a permanent solution, so I kinda demoralized and left the project die out.

However, I still wanted to create this inclinometer, so I started the search for a substitute board. I found once again a solution via Waveshare, the [Waveshare ESP32-S3 Touch AMOLED][WaveshareESP32S3].

## The Vision

My goal was to enhance my previous approach with an embedded touchscreen system that could:
- Display real-time inclinometer data (roll and pitch)
- Provide a responsive, modern user interface
- Support future expansion with additional sensors and screens
- Be completely over-the-air updateable
- Act as a small hub for vehicle telemetry and other things I might implement 👀
- Be able to debug it
- Use C++ features to create a cleaner architecture

The result is a system built on **LVGL**, and **ESP-IDF** that makes it easy to develop custom gauges and displays for any embedded or automotive application.



## Hardware Selection

For this project, As previously mentioned, I chose the [Waveshare ESP32-S3 Touch AMOLED][WaveshareESP32S3] board as the foundation.
The ESP32-S3 is a powerhouse compared to the RP2040/RP2350 - dual-core processor, more RAM, more flash, and native WiFi/BLE support.

The board includes:
- **1.75" AMOLED display** (454x454 pixels)
- **Touch controller** (GT911)
- **IMU** (QMI8658) - the heart of our inclinometer
- **Expansion headers** for additional sensors

I designed the code to be flexible enough to support multiple sensor variants, so you can adapt it to different hardware configurations.

## Building the User Interface

{{< figure src="/images/PajeroProjects/ESP32TouchScreen/ScreenDesign_noBckg.png" alt="SquareLine Studio UI Design" caption="Touchscreen dashboard designed in SquareLine Studio" width="100%" >}}

The resulting interface features:
- **Main inclinometer display** with live roll and pitch data
- **Arc gauges** showing vehicle tilt angles (-45° to +45°)
- **Central pitch indicator** as a vertical slider
- **Real-time numeric readouts** showing actual roll and pitch values and temperature
- **Multi-page support** for future expansions (vehicle telemetry, settings, diagnostics)
- **Software Update Button** to allow for over the air updates

The beauty of this approach is that:
1. All UI changes happen in SquareLine Studio (no manual C code editing)
2. The exported code integrates seamlessly into the firmware
3. Application logic remains completely decoupled from the UI layer
4. Screens can be redesigned without touching the sensor reading logic

## Implementation Architecture



## Sensor Data Processing

Just like in the previous project, the inclinometer relies on processing accelerometer data from the IMU.
The QMI8658 on the Waveshare board provides raw acceleration vectors. By calculating the angle of the gravity vector relative to the board's axes, I can determine roll and pitch:

$$roll = \arctan\left(\frac{y}{-x}\right)$$

$$pitch = \arctan\left(\frac{z}{-x}\right)$$

These values are then scaled to fit the display gauges and text labels in real-time.

## Project Structure

The firmware is organized into clean, modular components:

```
CarTouchScreenMiniDisplay_ESP32S3/
├── main/                    # Application entry point
│   ├── tasks initialization
│   ├── sensor polling loops
│   └── main firmware logic
│
├── components/
│   ├── display/            # Display driver & LVGL integration
│   │   ├── touch input handling
│   │   └── rendering loop
│   │
│   ├── ui/                 # SquareLine Studio generated UI
│   │   ├── screens
│   │   ├── widgets
│   │   ├── fonts
│   │   └── images
│   │
│   ├── SensorLib/          # Modular sensor drivers
│   │   ├── QMI8658 IMU interface
│   │   ├── Optional: QMC6310 magnetometer
│   │   ├── Optional: LTR553 light sensor
│   │   └── Optional: DRV2605 haptic motor
│   │
│   ├── System/             # FreeRTOS task coordination
│   │   ├── sensor reading tasks
│   │   ├── display update tasks
│   │   └── system state management
│   │
│   ├── OTAUpdater/         # Over-the-air update handler
│   │   └── automatic version checking & flashing
│   │
│   └── esp_lcd_sh8601/     # LCD controller driver
│       └── hardware interface layer
│
├── SquarelineProject/      # SquareLine Studio project file
└── partitions.csv          # Flash layout for OTA updates
```

## Key Design Decisions

**1. FreeRTOS Task Architecture**  
I used FreeRTOS tasks with queue-based communication to keep the system responsive. Sensor reading, UI updates, and display rendering run in parallel without blocking.

**2. UI Separation**  
The application logic knows nothing about the UI. Screen updates happen through a clean abstraction layer. This means I can completely redesign the interface without touching the sensor code.

**3. Component-Based Sensors**  
Each sensor is self-contained. Need to add a magnetometer? Drop in the new component. Want to remove the light sensor to save power? Just rebuild without it.


## how data is shared between accel and UI

## Over-The-Air Updates

One powerful feature I built in is OTA (Over-The-Air) update capability. 
Rather than physically connecting to the device with USB every time you want to update firmware, the ESP32-S3 can automatically check for and install new versions over WiFi.

This was crucial because I wanted the device to be permanently mounted in the vehicle. 
The OTA system uses the dual-partition scheme - one active partition and one for updates - ensuring you always have a bootable system even if an update fails.

## how events are handled between UI and OTA

## Getting Started

If you want to build this yourself:

### Prerequisites
- **ESP-IDF** v5.0+ (follow the [official setup guide](https://docs.espressif.com/projects/esp-idf/))
- **Waveshare ESP32-S3 Touch AMOLED** board
- **SquareLine Studio** 1.6.0+ (optional, only if modifying the UI)
- **CMake** 3.16+

### Build & Flash
```bash
# Clone the repository
git clone https://github.com/CarlosAlmeida4/CarTouchScreenMiniDisplay_ESP32S3.git
cd CarTouchScreenMiniDisplay_ESP32S3

# Configure for ESP32-S3
idf.py set-target esp32s3
idf.py menuconfig  # Enable PSRAM, set clock to 240MHz

# Build and flash
idf.py build
idf.py -p COM3 flash monitor  # Windows (adjust COM port)
```

The device will boot up with the inclinometer screen running. Tilt your board and watch the gauges respond in real-time!

### Customizing the UI
To modify the dashboard design:
1. Open `SquarelineProject/CarTouchScreenMiniDisplay.slvs` in SquareLine Studio
2. Edit screens and components as needed
3. Export to `components/ui/` (select "ESP-IDF" target)
4. Rebuild: `idf.py build && idf.py flash`

## What's Next?

This framework is designed to be extended. Some ideas:
- **Multi-screen system**: Navigation between inclinometer, telemetry, diagnostics
- **Sensor fusion**: Combine IMU data with magnetometer for heading information
- **Data logging**: Record vehicle data to SD card or cloud
- **Custom gauges**: Build analog-style gauges, progress bars, real-time graphs
- **Vehicle Integration**: Connect to vehicle CAN bus for engine parameters

The modular architecture makes all of this possible without major rewrites.

## Conclusion

This project represents a significant step forward from my initial RP2040 inclinometer. 
The combination of LVGL, SquareLine Studio, and ESP-IDF creates a powerful platform for building custom embedded dashboards.

The modular design means you can take what I've built and adapt it for your own projects - whether it's an automotive application, industrial display, IoT dashboard, or home automation interface.

If you found this project interesting or useful, please consider supporting my work:

[☕ Buy me a coffee][buymeacoffee]

The code is available on GitHub and released under MIT license - feel free to use, modify, and share!

---

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida
[CarInclinometer]: /posts/carinclinometer/
[WaveshareESP32S3]: https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75
[LVGL]: https://docs.lvgl.io/
[SquareLine]: https://www.squareline.io/
[RPi debug Probe]:https://www.raspberrypi.com/documentation/microcontrollers/debug-probe.html