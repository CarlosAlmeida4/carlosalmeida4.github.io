---
title: "Car Inclinometer update to RP2350 and Watchdog configuration"
date: "2025-07-20"
slug: "CarInclinometer_update"
tags: ["LVGL", "Software","RP2350","Car","Vehicle","Watchdog"]
categories: ["Software"]
series: ["Pajero"]
draft: false
---
# Introduction
This is a continuation of my last post on developing a [Car Inclinometer] Heres the what changed 
## What's Changed
* Develop rp2350 by @CarlosAlmeida4 in https://github.com/CarlosAlmeida4/CarTouchscreenMiniDisplay/pull/7
The full changelog is available in [git] 


# Hardware
Previously in my [Simulator Racing Hub][SimRacingHub] I used a similar board to the one Im using in this project.
It still is a [Waveshare RP2040] design, but with a capacitive touchscreen.
For this development, I dint't create any special hardware yet, I focused firstly on the software development.

# Watchdog code

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[project webpage]:/projects/ea-sports-wrc-datalogger/
[SimRacingHub]:/projects/SimRacingHub/
[Waveshare RP2040]:https://www.waveshare.com/wiki/RP2040-Touch-LCD-1.28
[Motion_Yaw_Roll]: https://www.formula1-dictionary.net/motions_of_f1_car.html
[Car Inclinometer]:/posts/CarInclinometer/
[gitComp]: https://github.com/CarlosAlmeida4/CarTouchscreenMiniDisplay/compare/CTMD_v001...CTMD_v010