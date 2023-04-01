# ESP32/SensESP Last Gasp Circuit

A power supply circuit that includes a SuperCap to power an ESP32 for 10+ seconds after power disconnection, thereby enabling ESP32/sensESP to update status details to Signalk Server before the ESP32 shuts down.

Details on sensESP can be found at ** insert link 

## Background

Under normal operation, when disconnecting power to an ESP32 module, it immediately shuts down, and hence cannot update sensor status or send any "final" messages to the signalk server.
In the case where the ESP32 is powered from the circuit that you are also wanting to monitor, this "last gasp" circuit will allow sufficient time for the senESP to update the circuit power status before it shuts down.
Example: An anchor washdown pump installed in the anchor locker has a local switch that powers the pump (& sensESP) by using power from the anchor winch circuit. 
I want to set a reminder/alarm if i forget to manually switch off power to the washdown pump; however because the ESP32 is also powered from the pump circuit - i want it to update the power status in signalk (to OFF) BEFORE it dies (because its on the same circuit).
Without a "last gasp" circuit, its last power status reported to signalk would remain showing 'ON" & the reminder/alarm would continue to trigger (along with dashboard display showing "ON" status for the pump)

## Block Diagram


## The Circuit

There are 3 main components in the power supply circuit, namely:
- buck regulator   
  (that includes an integrated current limiting circuit)

  ** insert link to pic ** 
- shottky diode  
(to prevent current from SuperCap "escaping" back thru buck regulator output when its de-powered)

** insert link to pic 
- SuperCap "Farad Capacitor" (5.5V 10F)

** insert link to pic

### ESP32 Power Input   
I chose to supply 3.3V direct to the ESP32 board from the buck regulator/supercap circuit, to avoid the additional current draw and uncertainties from the 5V to 3.3V linear regulator on the ESP32 module. 

Also the SuperCaps have a working voltage rating of "5.5V" which would be  uncomfortably close to the nominal 5V input required if using onboard regulator. 

Note: the mfr spec for ESP32 is "minimum 3.0V" for correct operation.
Typical current draw (as measured) for a ESP32 module with a 2 channel "817" OptoIsolator module: 
"idling" ~75mA; "when transmitting" (wifi) ~140mA

Note that additional sensors will increase the above current draw.
### Shottky Diode
The purpose of the shottky diode is to prevent reverse current flow "back into" the regulator output when the power is removed. The diode has a forward voltage drop of ~0.4V @1A, falling to around 0.32V @ 100mA, that needs to be compensated at the regulator output. As the current thru the diode can be >1A, a diode with an appropriate current rating is needed (eg 3A).
### Buck Regulator
The buck regulator must be the type that ALSO includes an onboard "adjustable current limiting" circuit, which is used to limit the "inrush" current when connecting power to the regulator and charging the SuperCap, otherwise current draw by the SuperCap can be many Amps, and would likely cause the regulator to sense a "short circuit" and to shutdown its output.
Limit the output current to a max of ~ 1A.

## Circuit Operation
When first powered up, the regulator will output ~1A at 3.62V, with a forward voltage drop of 
 ~0.32V across the shottky diode, reducing the voltage across the SuperCap & ESP32 to a max of 3.3V. 

 Because of the current limiting circuit and the capacitor charge characteristic, the voltage across the capacitor (& hence applied to the ESP32) will rise from 0V to 3.3V over approx 20 seconds (depending on Farad value of SuperCap & buck regulator current limit setting). 
The ESP32 starts operation as the voltage rises past 3.0V (nominal). 

 When power is disconnected to the regulator, the SuperCap supplies power to the ESP32, with the voltage falling from 3.3V to (eventually) 0V. 
  
 The ESP32 continues to operate normally as the voltage falls from 3.3V to 3.0V .. the "last gasp" period, but stops operating when the voltage falls below 3.0V (nominally), below which its in an indeterminate state ("brownout").
 Depending on the Farad value of SuperCap (& current draw of ESP32 module and any other sensors/circuitry) the "last gasp" period can be set to >20 seconds, during which it continues to send update messages to signalk server. 
 
 (note sensESP code needs to ensure a periodic update that falls well within the "last gasp" period; and/or sense that power has been disconnected and immediately trigger appropriate updates)

 ## Details for those interested

 ### ESP32 Power
 Measured current draw for typical ESP32 Dev module ranged between ~65mA to 140mA (when transmitting updates via wifi); with an assumed average current for design purposes of ~100mA.
 Whilst mfr spec for ESP32 correct operation is V >3.0V, in practise the observed updates from sensESP (as recorded in signalk data browser paths) exceeded the measured "last gasp" (3.3V to 3.0V ) period by a couple of seconds.
 My test setup is in a reasonably good wifi coverage area from the Access Point, I was unable to measure current draw under lower coverage wifi signal conditions which would likely adversely impact current draw. 
 ### SuperCap specs
 The typical SuperCap "technology" has a working voltage of ~2.7V. However commonly available devices (at reasonable prices) are available in working voltages of 2.7 & 5.5V (which is actually 2x2.7V  Caps in series), and Farad values ranging from 0.5F to 15F.
 eg depending on source, a 5.5V 10F SuperCap can be found for around $3. 
 Any capacitor has "losses" that stem from its design/characteristics which are "modelled" by its "equivalent series resistance" ESR. This resistance is effectively in "series" with the ESP32 "load", and together they determine the discharge current. In practise however the ~200 mOhm (typical) SuperCap ESR is much lower than the ESP32 equivalent load resistance of ~30 Ohm (3V @ 100mA) and can thus be ignored.
 I sourced both 5F & 10F SuperCaps and tested both in the circuit.

### Buck Regulator
I chose to limit the regulator output to ~1A, so the Cap will charge to 3.3V within around 15 seconds, thus the delay in "startup" of the ESP32 is minimal, but limits the demand on the regulator and power disspitated in the diode to manageable levels.
The buck regulator shown has a spec of "2A" (for continued use at this level additional heatsinking is required), however for a current of 1A (for the short period of SuperCap charging) its well within the spec and no additional heatsink is required. (although i did add a "stick on" small copper heatsink to be safe)

 ### Shottky Diode
A widely and commonly available shottky diode is the 1N5822 (40V 3A) that has a "flat" voltage drop over the current range of 0-1.5A.
By keeping the "charging" current of the SuperCap to 1A (max) the forward voltage drop across the diode varies between 0.4V (when charging @1A) and 0.32V once the SuperCap is charged and the current falls to ~100mA (avg) from the ESP32 load.
This enables the buck output to be set to 3.62V (0.32V above 3.3V) which allows the SuperCap to reach a stable 3.3V

![image](https://github.com/gregsyoung/ESP32-sensESP-Last-Gasp-Circuit/blob/main/1N5822%20diode.jpg)

 ### Charge and DisCharge (voltage) Curves
 Below are measured voltage vs time curves for each of a 5F & 10F SuperCap in the above circuit.
