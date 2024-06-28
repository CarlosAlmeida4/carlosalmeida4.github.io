---
title: "UDP link to EA Sports WRC"
date: "2024-06-23"
slug: "UDPEASportsWRC"
tags: ["SimRacing", "Software"]
categories: ["Software"]
series: ["UDPDatalogger"]
draft: false
---

# How to get access to the data
EA Sports WRC was a completely new development using unreal engine, which I guess meant that all software from dirt rally to a new platform.
Therefore, it took some patches until the development team was able to push a way for users to get access to data.
Before the "official" data, some very smart people developed a [patch].

I decided to wait until the oficial release, which came out during [November 2023].
You can read the original EA documentation, but the overall picture is the following

Theres a config.json where you configure the UDP packets that will be sent out
The packets definition is stored on another json file with a custom name, I called them CustomXYZ:

  - CustomStart.json
  - CustomResume.json
  - CustomPause.json
  - CustomEnd.json
  - CustomUpdate.json

For the current time Im only using the CustomUpdate structure because I noticed that the other packets were always being sent (so I had no way to use them to know Im in the start end or in the pause menu), this is one of the bugs I still need to figure out.

My json packet description
### CustomUpdate.json
```bash {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
{
	"versions":
	{
		"schema": 1,
		"data": 3
	},
	"id": "CustomUpdate",
	"header":
	{
		"channels": []
	},
	"packets": [
		{
			"id": "session_update",
			"channels": [
				"packet_4cc",
				"packet_uid",
				"shiftlights_fraction",
				"shiftlights_rpm_start",
				"shiftlights_rpm_end",
				"shiftlights_rpm_valid",
				"vehicle_gear_index",
				"vehicle_gear_index_neutral",
				"vehicle_gear_index_reverse",
				"vehicle_gear_maximum",
				"vehicle_speed",
				"vehicle_transmission_speed",
				"vehicle_position_x",
				"vehicle_position_y",
				"vehicle_position_z",
				"vehicle_velocity_x",
				"vehicle_velocity_y",
				"vehicle_velocity_z",
				"vehicle_acceleration_x",
				"vehicle_acceleration_y",
				"vehicle_acceleration_z",
				"vehicle_left_direction_x",
				"vehicle_left_direction_y",
				"vehicle_left_direction_z",
				"vehicle_forward_direction_x",
				"vehicle_forward_direction_y",
				"vehicle_forward_direction_z",
				"vehicle_up_direction_x",
				"vehicle_up_direction_y",
				"vehicle_up_direction_z",
				"vehicle_hub_position_bl",
				"vehicle_hub_position_br",
				"vehicle_hub_position_fl",
				"vehicle_hub_position_fr",
				"vehicle_hub_velocity_bl",
				"vehicle_hub_velocity_br",
				"vehicle_hub_velocity_fl",
				"vehicle_hub_velocity_fr",
				"vehicle_cp_forward_speed_bl",
				"vehicle_cp_forward_speed_br",
				"vehicle_cp_forward_speed_fl",
				"vehicle_cp_forward_speed_fr",
				"vehicle_brake_temperature_bl",
				"vehicle_brake_temperature_br",
				"vehicle_brake_temperature_fl",
				"vehicle_brake_temperature_fr",
				"vehicle_engine_rpm_max",
				"vehicle_engine_rpm_idle",
				"vehicle_engine_rpm_current",
				"vehicle_throttle",
				"vehicle_brake",
				"vehicle_clutch",
				"vehicle_steering",
				"vehicle_handbrake",
				"game_total_time",
				"game_delta_time",
				"game_frame_count",
				"stage_current_time",
				"stage_current_distance",
				"stage_length"
			]
		}
	]
}

```

# How to acquire and handle the data
Now that the game should be sending out the packet via UDP, we can check it with a network analyzer tool
As expected, wireshark shows the packets being sent out.

{{< figure src="/images/WiresharkWRC.png" alt="Wireshark packets" caption="Wireshark packets" width="100%" >}}

To get the information on the data frame we need to
- Setup a UDP listener socket
- split the data frame into usable variables

First, information storage needs to be defined
    
```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
struct EAtelemetry_data_t {
	
	int gear;
	
	float shiftlightFraction;
	float shiftlightStart;
	float shiftlightEnd;
	float VehSpeed;
	float VehTransSpeed;
	float VehPosX;
	float VehPosY;
	float VehPosZ;
	float VehSpdX;
	float VehSpdY;
	float VehSpdZ;
	float VehAccelX;
	float VehAccelY;
	float VehAccelZ;
	float VehLeftDirX;
	float VehLeftDirY;
	float VehLeftDirZ;
	float VehFwrdDirX;
	float VehFwrdDirY;
	float VehFwrdDirZ;
	float VehUpDirX;
	float VehUpDirY;
	float VehUpDirZ;
	float HubVertPosBL;
	float HubVertPosBR;
	float HubVertPosFL;
	float HubVertPosFR;
	float HubVertSpdBL;
	float HubVertSpdBR;
	float HubVertSpdFL;
	float HubVertSpdFR;
	float WheelSpeedBL;
	float WheelSpeedBR;
	float WheelSpeedFL;
	float WheelSpeedFR;
	float stear;
	float clutch;
	float brake;
	float throttle;
	float rpm;
	float rpm_idle;
	float max_rpm;
	float handbrake;
	float current_time;
	float current_minutes;
	float current_seconds;
	float game_total_time;
	float game_delta_time;
	float game_frame_count;
	float brake_temp_bl;
	float brake_temp_br;
	float brake_temp_fl;
	float brake_temp_fr;

	double track_length;
	double lap_distance;
};

```
    
