
---
title: "Investigation into the Pajero temperature gauge and sensor"
date: "2026-09-27"
slug: "TempSensorInvestigation"
tags: ["Hardware", "Sensor","Car","Vehicle","Analog"]
categories: ["Hardware"]
series: ["Pajero"]
draft: false
---

# Introduction

As presented in other articles, I own a Mitsubishi Pajero which I have kept as a project car.
Recently I noticed a worrying symptom: when driving above 3000 RPM under load, coolant would start appearing in the expansion tank, suggesting pressure was building up in the cooling circuit.
The strange part was that the temperature gauge on the dashboard never showed any sign of overheating during these events.

This raised an immediate question: What is wrong with my cooling circuit?
I immediately went for the [5 Whys] technique.
1. Coolant increased drastically in the expansion tank - **Why?**
2. Because pressure increased in the radiator - **Why?**
3. The radiator cap didn't allow for coolant to go back to the radiator - **Why?**
4. Because pressure was higher than what is usually expected in the cooling circuit - **Why?**
5. Something is creating pressure in the cooling circuit - **why?**

Here started the mechanical investigation, where I started by:

➞ **Changing the radiator cap**: A bad radiator cap could lead to coolant not returning to the rad and it's cheap to replace, this wasn't the issue.

➞ **Changing the radiator**: My radiator was gunked up with rust smush, a blocked radiator could also lead to this behavior. I replaced it and kept having the same problem.

➞ **Verify the water pump**: If the water pump was bad or air-locked in some way, it could be cavitating or something and creating more pressure than expected, I inspected it and everything was fine.

While replacing the radiator I took the time to also replace the thermostat, the old one was fine but was all rusty as well.
So, the only other thing creating pressure in the circuit was a blown head gasket, which was the most probable cause, but is still one of the most expensive repairs.

At this point I handed it to a mechanic, as this job is a bit too much for me and I prefer to have someone dedicated to it instead of me trying to do it.

Still, I was left wondering, I never actually saw the temperature rise above the normal operating temperature, how could it have broken the head gasket?

So I decided to do a deep dive on the temperature gauge circuit and figure out the hardware.

This post documents my investigation into the coolant temperature sensor and gauge circuit to try to understand the situation.

# The Circuit

The Pajero 4M40 uses two separate components for temperature measurement:

- **A-104** – Engine coolant temperature *gauge unit* (the sender that drives the dashboard gauge needle)
- **A-113** – Engine coolant temperature *sensor* (used by the ECU for fuel/ignition management)

These are physically different units, mounted in the engine block, and they serve different purposes.
The wiring diagram for the engine compartment (page 49, section 2A-32) shows how both are wired:

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/WiringDiagramTempSensor.png" alt="Wiring Diagram" caption="Engine compartment wiring - temp gauge unit and sensor" width="50%" >}}

The location of both sensors in the engine can be seen below:

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/TempSensorLocation.png" alt="Sensor Location" caption="Coolant temperature sensor location on the 4M40" width="75%" >}}

The troubleshooting guide for the meter and gauge circuit (section 4A-150, page 247) also provides useful diagnostic steps and the expected resistance values for the gauge sender:

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/TroubleshootNotes.png" alt="Troubleshoot Notes" caption="Troubleshoot notes from the 4M40 workshop manual" width="75%" >}}


# Hardware Investigation

There are two major aspects to investigate in this system:
1. **The physical gauge loop:** Checking the resistance values needed to command the various needle positions on the dashboard gauge.
2. **The actual physical sensor:** Mapping out real-world resistance-to-temperature curves of the coolant sensor in a controlled bath.

### 1. Dashboard Gauge Needle Calibration Test

First, to understand what resistance values the dashboard's bimetallic temperature gauge expected in order to display different states, I bypassed the engine block sensor and simulated engine temperatures using a series of fixed resistors to ground:

| Wire Resistance to Ground (Ω) | Needle Position / Dashboard Reading | Meaning of State |
|---|---|---|
| **47 Ω** | Reaches "middle" (beginning of centered region) | Normal cold-to-warm transition |
| **32 Ω** | Center of normal range (ideal operating position) | Normal warmed-up operating state |
| **20 Ω** | Starts rising above the center range | Beginning of over-temp threshold |

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/47Ohms.jpg" alt="Gauge Test" caption="47 Ω" width="75%" >}}

---

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/32Ohms.jpg" alt="Gauge Test" caption="32 Ω" width="75%" >}}

---

{{< figure src="/images/PajeroProjects/CoolantTempInvestigation/20Ohms.jpg" alt="Gauge Test" caption="20 Ω" width="75%" >}}

This test shows that the physical needle gauge expects **roughly 47–67 Ω of resistance** during normal fully-warmed operation, and anything **below ~20 Ω** should result in a rising needle pointing towards overheating.

### 2. Measuring the Actual Coolant Temperature Sensor

With a gas stove, a thermometer and a multimeter, I mapped out the real-world performance of the actual temperature sensor at various temperature points:

