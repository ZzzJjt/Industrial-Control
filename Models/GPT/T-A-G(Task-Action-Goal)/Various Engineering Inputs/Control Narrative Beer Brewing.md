1. Overview

This control narrative outlines the automated control strategy for a beer brewing system. The process includes the following stages:
	1.	Mash-In
	2.	Mashing
	3.	Mash-Out
	4.	Lautering
	5.	Boiling
	6.	Whirlpool Separation
	7.	Wort Cooling
	8.	Fermentation Transfer

⸻

2. Setpoint Summary

Parameter
Setpoint / Range
Notes
Mash temperature
65 °C ± 1 °C
Controlled using a mash tun heater
Mash-out temperature
76 °C ± 1 °C
Initiates enzyme deactivation
Lauter tun level
60–80 %
Controlled with sparge inflow
Sparge water temperature
76 °C ± 2 °C
Maintains grain bed temp
Lauter turbidity limit
200 NTU
Defines clarity for wort transfer
Lautering flow rate
5–8 L/min
Target wort transfer rate
Total wort volume
1000 L
Target for boiling phase

3. Required Equipment
	•	Mash Tun with agitator and heating coil
	•	Lauter Tun with rake mechanism
	•	Wort Pump
	•	Sparge Water System
	•	Boil Kettle
	•	Whirlpool Separator
	•	Plate Heat Exchanger

4. Lautering Phase – Section 4

4.1 Equipment
	•	Lauter Tun
	•	Rake Motor
	•	Sparge Water Valve and Heater
	•	Wort Transfer Pump
	•	Waste Valve

4.2 Instrumentation

4.3 Lautering Sequence
	1.	Start Recirculation
	•	Engage Wort Pump in loop-back mode.
	•	Close Waste Valve.
	•	Maintain recirculation for minimum 5 minutes or until Turbidity < 200 NTU.
	2.	Monitor Turbidity
	•	Continuously read TM-404.
	•	If Turbidity > 200 NTU, continue recirculation.
	•	If Turbidity < 200 NTU, enable transition to transfer.
	3.	Start Wort Transfer to Kettle
	•	Open Wort Outlet Valve to kettle.
	•	Activate Wort Pump for forward flow.
	•	Monitor FT-403 to maintain flow at 5–8 L/min.
	4.	Initiate Sparging
	•	Open Sparge Water Valve when LT-401 < 80%.
	•	Maintain sparge water temperature at 76 °C ± 2 °C using TT-402.
	•	Regulate sparge to keep LT-401 between 60–80%.
	5.	Terminate Lautering
	•	If LT-401 < 60% and Total Volume ≥ 1000 L, stop sparging.
	•	If TM-404 > 300 NTU, open Waste Valve and discard flow.

⸻

4.4 Interlock & Alarm Logic
