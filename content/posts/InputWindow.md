---
title: "Driver Input widget"
date: "2024-07-08"
slug: "EASportsWRCDriverInput"
tags: ["SimRacing", "Software"]
categories: ["Software"]
series: ["EASportsWRC datalogger"]
draft: false
---

### Earlier Work
To have an overview on this project, take a look at the [project webpage]


# What should be displayed
The driver has a small set of analog inputs that can be used to provide feedback on the exact position of their inputs:
- Accelerator pedal (0 to 1)
- Brake pedal 		(0 to 1)
- Clutch pedal 		(0 to 1)
- Steering input 	(-1 to 1)

# Accessing the data
As I've shared in the project mentioned above, my application is based on the Walnut application framework which in itself is based on [Dear ImGui]
The `data` is available via a public variable from the EASportsWRC class.

Initialization of the EASportsWRC object
 ```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
EASportsWRC l_EASportsWRC("127.0.0.1", 20782);
 ```
And access to the last values received via UDP link:
 ```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
VehicleSpeed = l_EASportsWRC.data.VehSpeed;
 ```
  
# Creating the User Interface layout

The widget is represented by the method `DriverInputStatus()` under the DataClientLayer class, the implementation looks like this:
```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
void DataClientLayer::DriverInputsStatus()
{
	ImGui::Begin("Driver Inputs",&m_ShowDriverInputStatus);
	char buf[32];

	//*******************************Throttle
	sprintf(buf, "%d/%d", (int)(l_EASportsWRC.data.throttle*100), 100);
	ImGui::ProgressBar(l_EASportsWRC.data.throttle, ImVec2(0.0f, 0.0f),buf);
	ImGui::SameLine(0.0f, ImGui::GetStyle().ItemInnerSpacing.x);
	ImGui::Text("Accel");
	//*******************************Brake
	sprintf(buf, "%d/%d", (int)(l_EASportsWRC.data.brake * 100), 100);
	ImGui::ProgressBar(l_EASportsWRC.data.brake, ImVec2(0.0f, 0.0f), buf);
	ImGui::SameLine(0.0f, ImGui::GetStyle().ItemInnerSpacing.x);
	ImGui::Text("Brake");
	//*******************************Clutch
	sprintf(buf, "%d/%d", (int)(l_EASportsWRC.data.clutch * 100), 100);
	ImGui::ProgressBar(l_EASportsWRC.data.clutch, ImVec2(0.0f, 0.0f), buf);
	ImGui::SameLine(0.0f, ImGui::GetStyle().ItemInnerSpacing.x);
	ImGui::Text("Clutch");
	//*******************************Handbrake
	sprintf(buf, "%d/%d", (int)(l_EASportsWRC.data.handbrake * 100), 100);
	ImGui::ProgressBar(l_EASportsWRC.data.handbrake, ImVec2(0.0f, 0.0f), buf);
	ImGui::SameLine(0.0f, ImGui::GetStyle().ItemInnerSpacing.x);
	ImGui::Text("Handbrake");
	//*******************************Steering
	sprintf(buf, "%d", (int)(l_EASportsWRC.data.stear));
	ImGui::SliderFloat("Steering", &l_EASportsWRC.data.stear, -1.0f, 1.0f, "%.3f", 1);
	
	ImGui::End();
}
```
And the UI output is:
{{< figure src="/images/DriverInputs.png" alt="Driver Input" caption="Driver Input" width="50%" >}}
The code snippets for throttle, brake, clutch and handbrake are all equal since they can all be analog inputs which range from a resting position (0) to a fully pressed position (1). The best way to represent this was by using `ImGui::ProgressBar` widget, which represents quite well this type of inputs

For steering the resting position is with the wheel aligned (0), and rotating fully to the right represents -1 and to the left 1
In this case, I felt that the best representation was a slider float ranging from -1 to 1, `ImGui::SliderFloat` is a good option for such a case

The window control can be achieved by providing a window state flag to the window, this is done by the class public flag `m_ShowDriverInputStatus', which can then be used in other routine to change and to know the current status of this window

Everytime the UI is rendered, this flag is evaluated and the method is called.
```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
void DataClientLayer::OnUIRender()
{
    ...    
    if (m_ShowDriverInputStatus) DriverInputsStatus();
    ...
}

```

[Dear ImGui]:https://www.dearimgui.com/
[project webpage]:/projects/ea-sports-wrc-datalogger/