Once that is defined, its time to create the UDP connection
The communication is started by the startClient method:

```cpp {class="EASportsWRC" id="my-codeblock" lineNos=inline tabWidth=2}
int EASportsWRC::startClient()
{
	if (!isRunning_b)
	{
		// Initialize Winsock
		WSADATA wsaData;
		if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
			std::cerr << "Failed to initialize Winsock" << std::endl;
			//return 1;
		}

		// Create a UDP socket
		serverSocket = socket(AF_INET, SOCK_DGRAM, 0);
		if (serverSocket == INVALID_SOCKET) {
			std::cerr << "Error creating socket" << std::endl;
			WSACleanup();
			return 1;
		}

		// Set up the server address
		sockaddr_in serverAddr;
		serverAddr.sin_family = AF_INET;
		serverAddr.sin_addr.s_addr = INADDR_ANY;
		serverAddr.sin_port = htons(PORT_i);

		// Bind the socket to the specified port
		if (bind(serverSocket, (const sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
			std::cerr << "Error binding socket" << std::endl;
			closesocket(serverSocket);
			WSACleanup();
			return 1;
		}

		std::cout << "UDP server listening on port " << PORT_i << std::endl;
		isRunning_b = true;

		m_NetworkThread = std::thread([this]() { receiveData(); });

	}

}
```
The method receiveData() is started when the UDP connection is achieved and runs until the app is stopped.
The cyclic time is defined by the UDP packet configuration

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
void EASportsWRC::receiveData()
{
	while (isRunning_b)
	{
		// Wait for incoming messages
		char buffer[265];
		uint8_t uintBuffer[265];
		sockaddr_in clientAddr;
		int clientAddrLen = sizeof(clientAddr);
		int bytesRead = recvfrom(serverSocket, buffer, sizeof(buffer), 0, (sockaddr*)&clientAddr, &clientAddrLen);
		//int bytesRead = recvfrom(serverSocket, uintBuffer, sizeof(uintBuffer), 0, (sockaddr*)&clientAddr, &clientAddrLen);

		if (bytesRead == SOCKET_ERROR) {
			std::cerr << "Error receiving message" << std::endl;
		}
		else {
			//buffer[bytesRead] = '\0';
			//std::cout << "Size of buffer" << bytesRead << std::endl;
			if (bytesRead <= 264) //Full size packet for Dirt Rally 2
			{
				//check if firts activation
				if (!GetOnStage()) StartStage();
				memcpy(UDPReceiveArray.data(), buffer, sizeof(buffer));
				//std::cout << "Received message from client: " << l_DirtTwo.UDPReceiveArray.data() << std::endl;
				HandleArray();
			}
			else
			{
				std::cout << "Received package with a different size " << sizeof(buffer) << std::endl;
			}
		}
	}

}
```

# How to read the telemetry

To de-serialize the dataframe, the following enum is defined:

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
enum class EAoffset_t : uint16_t {
	FourCC = 0,
	packet_uid = 4,
	shiftlights_fraction = 12,
	shiftlights_rpm_start = 16,
	shiftlights_rpm_end = 20,
	shiftlights_rpm_valid = 24,
	vehicle_gear_index = 25,
	vehicle_gear_index_neutral = 26,
	vehicle_gear_index_reverse = 27,
	vehicle_gear_maximum = 28,
	vehicle_speed = 29,
	vehicle_transmission_speed = 33,
	vehicle_position_x = 37,
	vehicle_position_y = 41,
	vehicle_position_z = 45,
	vehicle_velocity_x = 49,
	vehicle_velocity_y = 53,
	vehicle_velocity_z = 57,
	vehicle_acceleration_x = 61,
	vehicle_acceleration_y = 65,
	vehicle_acceleration_z = 69,
	vehicle_left_direction_x = 73,
	vehicle_left_direction_y = 77,
	vehicle_left_direction_z = 81,
	vehicle_forward_direction_x = 85,
	vehicle_forward_direction_y = 89,
	vehicle_forward_direction_z = 93,
	vehicle_up_direction_x = 97,
	vehicle_up_direction_y = 101,
	vehicle_up_direction_z = 105,
	vehicle_hub_position_bl = 109,
	vehicle_hub_position_br = 113,
	vehicle_hub_position_fl = 117,
	vehicle_hub_position_fr = 121,
	vehicle_hub_velocity_bl = 125,
	vehicle_hub_velocity_br = 129,
	vehicle_hub_velocity_fl = 133,
	vehicle_hub_velocity_fr = 137,
	vehicle_cp_forward_speed_bl = 141,
	vehicle_cp_forward_speed_br = 145,
	vehicle_cp_forward_speed_fl = 149,
	vehicle_cp_forward_speed_fr = 153,
	vehicle_brake_temperature_bl = 157,
	vehicle_brake_temperature_br = 161,
	vehicle_brake_temperature_fl = 165,
	vehicle_brake_temperature_fr = 169,
	vehicle_engine_rpm_max = 173,
	vehicle_engine_rpm_idle = 177,
	vehicle_engine_rpm_current = 181,
	vehicle_throttle = 185,
	vehicle_brake = 189,
	vehicle_clutch = 193,
	vehicle_steering = 197,
	vehicle_handbrake = 201,
	game_total_time = 205,
	game_delta_time = 209,
	game_frame_count = 213,
	stage_current_time = 221,
	stage_current_distance = 225,
	stage_length = 233

};
```



[patch]: https://www.overtake.gg/downloads/wrc-telemetry-patch.38991/
[November 2023]: https://answers.ea.com/t5/Guides-Documentation/EA-SPORTS-WRC-How-to-use-User-Datagram-Protocol-UDP-on-PC/m-p/13178407/thread-id/1