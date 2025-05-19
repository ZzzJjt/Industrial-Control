Control Narrative for Ammonium Nitrate Reactor
Overview
The ammonium nitrate reactor produces a concentrated ammonium nitrate solution by reacting liquid ammonia and nitric acid in a continuous stirred-tank reactor (CSTR). The process is exothermic, requiring precise control of temperature, pressure, flow, and pH to ensure safety, product quality, and efficiency. The system is automated via a DCS/PLC with HMI interface, incorporating PID loops, interlocks, and alarms to maintain setpoints and handle deviations. This narrative outlines control parameters, instrumentation, operating procedures, and safety measures, enabling reliable operation and integration into control systems.
Core Control Parameters

Temperature:
Setpoint: 175°C
Range: 173–177°C
Purpose: Maintains optimal reaction rate, prevents decomposition


Pressure:
Setpoint: 4.8 bar (gauge)
Range: 4.6–5.0 bar
Purpose: Ensures safe operation and proper gas/liquid dynamics


Flow Ratio (Ammonia:Nitric Acid):
Setpoint: 1.01:1 (volumetric)
Range: 0.91:1 to 1.11:1 (±10%)
Purpose: Maintains stoichiometric balance for complete reaction


pH:
Setpoint: 6.2
Range: 5.8–6.5
Purpose: Ensures neutral to slightly acidic product, prevents corrosion


Level:
Setpoint: 70% (3500 L)
Range: 60–80% (3000–4000 L)
Purpose: Maintains reaction volume, prevents overfill



Critical Instrumentation and Equipment
Equipment

Reactor Vessel: 5000 L, stainless steel, jacketed, agitator (100 RPM)
Ammonia Feed System: Metering pump (0–50 L/min), control valve
Nitric Acid Feed System: Metering pump (0–50 L/min), control valve
Cooling Jacket: Water circulation (20–50°C, 5 bar)
Product Pump: Transfers solution (0–100 L/min)
Vent System: Pressure relief valve, scrubber for NH₃, NOx
Isolation Valves: Manual and automated for feed, product, vent

Instrumentation

TIC-101: Temperature controller, 0–200°C, ±0.1°C, PID for jacket
PIC-102: Pressure controller, 0–10 bar, ±0.01 bar, PID for vent
FIC-103: Ammonia flow controller, 0–50 L/min, ±0.1 L/min, PID
FIC-104: Nitric acid flow controller, 0–50 L/min, ±0.1 L/min, PID
AIC-105: pH analyzer, 0–14, ±0.05
LSH-106: High-level switch, 90% (4500 L), ±1%
TT-101: Temperature transmitter (redundant)
PT-102: Pressure transmitter (redundant)
FT-103, FT-104: Flow transmitters for ammonia, nitric acid
LT-107: Level transmitter, 0–100%, ±1%
AT-108: Ammonia gas detector, 0–100 ppm, ±1 ppm

Operating Procedures
Startup
Objective: Initialize reactor to steady state.

Valve Checks (1 min):
Verify isolation valves closed, test automated valves.
Logic: IF Valve Feedback ≠ Command THEN alarm.


Initialize Jacket Heating (30 min):
Start cooling water (20°C, 50 L/min).
Preheat to 100°C (2°C/min) via TIC-101.
Logic: IF Temp < 98°C THEN increase steam; IF > 102°C THEN reduce steam.


Fill Reactor (10 min):
Fill with water to 60% (3000 L) via underlet valve.
Logic: IF Level > 65% THEN close valve; IF LSH-106 = TRUE THEN stop and alarm.


Ramp Up Feeds (10 min):
Start ammonia (FIC-103) at 10 L/min, nitric acid (FIC-104) at 9.9 L/min.
Ramp to 30 L/min ammonia, 29.7 L/min acid over 5 min.
Start agitator (100 RPM).
Set TIC-101 to 175°C, PIC-102 to 4.8 bar.
Logic:
IF Flow Ratio < 0.91 OR > 1.11 THEN adjust FIC-103/104.
IF Temp < 173°C THEN increase heating; IF > 177°C THEN increase cooling.
IF Pressure < 4.6 bar THEN close vent; IF > 5.0 bar THEN open vent.




Stabilize (5 min):
Monitor pH (AIC-105) at 6.2 ±0.3.
Adjust ammonia if pH < 5.8 (increase) or > 6.5 (decrease).
Confirm level at 70% (LT-107).
Logic: IF pH < 5.8 FOR >1 min THEN increase ammonia by 0.1 L/min; IF > 6.5 THEN decrease by 0.1 L/min.



