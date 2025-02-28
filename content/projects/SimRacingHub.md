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
 

# Introduction
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
As mentioned previously, this is a new version of an older project, so if you want more details on the entire build, please go to the [instructables]
For this version I just altered the base and the housing for the electronics
{{< figure src="/images/DisplayCase1.jpg"  width="50%" >}}
The PCB is kept in place by snap fittings I designed in the display case face
{{< figure src="/images/DisplayCase2.jpg"  width="50%" >}}
The two holes on the bottom are for the black and red buttons, while the keys on the top give me a way to secure the face to the rest of the housing
{{< figure src="/images/DisplayCase3.jpg"  width="50%" >}}
The housing has a hole on the side for the extra switch, the back holes allow access to the PCB headers in case its needed

# Software

This is a distributed system, some of the software runs in the PC running the game and the remaining runs in the raspberry pi.
In this post I will cover only the embedded software within the RP2040, I already went through the PC in this [post].
The microcontroller has two functions:
1. Send shift request to PC
2. Show current gear in the screen

I developed the project using platformIO beacuse I enjoy the flexibility of it.
In the next sections I will go through a summary of the architecture, the HID communication and gear display.

## Architecture

The software project is structured in the following way:
* IO_inputs: Here all input handling is handled
* LCD: LCD display driver specific to the screen used
* ShifterLogic: Handles everything related with gear shifting
* UI: I used [SquareLine Studio][Squareline] to generate the UI, more on that in a following section
* UIHandler: handling init and communication with the UI
* USBComm: Handling the HID communication

The RP2040 is a multicore microcontroller, so we need to split the tasks between both cores, and also its required to share information between both cores.
The simplest way to share the information is using global variables, since in this application theres only one producer for each of the shared variables the chances to run into deadlocks is very limited.
Therefore, the Information shared between all tasks is defined within the SharedDatatype structure:
```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
typedef volatile struct SharedDatatype
{
    uint8_t ShiftUpRequest;
    uint8_t ShiftDownRequest;
    uint8_t CurrentGear;
}SharedData_t;
```
As you can see its not much, so the risk of deadlocks is not very high

## HID Communication

## Display
{{< figure src="/images/SquarlineStudioShifter.png"  width="50%" >}}

- [Github]
- [Printables]

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida
[Squareline]: https://squareline.io/
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
