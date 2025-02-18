---
title: "Rust HID packet communication"
date: "2025-02-18"
slug: "RustHIDCommunication"
tags: ["SimRacing", "Software"]
categories: ["Software","Rust","USB"] 
series: ["EASportsWRC datalogger Rust"]
draft: true
---
### Previous Work
This is a continuation of the work I developed previously in [Rust UDP packet receiver & Handler][RUST UDP Communication]

### Introduction and Motivation
As I stated in my previous post, I needed a new interface between EASportsWRC and my new revamped homemade rally sim shifter.
The shifter connects to the computer via a USB interface and is defined as a [HID device][HIDDescription], The rust program receives game information via UDP and then sends it out via USB. The USB bandwidth is more limited than the UDP packets so I had to reduce the signals received from UDP, which also made sense because for the moment Im only interested in a single signal.

### Concept
Overall the target is simple, receive data from UDP, chose what I need, send it as a HID message.

```goat 
.---------------.     .-.     .---.
| UDP Telemetry +--->| 1 |<---+ B |
'---------------'     '-'     '---'
```
```goat 
+-------------------+                           ^                      .---.
|    A Box          |__.--.__    __.-->         |      .-.             |   |
|                   |        '--'               v     | * |<---        |   |
+-------------------+                                  '-'             |   |
                       Round                                       *---(-. |
  .-----------------.  .-------.    .----------.         .-------.     | | |
 |   Mixed Rounded  | |         |  / Diagonals  \        |   |   |     | | |
 | & Square Corners |  '--. .--'  /              \       |---+---|     '-)-'       .--------.
 '--+------------+-'  .--. |     '-------+--------'      |   |   |       |        / Search /
    |            |   |    | '---.        |               '-------'       |       '-+------'
    |<---------->|   |    |      |       v                Interior                 |     ^
    '           <---'      '----'   .-----------.              ---.     .---       v     |
 .------------------.  Diag line    | .-------. +---.              \   /           .     |
 |   if (a > b)     +---.      .--->| |       | |    | Curved line  \ /           / \    |
 |   obj->fcn()     |    \    /     | '-------' |<--'                +           /   \   |
 '------------------'     '--'      '--+--------'      .--. .--.     |  .-.     +Done?+-'
    .---+-----.                        |   ^           |\ | | /|  .--+ |   |     \   /
    |   |     | Join        \|/        |   | Curved    | \| |/ | |    \    |      \ /
    |   |     +---->  o    --o--        '-'  Vertical  '--' '--'  '--  '--'        +  .---.
 <--+---+-----'       |     /|\                                                    |  | 3 |
                      v                             not:line    'quotes'        .-'   '---'
  .-.             .---+--------.            /            A || B   *bold*       |        ^
 |   |           |   Not a dot  |      <---+---<--    A dash--is not a line    v        |
  '-'             '---------+--'          /           Nor/is this.            ---
```


[RUST UDP Communication]: /posts/RustUDPPacketHandler
[HIDDescription]: https://en.wikipedia.org/wiki/Human_interface_device