Safety:

Interlock: IF LSH-106 = TRUE THEN close feed valves.
Alarm: IF Temp > 180°C OR Pressure > 5.1 bar FOR >30 s THEN alert.

Normal Operation
Objective: Maintain steady-state production.

Maintain Setpoints:
TIC-101: 175°C ±2°C (PID).
PIC-102: 4.8 bar ±0.2 bar (PID).
FIC-103/104: 1.01:1 ratio (30 L/min ammonia, 29.7 L/min acid, PID).
AIC-105: pH 6.2 ±0.3, adjust ammonia.
LT-107: 70% ±10% (3500 L) via product pump (50 L/min).
Logic:
IF Temp < 173°C THEN reduce cooling; IF > 177°C THEN increase cooling.
IF Pressure < 4.6 bar THEN close vent; IF > 5.0 bar THEN open vent.
IF Flow Ratio < 0.91 OR > 1.11 THEN adjust FIC-103/104, alarm.
IF pH < 5.8 THEN increase ammonia; IF > 6.5 THEN decrease.
IF Level < 60% THEN reduce pump; IF > 80% THEN increase pump.




Monitor and Adjust:
Log temperature, pressure, flow, pH, level.
Check agitator (100 RPM ±10).
Monitor AT-108 (<5 ppm).
Logic: IF AT-108 > 10 ppm THEN ventilate; IF Agitator < 90 RPM THEN stop feeds, alarm.


Product Transfer:
Transfer solution at 50 L/min ±5 L/min.
Logic: IF Level > 80% THEN increase pump to 60 L/min; IF < 60% THEN decrease to 40 L/min.



Safety:

Interlock: IF Temp > 185°C OR Pressure > 5.2 bar OR pH < 5.5 THEN ESD.
Alarm: IF Flow Ratio ±10% FOR >1 min THEN alert, lockout feeds.

Shutdown
Objective: Stop reaction safely.

Stop Feeds (1 min):
Close FIC-103/104 valves, stop pumps.
Logic: IF Valve Feedback ≠ Closed THEN alarm, ESD.


Vent Pressure (5 min):
Open vent valve (PIC-102) to 0 bar.
Logic: IF Pressure > 0.5 bar FOR >5 min THEN alarm.


Cool Reactor (60 min):
Increase cooling water (100 L/min, 20°C).
Ramp to 50°C (2°C/min) via TIC-101.
Logic: IF Temp > 60°C FOR >30 min THEN alarm.


Drain and Close (10 min):
Drain via product pump (LT-107 < 10%).
Close all valves, stop agitator, cooling.
Logic: IF Level > 10% FOR >10 min THEN alarm.



Safety:

Interlock: IF LSH-106 = TRUE DURING DRAIN THEN stop pump, alarm.
Alarm: IF AT-108 > 5 ppm THEN ventilate, alert.

Safety Interlocks and Alarms
Emergency Shutdown (ESD)

Conditions:
Temp > 185°C FOR >10 s
Pressure > 5.2 bar FOR >10 s
pH < 5.5 FOR >30 s
LSH-106 = TRUE
AT-108 > 50 ppm


Actions:
Close feed valves
Stop pumps
Open vent valve
Maximize cooling
Stop agitator
Alarm, require manual reset



Alarms and Lockouts

Flow Ratio: IF < 0.91 OR > 1.11 FOR >1 min THEN alarm, lockout feeds.
Temperature: IF < 170°C OR > 180°C FOR >2 min THEN alarm, adjust cooling.
Pressure: IF < 4.4 bar OR > 5.1 bar FOR >2 min THEN alarm, adjust vent.
pH: IF < 5.7 OR > 6.6 FOR >2 min THEN alarm, adjust ammonia.
Level: IF < 50% OR > 85% FOR >5 min THEN alarm, adjust pump.
Ammonia Leak: IF AT-108 > 10 ppm FOR >30 s THEN ventilate, alarm; >20 ppm THEN stop feeds.

Integration and Documentation

DCS/PLC: Maps to function blocks (e.g., FB_TIC101, FB_FlowRatio), state machine for startup/operation/shutdown.
HMI: Displays Temp, Pressure, pH, Level, FlowRatio; controls for Start, Stop, Reset.
Interlocks: ESD conditions translate to logic diagrams.
Alarms: Configurable in SCADA (e.g., FlowRatioDeviation).
Documentation: Supports training with clear parameters, procedures, and logic.

