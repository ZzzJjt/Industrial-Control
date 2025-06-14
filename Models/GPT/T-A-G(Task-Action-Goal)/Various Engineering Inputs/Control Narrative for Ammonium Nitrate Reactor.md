1. Process Overview
	•	Objective: Continuous neutralization of nitric acid (HNO₃) with anhydrous ammonia (NH₃) to produce ammonium nitrate (NH₄NO₃).
	•	Reactor Type: Continuous Stirred Tank Reactor (CSTR) with steam jacket.
	•	Production Mode: Continuous operation with closed-loop controls.

⸻
2. Key Process Parameters

Parameter
Tag
Setpoint / Range
Control Type
Reactor Temperature
TIC-101
175 °C ± 2 °C
PID with steam valve control
Reactor Pressure
PIC-102
4.8 bar ± 0.2 bar
PID with vent control
Ammonia Flow Rate
FIC-103
Calculated by molar ratio
Feed-forward loop
Acid Flow Rate
FIC-104
Master flow (basis of ratio)
PID
Ammonia-to-Acid Ratio
—
1.01:1 molar
Computed in DCS
pH of Reaction Mixture
AIC-105
6.2 ± 0.3
Cascade loop to ammonia feed
Reaction Residence Time
—
~15 minutes
Design parameter
3. Instrumentation and Control Elements
Device
Function
TIC-101
Controls reactor temperature via steam valve
PIC-102
Maintains reactor pressure with pressure relief valve
FIC-103
Controls ammonia flow
FIC-104
Controls nitric acid flow
AIC-105
Monitors and controls pH via ammonia ratio
Level Sensor
High/low alarms for reactor overfill/empty
