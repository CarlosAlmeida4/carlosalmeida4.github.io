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

# Introduction and Motivation
As I stated in my previous post, I needed a new interface between EASportsWRC and my new revamped homemade rally sim shifter.
The shifter connects to the computer via a USB interface and is defined as a [HID device][HIDDescription], The rust program receives game information via UDP and then sends it out via USB. The USB bandwidth is more limited than the UDP packets so I had to reduce the signals received from UDP, which also made sense because for the moment Im only interested in a single signal.

# Concept
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

The UDP listener starts a UDP socket and receives the buffer from the local port to which EASportsWRC produced the dataset.
Using the sender side of a [mpsc][mpscChannel] channel the parsed packet is sent out to the HID Interaction thread.


```rust {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
use hidapi::{HidApi, HidDevice};
use core::str;
use std::io::{self, Write};
use tokio::time::{sleep,Duration};
use tokio::{net::UdpSocket as AsyncUdpSocket, task, sync::mpsc};

async fn start_udp_listener(tx: mpsc::Sender<TelemetryData>) -> io::Result<()> {
    //let mut addr = String::new();
    //println!("Please input the IP and the port:");
    //io::stdin().read_line(&mut addr)?;
    let mut addr = String::from("127.0.0.1:20782");

    addr = addr.trim().to_string(); // Remove newline characters
    let socket = AsyncUdpSocket::bind(&addr).await?;
    println!("Listening on socket {}", addr);

    let mut buf = [0u8; 1024];

    loop {
        match socket.recv_from(&mut buf).await {
            Ok((size, _src)) => {
                //println!("Received {} bytes from {}", size, src);
                if let Ok(packet) = parse_packet(&buf[..size]) {
                    // Send the packet to the HID task
                    if tx.send(packet).await.is_err() {
                        eprintln!("Failed to send packet to HID task");
                    }
                }
            }
            Err(e) => {
                eprintln!("Error receiving data: {}", e);
            }
        }
    }
}

```
On the HID side theres two threads: the `start_hid_listener`, which is responsible to look for the correct HID device, and once it finds it to connect and spawn the thread that handles the HID communication  `cyclic_hid_interaction`

```rust {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}

async fn cyclic_hid_interaction(mut device: HidDevice, mut rx: mpsc::Receiver<TelemetryData>) {
    println!("Connected to HID device. Sending UDP packets every 10ms.");

    let mut last_packet = TelemetryData {
        packet_4cc: 0,
        packet_uid: 0,
        shiftlights_fraction: 0.0,
        shiftlights_rpm_start: 0.0,
        shiftlights_rpm_end: 0.0,
        shiftlights_rpm_valid: false,
        vehicle_gear_index: 0,
        vehicle_gear_index_neutral: 0,
        vehicle_gear_index_reverse: 0,
        vehicle_gear_maximum: 0,
        vehicle_speed: 0.0,
        vehicle_transmission_speed: 0.0,
        vehicle_position_x: 0.0,
        vehicle_position_y: 0.0,
        vehicle_position_z: 0.0,
        vehicle_velocity_x: 0.0,
        vehicle_velocity_y: 0.0,
        vehicle_velocity_z: 0.0,
        vehicle_acceleration_x: 0.0,
        vehicle_acceleration_y: 0.0,
        vehicle_acceleration_z: 0.0,
        vehicle_left_direction_x: 0.0,
        vehicle_left_direction_y: 0.0,
        vehicle_left_direction_z: 0.0,
        vehicle_forward_direction_x: 0.0,
        vehicle_forward_direction_y: 0.0,
        vehicle_forward_direction_z: 0.0,
        vehicle_up_direction_x: 0.0,
        vehicle_up_direction_y: 0.0,
        vehicle_up_direction_z: 0.0,
        vehicle_hub_position_bl: 0.0,
        vehicle_hub_position_br: 0.0,
        vehicle_hub_position_fl: 0.0,
        vehicle_hub_position_fr: 0.0,
        vehicle_hub_velocity_bl: 0.0,
        vehicle_hub_velocity_br: 0.0,
        vehicle_hub_velocity_fl: 0.0,
        vehicle_hub_velocity_fr: 0.0,
        vehicle_cp_forward_speed_bl: 0.0,
        vehicle_cp_forward_speed_br: 0.0,
        vehicle_cp_forward_speed_fl: 0.0,
        vehicle_cp_forward_speed_fr: 0.0,
        vehicle_brake_temperature_bl: 0.0,
        vehicle_brake_temperature_br: 0.0,
        vehicle_brake_temperature_fl: 0.0,
        vehicle_brake_temperature_fr: 0.0,
        vehicle_engine_rpm_max: 0.0,
        vehicle_engine_rpm_idle: 0.0,
        vehicle_engine_rpm_current: 0.0,
        vehicle_throttle: 0.0,
        vehicle_brake: 0.0,
        vehicle_clutch: 0.0,
        vehicle_steering: 0.0,
        vehicle_handbrake: false,
        game_total_time: 0.0,
        game_delta_time: 0.0,
        game_frame_count: 0,
        stage_current_time: 0.0,
        stage_current_distance: 0.0,
        stage_length: 0.0,
    };

    loop {
        // Check if there's a new packet from UDP
        if let Ok(packet) = rx.try_recv() {
            last_packet = packet; // Update last received packet
        }
        //TODO! - Reduce the packets sent by only sending when theres a change
        // Prepare the message to send
        let mut output: Vec<u8> = vec![0x00]; // Report ID = 0x00
        output = create_hid_packet(&last_packet,1);
        //output.extend_from_slice(&last_packet);//TODO: ve la o que fazes aqui
        println!("Sent to HID Device: {:?}\n", &output);
        // Send to HID device
        if let Err(e) = device.write(&output) {
            eprintln!("Failed to write to device: {}", e);
        }
        
        // Read response
        let mut buf = [0u8; 5];
        match device.read(&mut buf) {
            Ok(len) => {
                println!("Received from HID Device: {:?}\n", &buf[..len]);
            }
            Err(e) => {
                eprintln!("Failed to read from device: {}", e);
            }
        }

        // Sleep asynchronously for 10ms
        //sleep(Duration::from_millis(1)).await;
    }
}

```

In this current iteration the HID packet has 5 bytes where the first byte is considered the packet ID
As this is still in development, only one packet existes `PacketID:1`, which has the following structure:

                                        
| Byte 0 | Byte 1      | Byte 2 | Byte 3 | Byte 4 |
| ------ |:-----------:|:------:|:------:|:------:|
|    1   | Current Gear| Unused | Unused | Unused |

The HID device is configured to send the received packet, so in the cyclic method we also expect that we receive the same packet that was sent out.

In the following posts I will go into detail on the embedded device software so keep on the lookout :smile:.
I will also probably create a [project post][projects] detailing the entire project

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[RUST UDP Communication]: /posts/RustUDPPacketHandler
[HIDDescription]: https://en.wikipedia.org/wiki/Human_interface_device
[udpeasportswrc]:/posts/udpeasportswrc/
[mpscChannel]:https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html
[projects]:/projects