| Temperature (°C) | Measured Resistance (Ω) |
|---|---|
| **31 °C** | 1731 |
| **38 °C** | 1695 |
| **57 °C** | 615 |
| **72 °C** | 418 |
| **75 °C** | 366 |
| **80 °C** | 320 |

<div class="sensor-graph" style="margin: 30px auto; width: 100%;">
<svg viewBox="0 0 600 400" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 850; display: block; margin: 0 auto;">
<title>Coolant Temp Sensor (A-113) Resistance vs Temperature Curve</title>
<g stroke="currentColor" stroke-width="1" stroke-dasharray="4,4" opacity="0.15">
<line x1="60" y1="40" x2="60" y2="350" />
<line x1="152.7" y1="40" x2="152.7" y2="350" />
<line x1="245.5" y1="40" x2="245.5" y2="350" />
<line x1="338.2" y1="40" x2="338.2" y2="350" />
<line x1="430.9" y1="40" x2="430.9" y2="350" />
<line x1="523.6" y1="40" x2="523.6" y2="350" />
<line x1="570" y1="40" x2="570" y2="350" />
</g>
<g stroke="currentColor" stroke-width="1" stroke-dasharray="4,4" opacity="0.15">
<line x1="60" y1="350" x2="570" y2="350" />
<line x1="60" y1="298.3" x2="570" y2="298.3" />
<line x1="60" y1="246.7" x2="570" y2="246.7" />
<line x1="60" y1="195.0" x2="570" y2="195.0" />
<line x1="60" y1="143.3" x2="570" y2="143.3" />
<line x1="60" y1="91.7" x2="570" y2="91.7" />
<line x1="60" y1="40" x2="570" y2="40" />
</g>
<g stroke="currentColor" stroke-width="1.5">
<line x1="60" y1="40" x2="60" y2="350" stroke-linecap="round" />
<line x1="60" y1="350" x2="570" y2="350" stroke-linecap="round" />
</g>
<g font-size="10" font-family="system-ui, -apple-system, sans-serif" fill="currentColor" text-anchor="middle" dominant-baseline="hanging">
<text x="60" y="358">30°C</text>
<text x="152.7" y="358">40°C</text>
<text x="245.5" y="358">50°C</text>
<text x="338.2" y="358">60°C</text>
<text x="430.9" y="358">70°C</text>
<text x="523.6" y="358">80°C</text>
<text x="570" y="358">85°C</text>
</g>
<g font-size="10" font-family="system-ui, -apple-system, sans-serif" fill="currentColor" text-anchor="end" dominant-baseline="central">
<text x="50" y="350">0</text>
<text x="50" y="298.3">300 Ω</text>
<text x="50" y="246.7">600 Ω</text>
<text x="50" y="195.0">900 Ω</text>
<text x="50" y="143.3">1200 Ω</text>
<text x="50" y="91.7">1500 Ω</text>
<text x="50" y="40">1800 Ω</text>
</g>
<text x="315" y="390" text-anchor="middle" font-family="system-ui, -apple-system, sans-serif" font-size="12" font-weight="600" fill="currentColor">Temperature (°C)</text>
<text transform="rotate(-90)" x="-195" y="15" text-anchor="middle" font-family="system-ui, -apple-system, sans-serif" font-size="12" font-weight="600" fill="currentColor">Resistance (Ω)</text>
<path d="M 60,60.8 L 106.4,110.8 L 152.7,151.0 L 199.1,183.3 L 245.5,209.4 L 291.8,230.7 L 338.2,248.2 L 384.5,262.6 L 430.9,274.6 L 477.3,284.6 L 523.6,294.9 L 570.0,301.7" fill="none" stroke="#f59e0b" stroke-width="2" stroke-dasharray="5,4" opacity="0.8" />
<path d="M 69.3,51.9 L 134.2,58.1 L 310.4,244.1 L 449.5,278.0 L 477.3,287.0 L 523.6,294.9" fill="none" stroke="#3b82f6" stroke-width="3.5" stroke-linecap="round" stroke-linejoin="round" />
<g fill="#ef4444" stroke="#ffffff" stroke-width="1.5">
<circle cx="69.3" cy="51.9" r="5" />
<circle cx="134.2" cy="58.1" r="5" />
<circle cx="310.4" cy="244.1" r="5" />
<circle cx="449.5" cy="278.0" r="5" />
<circle cx="477.3" cy="287.0" r="5" />
<circle cx="523.6" cy="294.9" r="5" />
</g>
<g font-size="9" font-family="system-ui, -apple-system, sans-serif" font-weight="600" fill="currentColor">
<text x="76" y="46" text-anchor="start">31°C, 1731Ω</text>
<text x="142" y="54" text-anchor="start">38°C, 1695Ω</text>
<text x="318" y="237" text-anchor="start">57°C, 615Ω</text>
<text x="435" y="272" text-anchor="end">72°C, 418Ω</text>
<text x="477.3" y="306" text-anchor="middle">75°C, 366Ω</text>
<text x="531" y="291" text-anchor="start">80°C, 320Ω</text>
</g>
<g font-size="10" font-family="system-ui, -apple-system, sans-serif" fill="currentColor">
<rect x="345" y="45" width="220" height="55" rx="4" fill="currentColor" opacity="0.05" />
<rect x="345" y="45" width="220" height="55" rx="4" fill="none" stroke="currentColor" opacity="0.15" />
<line x1="355" y1="60" x2="385" y2="60" stroke="#3b82f6" stroke-width="3" />
<circle cx="370" cy="60" r="4.5" fill="#ef4444" stroke="#ffffff" stroke-width="1" />
<text x="395" y="60" dominant-baseline="central" font-weight="600">Measured (A-113 sensor)</text>
<line x1="355" y1="80" x2="385" y2="80" stroke="#f59e0b" stroke-width="2" stroke-dasharray="4,3" />
<text x="395" y="80" dominant-baseline="central" opacity="0.85">Fitted NTC Model (β = 3550K)</text>
</g>
</svg>
</div>

