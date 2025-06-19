Gas Turbine Interlock List
This table lists interlocks for a gas turbine in a power plant, defining triggering conditions and safety actions to protect against mechanical failure, personnel hazards, and operational disruptions.



Interlock
Triggering Condition
Safety Action



Overtemperature
Exhaust gas temperature (TGT-101) > 650°C
Shut down turbine (stop fuel flow, open vent valve, initiate cooldown), issue alarm (ALM-101)


Overspeed
Rotor speed (ST-101) > 105% of rated speed (e.g., >5250 RPM for 5000 RPM nominal)
Emergency stop (isolate fuel, engage brake, open vent valve), issue alarm (ALM-102)


Overpressure
Combustion pressure (PT-101) > 30 bar
Open pressure relief valve (PRV-101), reduce fuel flow, issue alarm (ALM-103)


Low Lubrication Pressure
Lube oil pressure (PT-102) < 1.5 bar
Shut down turbine (stop fuel flow, initiate cooldown), issue alarm (ALM-104)


High Vibration
Vibration amplitude (VT-101) > 10 mm/s
Shut down turbine (stop fuel flow, initiate cooldown), issue alarm (ALM-105)


Flame Failure
Flame detector (FD-101) indicates no flame
Stop fuel flow (close fuel valve FV-101), issue alarm (ALM-106)


Low Fuel Gas Pressure
Fuel gas pressure (PT-103) < 2 bar
Close fuel valve (FV-101), issue alarm (ALM-107)


Low Cooling Water Flow
Cooling water flow (FT-101) < 200 L/min
Shut down turbine (stop fuel flow, initiate cooldown), issue alarm (ALM-108)


Compressor Surge
Surge detected (PT-104 differential pressure > threshold or flow reversal)
Open bypass valve (BV-101), reduce load (adjust fuel flow), issue alarm (ALM-109)


Emergency Stop
Manual emergency stop button (ES-101) pressed
Isolate fuel (close FV-101), shut down turbine (stop fuel flow, open vent valve), issue alarm (ALM-110)


Notes

Triggering Conditions:
Sensor thresholds (e.g., TGT-101 > 650°C, PT-101 > 30 bar) are based on typical gas turbine safety limits; specific values may vary per design.
Fault detections (e.g., flame failure, surge) rely on dedicated sensors or algorithms in the PLC/DCS.
Manual emergency stop is operator-initiated via a physical button or HMI command.


Safety Actions:
Shutdown: Stops fuel flow, initiates cooldown (e.g., vent valve opens to release hot gases), and halts rotor operation to a safe state.
Emergency Stop: Immediate fuel isolation and braking, prioritizing rapid cessation of operation.
Valve Operations: Open/close valves (e.g., PRV-101, FV-101) to control pressure or flow, typically pneumatic or solenoid-driven.
Load Reduction: Adjusts fuel flow to reduce turbine load, stabilizing operation without shutdown.
Alarms: Activate audible/visual alerts (ALM-101 to ALM-110) on HMI, logged with timestamps for traceability.


Assumptions:
Turbine uses PLC or DCS for control, with Profibus DP or similar for sensor/actuator communication.
Sensors (TGT-101, ST-101, etc.) provide real-time data with millisecond response times.
Actions execute within seconds (e.g., valve closure <2s, shutdown <10s) to prevent escalation.
Alarms are integrated with SCADA/HMI for operator notification and logging.


Usage:
Overtemperature: Shuts down turbine if TGT-101 > 650°C, protecting blades, issues ALM-101.
Low Lubrication Pressure: Shuts down turbine if PT-102 < 1.5 bar, preventing bearing damage, issues ALM-104.
Emergency Stop: Manually isolates fuel and shuts down turbine, issues ALM-110.



