---
title: "Car Inclinometer with RP2040 and LVGL"
date: "2025-05-09"
slug: "CarInclinometer"
tags: ["LVGL", "Software","RP2040","Car","Vehicle"]
categories: ["Software"]
series: ["Pajero"]
draft: false
---
# Introduction
So, I have a old car, a 1994 Mitsubishi Pajero which I affectionately call Rafiki. Im restoring/improving it with some new tech
{{< figure src="/images/PajeroProjects/PajeroMud.jpg" width="100%" >}}

These 4x4s usually have a center display with 3 circular screens with useful information, mine has a thermometer with outside and inside temperature, an analog inclinometer and an analog altimeter.
Due to its age (the car as it sits is almost completely original as it was in 1994), the inclinometer does not work as well as it used to.
Queue in the engineering mantra:
> To the engineer, all matter in the universe can be placed into one of two categories: 
(1) things that need to be fixed, and (2) things that will need to be fixed after you've had a few minutes to play with them. If there are no problems handily available, they will create their own problems.  - Scott Adams

An old inclinometer clearly falls under the first statement.

# Hardware
Previously in my [Simulator Racing Hub][SimRacingHub] I used a similar board to the one Im using in this project.
It still is a [Waveshare RP2040] design, but with a capacitive touchscreen.
For this development, I dint't create any special hardware yet, I focused firstly on the software development.

# Target
The requirements were easy to understand, I just wanted to read IMU data and display it in an inclinometer screen with a max of +45 and -45 degrees of roll and pitch
If you need to understand what I mean by roll and pitch, take a look at this [article][Motion_Yaw_Roll].
{{< figure src="/images/PajeroProjects/motion_yaw_pitch_roll.jpg" alt="[Motion_Yaw_Roll]" caption="Pitch Roll definition" width="50%" >}}

# IMU transform
The IMU installed is installed in the PCB in a specific direction, but of course once its installed in the vehicle, the IMU data needs to be transformed in the vehicle axis.
The roll and pitch angles use the gravity vector as reference, meaning that the roll and pitch angles are the angle between the gravity vector and the vehicle axis.
With this information, and the IMU mount on the PCB, the following equations can be derived:
$$
 roll = \arctan(\frac{y}{-x})
$$

$$
 pitch = \arctan(\frac{z}{-x})
$$

# Screen Design
As mentioned in the title, LVGL was used to develop the screen. Instead of manually creating the C code, U used the squareline editor.


{{< figure src="/images/PajeroProjects/Screenshot1.png" width="50%" >}}


The arcs show the roll of the vehicle, going from +45 to -45 degrees, while the center bar shows the pitch of the vehicle.
For the future, I also want to add other screens with different functionalities, like engine motor temperature measurement. That is why theres a second screen available.
Also, on the top and bottom I added the real values of pitch and roll, not limited to [-45,+45].
This is the function that transforms the IMU data into roll and pitch angles.
The `normalize` functions turn the interval [-45,+45] to [0,100].

```cpp {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
/********************************************************************************
function:	Calculate Roll and pitch
parameter:
********************************************************************************/
void CalculateRP(float acc[3], float *RP)
{
    float Xbuff = acc[0];
    float Ybuff = acc[1];
    float Zbuff = acc[2];
    
    RP[0] = atan2(Ybuff , -Xbuff) * 57.3;
    RP[1] = atan2(Zbuff,-Xbuff ) * 57.3;

    _ui_label_set_property(uic_RollText,_UI_LABEL_PROPERTY_TEXT,turnFloat2Char(RP[0]));
    _ui_label_set_property(uic_PitchText,_UI_LABEL_PROPERTY_TEXT,turnFloat2Char(RP[1]));
    lv_arc_set_value(uic_RollA,(int16_t)(100-normalize(RP[0])));
    lv_arc_set_value(uic_RollB,(int16_t)(100-normalize(RP[0])));
    lv_slider_set_value(uic_Pitch,(int32_t)normalize(RP[1]), LV_ANIM_ON);
}

```

If you like my projects please consider supporting my hobby by [buying me a coffee][buymeacoffee]:coffee: :smile:

[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida

[project webpage]:/projects/ea-sports-wrc-datalogger/
[SimRacingHub]:/projects/SimRacingHub/
[Waveshare RP2040]:https://www.waveshare.com/wiki/RP2040-Touch-LCD-1.28
[Motion_Yaw_Roll]: https://www.formula1-dictionary.net/motions_of_f1_car.html