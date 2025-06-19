I/O List for Rolling Mill System
1. Overview
The I/O list provided below details 30 critical signals for a rolling mill system in a steel production plant, ensuring accurate instrumentation wiring, PLC configuration, and alignment with control narratives. The rolling mill processes steel slabs into sheets or strips through sequential rolling stands, requiring precise control of parameters like roll force, tension, speed, temperature, and cooling. This table includes signal characteristics such as type, range, setpoint, and P&ID reference, facilitating integration across control panels, HMI systems, and documentation for regulatory compliance and operational efficiency.
2. I/O List Table
The table lists each signal with the following columns:

Signal Name: Descriptive name of the process parameter or action.
Input/Output: Indicates if the signal is an Input (sensor) or Output (actuator).
Signal Tagnumber: Unique identifier following ISA-5.1 conventions (e.g., TT-101, FV-201).
Analog/Digital: Specifies signal type (Analog for continuous, Digital for discrete).
Engineering Unit: Unit of measurement (e.g., °C, bar, m/min).
Range: Expected operational range (e.g., 0–1000°C).
Setpoint: Target value for control loops (e.g., 800°C).
P&ID Reference: Drawing reference for traceability (e.g., P&ID-001).




Signal Name
Input/Output
Signal Tagnumber
Analog/Digital
Engineering Unit
Range
Setpoint
P&ID Reference



Entry Slab Temperature
Input
TT-101
Analog
°C
0–1200
1000
P&ID-001


Roll Force Stand 1
Input
FT-102
Analog
kN
0–5000
3000
P&ID-002


Roll Speed Stand 1
Input
ST-103
Analog
m/min
0–100
50
P&ID-002


Entry Tension
Input
TT-104
Analog
kN
0–200
100
P&ID-003


Motor Current Stand 1
Input
IT-105
Analog
A
0–2000
1500
P&ID-002


Cooling Water Flow
Input
FT-106
Analog
L/min
0–5000
3000
P&ID-004


Roll Gap Position
Input
LT-107
Analog
mm
0–50
10
P&ID-002


Exit Strip Temperature
Input
TT-108
Analog
°C
0–1000
600
P&ID-005


Hydraulic Pressure
Input
PT-109
Analog
bar
0–250
200
P&ID-006


Looper Angle
Input
AT-110
Analog
°
0–90
45
P&ID-003


Roll Speed Stand 2
Input
ST-111
Analog
m/min
0–120
60
P&ID-007


Roll Force Stand 2
Input
FT-112
Analog
kN
0–5000
3200
P&ID-007


Exit Tension
Input
TT-113
Analog
kN
0–200
110
P&ID-005


Motor Temperature Stand 1
Input
TT-114
Analog
°C
0–150
80
P&ID-002


Cooling Water Pressure
Input
PT-115
Analog
bar
0–10
5
P&ID-004


Strip Thickness
Input
LT-116
Analog
mm
0–10
2
P&ID-005


Vibration Sensor Stand 1
Input
VT-117
Analog
mm/s
0–50
10
P&ID-002


Lube Oil Flow
Input
FT-118
Analog
L/min
0–100
50
P&ID-006


Emergency Stop
Input
XS-119
Digital
-
0/1
0
P&ID-008


Roll Stand 1 Running
Input
ZS-120
Digital
-
0/1
1
P&ID-002


Cooling Valve Command
Output
FV-201
Analog
%
0–100
50
P&ID-004


Roll Speed Command Stand 1
Output
SV-202
Analog
%
0–100
60
P&ID-002


Hydraulic Valve Command
Output
PV-203
Analog
%
0–100
70
P&ID-006


Roll Gap Adjustment
Output
LV-204
Analog
%
0–100
20
P&ID-002


Looper Motor Command
Output
SV-205
Analog
%
0–100
50
P&ID-003


Alarm Horn
Output
YA-206
Digital
-
0/1
0
P&ID-008


Motor Start/Stop Stand 1
Output
YC-207
Digital
-
0/1
1
P&ID-002


Lube Oil Pump Command
Output
YC-208
Digital
-
0/1
1
P&ID-006


Tension Control Valve
Output
TV-209
Analog
%
0–100
55
P&ID-003


Cooling Pump Start/Stop
Output
YC-210
Digital
-
0/1
1
P&ID-004


3. Notes and Usage

Purpose: This I/O list serves as a foundation for wiring, PLC configuration, and control narrative alignment for a rolling mill system, ensuring accurate signal mapping and documentation consistency.
Signal Selection: Entries cover typical rolling mill signals, including temperature (slab, strip, motor), force, speed, tension, pressure, flow, position, and safety/control signals, reflecting real-world requirements.
Tagnumber Convention: Follows ISA-5.1 (e.g., TT for temperature transmitter, FV for flow valve), ensuring standardization.
Analog/Digital:
Analog signals (e.g., TT-101, FV-201) use 4–20 mA or 0–10 V, typical for precise measurements/control.
Digital signals (e.g., XS-119, YA-206) use 24 VDC discrete I/O for on/off states.


Ranges and Setpoints:
Ranges reflect operational limits (e.g., 0–5000 kN for roll force, 0–100% for valve commands).
Setpoints are typical targets (e.g., 1000°C for slab temperature, 50% for cooling valve), adjustable per process requirements.


P&ID References: Hypothetical references (P&ID-001 to P&ID-008) link signals to process drawings for traceability; actual references depend on plant documentation.
Usage:
Wiring: Guides field instrumentation to PLC I/O modules (e.g., TT-101 to AI channel, FV-201 to AO channel).
PLC Configuration: Maps signals to PLC tags (e.g., TT-101 to AI1, YC-207 to DO1).
HMI: Provides tag names and units for operator displays (e.g., Roll Force: 3000 kN).
Control Narrative: Aligns with control loops (e.g., TIC-101 for temperature, SIC-202 for speed).


Safety Considerations:
Includes emergency stop (XS-119) and alarm horn (YA-206) for safety.
Interlocks (e.g., stop motor if VT-117 > 20 mm/s) can be derived from signals like vibration (VT-117).


Maintenance:
Table format supports Excel/PDF export for datasheets or documentation.
Clear tagnumbers and P&ID references aid troubleshooting and calibration.



4. Assumptions

The rolling mill consists of multiple stands (e.g., Stand 1, Stand 2) with typical instrumentation (thermocouples, load cells, VFDs).
Signals are based on a hot rolling mill processing steel slabs into strips; cold rolling or other processes may require adjustments.
Ranges and setpoints are typical for medium-scale rolling mills; specific values depend on mill size and steel grade.
PLC supports standard 4–20 mA analog I/O and 24 VDC digital I/O, with sufficient channels for 30 signals.
P&ID references are hypothetical; actual drawings must be provided for real projects.

This I/O list provides a structured, standardized reference for the rolling mill system, ensuring accurate instrumentation, PLC integration, and documentation alignment.