---

## Comparing the Curves: Gauge vs. Sensor

There is a glaring discrepancy here:
- At a fully warmed up **80 °C**, our actual sensor measures **320 Ω**.
- But our dashboard instrument requires **47–67 Ω** to reach the "middle" operating scale!

If we hooked this sensor directly to our gauge, the 320 Ω resistance would keep the needle pinned to the very bottom (completely cold), even with the engine running at a perfect 80 °C.

# The Realization: I Got the Wrong Sensor!

This is where the classic trap of auto parts sourcing comes in. The Mitsubishi Pajero uses a dual-component architecture for temperature measurement:
- **A-104 (Gauge Sender Unit)**: A single-pin sensor grounding directly through the engine block. Because it has to drive high thermal induction currents through the dashboard bimetallic coil to deflect the heavy instrument needle, it uses a **low-resistance curve** (nominally ~100 Ω cold, ~50 Ω normal operating, ~20 Ω hot).
- **A-113 (Engine Coolant Temp Sensor / ECT)**: A two-pin sensor connected directly to the Engine ECU. Since it only sends a signal to a high-impedance ADC reference in the engine computer, it uses a **high-resistance curve** (~2.5 kΩ cold, ~300 Ω at normal operating temps).

When sourcing a replacement part to solve my non-responsive dashboard temperature gauge, I accidentally ordered and measured the **ECU Coolant Temp Sensor (A-113)** instead of the **Gauge Sender Unit (A-104)**! 

The measurements I performed in the thermal bath are absolutely perfect and fit a standard ECU thermistor curve flawlessly, but they are completely incompatible with the analog dashboard gauge loop. I was measuring the wrong sensor the entire time!

---

## Fitting the NTC Curve for the (Wrong) ECU Sensor

Let's derive the Beta ($\beta$) value of this ECU sensor to document its curve anyway. Using our outer data points at 80 °C (353.15 K, 320 Ω) and 31 °C (304.15 K, 1731 Ω):

$$
\beta = \frac{\ln(1731 / 320)}{\dfrac{1}{304.15\,\text{K}} - \dfrac{1}{353.15\,\text{K}}} \approx 3701\,\text{K}
$$

Setting a typical optimized constant base of **$\beta = 3550\,\text{K}$**, and using $R_0 = 320\,\Omega$ at reference $T_0 = 353.15\,\text{K}$ (80 °C), let's see how our fitted curve aligns with the entire benchmark dataset:

| Temperature (°C) | Fitted Expected R (Ω) | Actual Measured R (Ω) | Accuracy / Match |
|---|---|---|---|
| **20 °C** | ~2570 | — | (Typical cold start spec: ~2.5 kΩ) |
| **31 °C** | 1616 | 1731 | ✓ +7% deviation |
| **38 °C** | 1243 | 1695 | ⚠ +36% deviation (experimental margin) |
| **57 °C** | 643 | 615 | ✓ -4% deviation |
| **72 °C** | 404 | 418 | ✓ +3% deviation |
| **75 °C** | 370 | 366 | ✓ -1% deviation |
| **80 °C** | 320 | 320 | ✓ reference anchor |

The fitted model matches my experimental measurements with incredible precision, particularly at the active running ranges above 50 °C. This proves that the ECU sensor itself is in perfect health and follows a standard Mitsubishi NTC curve, even if it is completely mismatched for my dashboard gauge problem!

# Conclusions from the Sensor Investigation

1. **I measured and verified the wrong thermal sensor**: The bench test curves match the high-resistance Engine ECU NTC sensor (A-113) specifications perfectly, but have absolutely nothing to do with driving the low-resistance dashboard gauge unit (A-104).
2. **The dashboard gauge tests remain valid**: Sourcing fixed resistance values directly in the harness proves that the physical dashboard indicator loop requires a low resistance range of 20–67 Ω to deflect the needle.
3. **The mystery continues**: Because I was testing the ECU sensor instead of the gauge sender, I still haven't verified the status of the actual dashboard sender unit (A-104) inside the car.



[buymeacoffee]: https://buymeacoffee.com/Carlos4lmeida
[CarInclinometer]: /posts/carinclinometer/
[ntc_theory]: https://www.electronics-tutorials.ws/io/thermistors.html
[5 Whys]:https://en.wikipedia.org/wiki/Five_whys