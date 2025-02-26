---
title: "Sim Racing Hub"
date: "2025-02-24"
slug: "SimRacingHub"
tags: ["SimRacing", "Software","Hardware","3dPrinting"]
categories: ["Projects"]
draft: false
---

### Table of contents
1. [Introduction](#introduction)
2. [New Features](#new-features)
3. [Hardware procurement and development](#hardware-procurement-and-development)
4. [Housing](#housing)
5. [Software](#software)
 

# Introduction <a name="Introduction"></a>
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


| Type   | Pin ID | Usage       |
|:------:|:------:|:-----------:|
|  DIO   |    0   | Shift Up    |
|  DIO   |    1   | Shift Down  |
|  DIO   |   18   | Reserved    |
|  DIO   |   19   | Reserved    |
|  DIO   |   20   | Reserved    |
|  DIO   |   21   | Reserved    |
|  ADC   |   26   | ADCReserved2|
|  ADC   |   27   | ADCReserved1|
|  ADC   |   28   | ADCHandbrake|


Having defined the IOs, I created a circuit board to handle the inputs. Of course you could just have a switch connected directly to the microcontroller input, however hardware buttons have a tendency to "bounce" quite a bit when activated. To reduce the chances of this bouncing creating multiple edge readings, I implemented a low pass filter on all digital inputs, this is always a good practice for any input really, this way you cut all high frequency interferences that might find their way to the micro input.
For the ADC signals I created just a voltage divider, this is for when I get around to create the handbrake.

As digital inputs, I sourced the following:
 * 2 x [Microswitches]
 * 1 x [Switch]
 * 1 x [Black pressure button]
 * 1 x [Red pressure button]

 The microswitches are for the shifter, the other inputs are for further developments.

With this the procurement done, I created the circuit board.
{{< figure src="/images/PCBBackSimRacingHub.jpg" alt="PCB back" caption="PCB back" width="50%" >}}
{{< figure src="/images/PCBFrontSimRacingHub.jpg" alt="PCB front" caption="PCB front with waveshare RP2040 board connected" width="50%" >}}

# Housing


# Software

This is a distributed system, some of the software runs in the PC running the game and the remaining runs in the raspberry pi.
In this post I will cover only the embedded software within the raspberry pi, I already went through the PC in this [post]

## Architecture

## HID Communication

## Display

- [Github]
- [Printables]

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[instructables]: https://www.instructables.com/A-Sequential-Gear-Shifter-for-Simracing/
[Github]: https://github.com/CarlosAlmeida4/StandaloneShifter
[Printables]: https://www.printables.com/model/585572-sim-racing-shifter
[SinRacingShifter]: projects/SimRacingShifter
[WaveshareProductLink]: https://www.waveshare.com/wiki/RP2040-LCD-1.28
[Microswitches]:https://mauser.pt/catalog/product_info.php?products_id=010-0122
[Switch]: https://mauser.pt/catalog/product_info.php?products_id=010-0172
[Black pressure button]: https://mauser.pt/catalog/product_info.php?products_id=010-0097
[Red pressure button]: https://mauser.pt/catalog/product_info.php?products_id=010-0127
[post]: /posts/rusthidcommunication