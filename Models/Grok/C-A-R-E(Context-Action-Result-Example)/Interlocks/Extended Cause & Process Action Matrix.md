Chemical Reactor Cause and Process Action Matrix
This matrix maps hazardous causes to safety responses (interlocks) for a chemical reactor system. Rows represent potential causes, and columns represent process actions. Checkmarks (✔) indicate actions triggered by each cause.



Cause
Isolate Feed Valve
Shut Down Agitator
Activate Emergency Vent
Stop Heating System
Open Cooling System
Trigger Safe Shutdown
Sound Alarm
Log Event



High Pressure
✔

✔
✔


✔
✔


Low Pressure
✔





✔
✔


High Temperature


✔
✔
✔

✔
✔


Low Temperature
✔





✔
✔


High Liquid Level
✔
✔




✔
✔


Low Liquid Level
✔
✔

✔


✔
✔


Level Sensor Failure
✔
✔

✔

✔
✔
✔


Pressure Sensor Failure
✔

✔
✔

✔
✔
✔


Temperature Sensor Failure
✔

✔
✔
✔
✔
✔
✔


Power Supply Failure
✔
✔
✔
✔

✔
✔
✔


Notes

Causes:
High/Low Pressure: Exceeds/falls below safe pressure limits (e.g., >10 bar or <2 bar).
High/Low Temperature: Exceeds/falls below safe temperature limits (e.g., >200°C or <50°C).
High/Low Liquid Level: Exceeds/falls below safe level limits (e.g., >90% or <10% of tank).
Sensor Failures: Loss of signal or invalid readings from level, pressure, or temperature sensors.
Power Supply Failure: Loss of power to critical systems (e.g., control or actuators).


Actions:
Isolate Feed Valve: Closes inlet to prevent material addition.
Shut Down Agitator: Stops mixing to reduce reaction or level risks.
Activate Emergency Vent: Opens relief valve to reduce pressure.
Stop Heating System: Halts heaters to prevent overheating.
Open Cooling System: Activates cooling to lower temperature.
Trigger Safe Shutdown: Initiates controlled reactor shutdown.
Sound Alarm: Alerts operators via audible/visual signals.
Log Event: Records event with timestamp for analysis.


Assumptions:
Reactor has pressure, temperature, and level sensors with defined safety limits.
Interlocks are implemented via PLC with Profibus DP or similar communication.
All actions are executable within seconds to ensure safety.


Usage:
High Pressure: Triggers isolate feed, emergency vent, stop heating, alarm, and log.
Level Sensor Failure: Triggers isolate feed, shut down agitator, stop heating, safe shutdown, alarm, and log.



