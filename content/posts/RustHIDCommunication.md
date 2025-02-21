---
title: "Rust HID packet communication"
date: "2025-02-18"
slug: "RustHIDCommunication"
tags: ["SimRacing", "Software"]
categories: ["Software","Rust","USB"] 
series: ["EASportsWRC datalogger Rust"]
draft: false
---
### Previous Work
This is a continuation of the work I developed previously in [Rust UDP packet receiver & Handler][RUST UDP Communication]

### Introduction and Motivation
As I stated in my previous post, I needed a new interface between EASportsWRC and my new revamped homemade rally sim shifter.
The shifter connects to the computer via a USB interface and is defined as a [HID device][HIDDescription], The rust program receives game information via UDP and then sends it out via USB. The USB bandwidth is more limited than the UDP packets so I had to reduce the signals received from UDP, which also made sense because for the moment Im only interested in a single signal.

### Concept
Overall the target is simple, receive data from UDP, chose what I need, send it as a HID message.


```goat 
Game              |Rust Client                                 | Physical
                  
                  |                                            |                    
  
.-----------.     |  .------------.    .---------------.       |     .-------------.
|EASportsWRC|        |UDP Listener|    |HID Interaction|             |SimRacing Hub|
'-----------'     |  '------------'    '---------------'       |     '-------------'
      |                    |                   |                            |       
      |Raw Packet |        |                   |               |            |       
      |------------------->|                   |                            |       
      |           |        |                   |               |            |       
      |                    |Telemetry          |                            |       
      |           |        |------------------>|               |            |       
      |                    |                   |                            |       
      |           |        |                   |Selected Gear  |            |       
      |                    |                   |--------------------------->|       
      |           |        |                   |               |            |       
      |                    |  GearUp   GearDown|                            |       
      |<----------+--------------------------------------------+------------|       
      |           |        |                   |               |            |
.-----------.        .------------.    .---------------.             .-------------.
|EASportsWRC|     |  |UDP Listener|    |HID Interaction|       |     |SimRacing Hub|
'-----------'        '------------'    '---------------'             '-------------'

```
Lets go through this sequence diagram.
Firstly, EASportsWRC sends out a UPD packet containg the game telemetry (I have gone into more detail how this works in previous [posts][udpeasportswrc])
In my rust client, I run two important threads:

#### UDP Listener -> `start_udp_listener(tx: mpsc::Sender<TelemetryData>)`
#### HID Interaction  -> `cyclic_hid_interaction(mut device: HidDevice, mut rx: mpsc::Receiver<TelemetryData>)`

[RUST UDP Communication]: /posts/RustUDPPacketHandler
[HIDDescription]: https://en.wikipedia.org/wiki/Human_interface_device
[udpeasportswrc]:/posts/udpeasportswrc/