---
title: "Touchscreen Inclinometer - Fix Updated and new Features"
date: "2026-07-17"
slug: "CarTouchScreeMiniDisplayV120"
tags: ["LVGL", "Software","ESP32","Car","Vehicle","OTA",]
categories: ["Software"]
series: ["Pajero"]
draft: true
---
# Whats new

This is a continuation of my [incinometer][CarInclinometerProject], where I talk about my objective, the software developed and also how I intregated it into my beloved Pajero. 

After this work, I still feeled that there were multiple improvements to achieve and some very annoying bugs that dont show up on the test bench and I cannot diagnose due to how random they are.

So, what did I do? I let scope creep get the better of me, instead of only fixing the issues, I increase the features :smile:

Between the version I used for vehicle integration [CarTSmD-v1.0.0], and the current [CarTSmD-v1.2.0] version, alot has changed, here's the list:

1. [Fixed multiple issues that could lead to panic and resets]()
2. [Added a new Inclinometer screen and zero out functionality]()
3. [Created a new configuration and wifi screens]()
4. [Added Wifi provisioning and wifi management state machine]()
5. [Updated OTA server access]()
6. [Version String]()

## Fix the panic!

During vehicle integration testing, I encountered random resets and panic events that were incredibly hard to diagnose. They didn't happen consistently on the test bench, which made debugging a nightmare. After some investigation, I discovered the culprit: **stack overflow caused by excessive logging in the WiFi event handlers**.

The problem was in the `WifiManager` component. Every time a WiFi event occurred (scanning, connecting, disconnecting), the event handler would trigger detailed logging with formatted strings. Since event handlers typically run in interrupt or callback contexts with limited stack space, the accumulation of log buffers and format operations would overflow the stack, causing the system to panic and reset.

The fix was straightforward but effective:
- **Reduced logging verbosity** in `WifiManager::storeAPPoints()` by replacing detailed per-AP logs with a single summary log after scanning completes
- **Simplified event handler logging** by removing format strings and complex operations from disconnect and scan-done event handlers
- **Minimized stack usage** in critical event-driven code paths

This approach follows the principle that logging in interrupt/event contexts should be as lightweight as possible. Instead of logging every detail, we now log a summary after the critical operation completes, keeping the event handler itself lean and stack-efficient.

After these changes, the random panics disappeared completely, and the system has been stable through extensive vehicle testing.

## New Inclinometer and where is ground

### Where is Ground?

One of the issues that I didnt consider before goint into the car was that the actual display would be slightly tilted, which in hindsight makes sense, because the displays are skewed towards the better angle to the user.

Another reason is the mounting, its plastic and has a bit of play, which means that the screen might appear a bit tilted, but not much that I would consider a brand new plastic support.

**So, how did I fix this?**

The right answer was to include a zeroing functionality to the inclinometer, so that in any moment in time, I can include an angle offset to the inclinometer angles.

In the code, it looks like this:
```cpp
// Calculate raw pitch and roll from accelerometer
out.roll  = atan(mask) * RAD_TO_DEG;
out.pitch = atan2(acc.z, acc.x) * RAD_TO_DEG;

// Apply offset (zero out the reference position)
out.roll  -= RPOffset.roll;
out.pitch -= RPOffset.pitch;
```
the `RPOffset` is nothing more than a set of roll and pitch I grabed while on a horizontal plane and stored in the NVM.

The data flow diagram for this use case looks like this:

{{< mermaid >}}
graph TD
    A["Sensor Initialization<br/>init()"] --> B["Load Stored Offset<br/>getStoredOffset()"]
    B --> C["RPOffset = loaded values - or 0,0,0 if not found"]
    
    D["User Calibration<br/>setInclinometerOffset()"] --> E["Reset RPOffset to 0,0"]
    E --> F["Read Current Position<br/>getPitchAndRoll"]
    F --> G["Capture as Offset<br/>RPOffset = rp"]
    G --> H["Store to NVS<br/>setStoredOffset"]
    H --> I["Next Power-on:<br/>Offset is reloaded"]
    
    J["Continuous Operation<br/>read_sensor_data()"] --> K["Get Raw Accelerometer Data"]
    K --> L["Calculate Pitch & Roll"]
    L --> M["Apply Offset<br/>angle -= RPOffset"]
    M --> N["Output Corrected Angle"]
    
    C -.-> M
    I -.-> C
{{< /mermaid >}}

{{< mermaid >}}
graph LR
    A["Calibration<br/>setStoredOffset"] --> B["NVS Handle<br/>QMIOffset"]
    B --> C["int32 Roll"]
    B --> D["int32 Pitch"]
    C --> E["ESP32 Flash"]
    D --> E
    
    F["Boot/Init"] --> G["getStoredOffset"]
    G --> B
    B --> H["Restore to Memory<br/>RPOffset"]
{{< /mermaid >}}

So in short, the flash storage and retrieve is done in the following way:

**Writing Offset to Flash** (`setStoredOffset`):
1. Open NVS namespace `"QMIOffset"` in READ_WRITE mode
2. Cast float roll/pitch to int32_t
3. Store using `nvs_set_i32()` with keys `"QMIOffsetRoll"` and `"QMIOffsetPitch"`
4. Commit changes to flash using `nvs_commit()`
5. Close NVS handle

