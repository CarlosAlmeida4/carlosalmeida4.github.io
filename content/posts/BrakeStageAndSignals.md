---
title: "Brake Pedal and Brake Temperature"
date: "2024-07-19"
slug: "BrakeStageAndSignals"
tags: ["SimRacing", "Software"]
categories: ["Software"]
series: ["EASportsWRC datalogger"]
draft: false
---
### Earlier Work
To have an overview on this project, take a look at the [project webpage]

# Target

The idea with these displays is to explore what can be achieved using ImPlot available and to allow all signals to be visible.
Also, we need to figure out a way store the run so its possible to review the data at a later instance.

# Brake live data
The brake data widget shows only to performance indicators, the brake pedal position and the brake temperature for all four corners.

The temperature of the corners is shown in two different ways, first in a live graph with a variable history time, and second with a heat map picture.

The Temp v Time graphic is helpful to check what happened in the last few seconds while the heatmap allows the user to have a real time feeling of how the temperature is evolving in each corner of the vehicle.

Everything discussed inside this chapter can be found within the `DataClientLayer::BrakeData()`

{{< figure src="/images/BrakeData2.png" alt="Brake Signals" caption="Brake Signals" width="25%" >}}

## Circular buffer
To only display the last few seconds, we need to have access to signal history and not only to the last value. This could be done by accessing the `EASportsWRC` storage vector. However, this would mean using some logic to always obtain the last vector value + the determined history (user defined). Also another issue is that for a stress free display, you need to create a ImVec2 vector.
The Example from ImPlot is really helpful for this specific case, there you can find a definition for a `ScrollingBuffer`

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
// utility structure for realtime plot
struct ScrollingBuffer {
	int MaxSize;
	int Offset;
	ImVector<ImVec2> Data;
	ScrollingBuffer(int max_size = 2000) {
		MaxSize = max_size;
		Offset = 0;
		Data.reserve(MaxSize);
	}
	void AddPoint(float x, float y) {
		if (Data.size() < MaxSize)
			Data.push_back(ImVec2(x, y));
		else {
			Data[Offset] = ImVec2(x, y);
			Offset = (Offset + 1) % MaxSize;
		}
	}
	void Erase() {
		if (Data.size() > 0) {
			Data.shrink(0);
			Offset = 0;
		}
	}
};
```
The `ScrollingBuffer.Data` contains a X and a Y field, where X is time and Y is the displayed data.
Using this structure allows for a stress free usage since it implements everything that we already needed, now adding 
a new point is as simple as calling the `.AddPoint()` method.

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
//Use always the lastest value from the stored vector
current_time = l_EASportsWRC.TelemetryData_v.current_time.back();
BrakePos.AddPoint(current_time, BrakePosition);
BrakeTempbl.AddPoint(current_time, l_EASportsWRC.TelemetryData_v.brake_temp_bl.back());

...

ImPlot::PlotLine("BL", &BrakeTempbl.Data[0].x, &BrakeTempbl.Data[0].y, BrakeTempbl.Data.size(), 0, BrakeTempbl.Offset, 2 * sizeof(float));

```
## Heatmap
The heat map is very intuitive to use, it just needs the right matrix as input and it will take the right shape

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
	/********************************************************************************************/
	/*																							*/
	/*											Brake HeatMaps									*/
	/*																							*/
	/********************************************************************************************/

	static const char* xlabels[] = { "Left","Right"};
	static const char* ylabels[] = { "Front","Back"};
	static float values[2][2] = { {0,0},
								{0,0} };

	if (l_EASportsWRC.GetOnStage() && l_EASportsWRC.TelemetryData_v.brake_temp_bl.size() != 0)// only need to check size of one
	{
		values[0][0] = l_EASportsWRC.TelemetryData_v.brake_temp_fl.back();
		values[0][1] = l_EASportsWRC.TelemetryData_v.brake_temp_fr.back();
		values[1][0] = l_EASportsWRC.TelemetryData_v.brake_temp_bl.back();
		values[1][1] = l_EASportsWRC.TelemetryData_v.brake_temp_br.back();
	}
	
	static float scale_min = 0;
	static float scale_max = 400;

	static ImPlotColormap map = ImPlotColormap_Jet;
	static ImPlotHeatmapFlags hm_flags = 0;
	static ImPlotAxisFlags axes_flags = ImPlotAxisFlags_Lock | ImPlotAxisFlags_NoGridLines | ImPlotAxisFlags_NoTickMarks;
	ImPlot::BustColorCache("##Heatmap1");
	ImPlot::PushColormap(map);

	if (ImPlot::BeginPlot("##Heatmap1", ImVec2(225, 225), ImPlotFlags_NoLegend | ImPlotFlags_NoMouseText)) {
		//ImPlot::PushColormap(map);
		ImPlot::SetupAxes(NULL, NULL, axes_flags, axes_flags);
		ImPlot::SetupAxisTicks(ImAxis_X1, 0 + 1.0 / 4.0, 1 - 1.0 / 4.0,2,xlabels);
		ImPlot::SetupAxisTicks(ImAxis_Y1, 1 - 1.0 / 4.0, 0 + 1.0 / 4.0,2, ylabels);
		ImPlot::PlotHeatmap("heat", values[0],2, 2, scale_min, scale_max, "%g", ImPlotPoint(0, 0), ImPlotPoint(1, 1), hm_flags);
		ImPlot::EndPlot();
	}
	ImGui::SameLine();
	ImPlot::ColormapScale("##HeatScale", scale_min, scale_max, ImVec2(60, 225));
	ImPlot::PopColormap();
```
If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[project webpage]:/projects/ea-sports-wrc-datalogger/