---
title: "Sim Racing Hub"
date: "2025-02-24"
slug: "SimRacingHub"
tags: ["SimRacing", "Software","Hardware","3dPrinting"]
categories: ["Projects"]
draft: true
---
## Introduction
The Sim Racing Hub is a new version of my [old Racing Shifter][SinRacingShifter], with some upgrades that I wanted to have in my old setup.
I really liked the old shift functionality, but to quote Scott Adams 
> Normal people... believe that if it ain't broke, don't fix it. Engineers believe that if it ain't broke, it doesn't have enough features yet

# New Features
### Better screen display
The old gear display was a single digit 8 bit display, I want to enhance this a little bit to make it look better, so I had to search for a new solution.
### The current gear should be the same as in game
In the previous version, the displayed gear is calculated internally by the shifter, sometimes the communication between the shifter and the PC misses, meaning that from time to time its possible that the in game gear and the displayed gear are incorrect. Since telemetry is available in game, should be easy to transmit this to the shifter... right? right?!
### Additional Inputs
Besides the shifter functionality, I also want to have room for future developments, one of them is a analog handbrake. But I also want to future proof the design, so I added more digital inputs for further implementations.

# Hardware procurement and development
With this new requirements, I started searching for new hardware solutions.
The best suited solution that I found was the [waveshare RP2040][WaveshareProductLink]. Its a RP2040 integrated with a round LCD display and GPIO headers.

{{< figure src="/images/RP2040-LCD-1.28.jpg" width="25%" >}}

The board schematic provided by waveshare shows that some of the GPIO pins are used for screen functions, so if we want to use the screen those IOs are already used up.
Besides the screen, we need the following IOs:



I've written a full [instructables] post on how to build it and you can find all the needed information there, I'll just leave the other interesting links here:


- [Github]
- [Printables]

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[instructables]: https://www.instructables.com/A-Sequential-Gear-Shifter-for-Simracing/
[Github]: https://github.com/CarlosAlmeida4/StandaloneShifter
[Printables]: https://www.printables.com/model/585572-sim-racing-shifter
[SinRacingShifter]: projects/SimRacingShifter
[WaveshareProductLink]: https://www.waveshare.com/wiki/RP2040-LCD-1.28