**Reading Offset from Flash** (`getStoredOffset`):
1. Open NVS namespace `"QMIOffset"` in READ_ONLY mode
2. Retrieve int32_t values using `nvs_get_i32()` for both roll and pitch
3. Cast back to float
4. Return as `std::optional<RollPitch>` (null if not found)
5. Close NVS handle

For this use case, the `std::optional<>` is very useful. It allows to safely handle cases where a value might not exist, very similar to what Rust does with its `Option` type.

In most typical embedded approaches (where C is king), you'd have to handle "no value" situations in messy ways: returning a null pointer (dangerous with value types), using a error value, or returning a boolean and using an output parameter. With `std::optional`, the intent is crystal clear—the value either exists or it doesn't.

In our inclinometer offset case, when the system boots for the first time, there's no stored offset in flash yet. The `getStoredOffset()` method returns `std::optional<RollPitch>`:

```cpp
std::optional<RollPitch> getStoredOffset() {
    RollPitch storedOffset;
    esp_err_t nvs_err;

    nvs_handle_t nvsHandle;

    nvs_err = nvs_open("QMIOffset",NVS_READONLY,&nvsHandle);
    if (nvs_err != ESP_OK) {
        ESP_LOGE(QMI8658C_TAG, "Error (%s) opening NVS handle!", esp_err_to_name(nvs_err));
        return std::nullopt;
    }

    ESP_LOGI(QMI8658C_TAG,"Reading Roll");
    int32_t lRoll;
    nvs_err = nvs_get_i32(nvsHandle,"QMIOffsetRoll",&lRoll);
    switch (nvs_err) {
        case ESP_OK:
            ESP_LOGI(QMI8658C_TAG, "Read Roll = %" PRIu32, lRoll);
            break;
        case ESP_ERR_NVS_NOT_FOUND:
            ESP_LOGW(QMI8658C_TAG, "The value is not initialized yet!");
            break;
        default:
            ESP_LOGE(QMI8658C_TAG, "Error (%s) reading!", esp_err_to_name(nvs_err));
    }

    ESP_LOGI(QMI8658C_TAG,"Reading Pitch");
    int32_t lPitch;
    nvs_err = nvs_get_i32(nvsHandle,"QMIOffsetPitch",&lPitch);
    switch (nvs_err) {
        case ESP_OK:
            ESP_LOGI(QMI8658C_TAG, "Read Pitch = %" PRIu32, lPitch);
            break;
        case ESP_ERR_NVS_NOT_FOUND:
            ESP_LOGW(QMI8658C_TAG, "The value is not initialized yet!");
            break;
        default:
            ESP_LOGE(QMI8658C_TAG, "Error (%s) reading!", esp_err_to_name(nvs_err));
    }

    if(nvs_err!=ESP_OK)
    {
        nvs_close(nvsHandle);
        return std::nullopt;
    }
    else
    {
        storedOffset.pitch = static_cast<float>(lPitch);
        storedOffset.roll = static_cast<float>(lRoll);
        nvs_close(nvsHandle);
        return std::optional<RollPitch>(storedOffset);
    }
}
```

Then on the caller, instead of if cases handling the exceptions, in one line I can define the expected reaction to all use case:

```cpp
    {
        std::lock_guard<std::mutex> lock(RPOffsetMutex);
        RPOffset = getStoredOffset().value_or(RollPitch{0,0,0});
    }
```

This is much more expressive than older approaches. Just like in Rust where you must explicitly handle the `Some` and `None` cases, C++'s `std::optional` makes the compiler aware that the value might not exist, helping prevent bugs where you accidentally use an uninitialized value. It's defensive programming that reads naturally.

### New Inclinometer screen

To introduce the reset button, I also wanted to create a new inclinometer screen, the last one had a very good performance and gave me the exact information I wanted, from a engineering perspective, thats good enough, but its not pretty and not flashy.
So I entirely created a new screen. I wanted to have something that shows a small picture of my car and also look colorful. After sleeping on the topic, I came up with this design:




## New Configuration and Wifi screens

## Wifi Provisioning and why I didnt do this earlier

## OTA cheat

## This was just dumb


## Conclusion


If you found this project interesting or useful, please consider supporting my work:

[☕ Buy me a coffee][buymeacoffee]

The code is available on GitHub and released under MIT license - feel free to use, modify, and share!

---


[CarTSmD-v1.2.0]:https://github.com/CarlosAlmeida4/CarTouchScreenMiniDisplay_ESP32S3/releases/tag/CarTSmD-v1.2.0
[CarTSmD-v1.0.0]:https://github.com/CarlosAlmeida4/CarTouchScreenMiniDisplay_ESP32S3/releases/tag/CarTSmD-v1.0.0
[CarInclinometerProject]:/projects/pajerocentralscreenminidashboard/
[IntegrationHell]: https://wiki.c2.com/?IntegrationHell
[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida
[CarInclinometer]: /posts/carinclinometer/
[WaveshareESP32S3]: https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75
[LVGL]: https://docs.lvgl.io/
[SquareLine]: https://www.squareline.io/
[RPi debug Probe]:https://www.raspberrypi.com/documentation/microcontrollers/debug-probe.html