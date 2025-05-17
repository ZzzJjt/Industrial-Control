Chemical Reactor Cause and Effect Matrix
This matrix maps sensor-based causes to safety effects (interlocks) for a chemical reactor system. Rows represent causes (deviations from safe thresholds), and columns represent effects (safety responses). Checkmarks (✔) indicate effects triggered by each cause.



Cause
Close Feed Valve
Stop Feed Pump
Activate Relief System
Stop Heating System
Activate Cooling System
Issue Alarm
Initiate Safe Shutdown



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



Low Temperature
✔




✔



High Liquid Level
✔
✔



✔



Low Liquid Level
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


Notes

Causes:
High Pressure: Pressure exceeds safe limit (e.g., >12 bar).
Low Pressure: Pressure falls below safe limit (e.g., <1 bar).
High Temperature: Temperature exceeds safe limit (e.g., >180°C).
Low Temperature: Temperature falls below safe limit (e.g., <40°C).
High Liquid Level: Liquid level exceeds safe limit (e.g., >95% of tank capacity).
Low Liquid Level: Liquid level falls below safe limit (e.g., <5% of tank capacity).
Pressure Sensor Failure: Loss of signal or invalid pressure sensor reading.
Temperature Sensor Failure: Loss of signal or invalid temperature sensor reading.


Effects:
Close Feed Valve: Shuts off inlet to stop material addition.
Stop Feed Pump: Halts feed pump to prevent material inflow.
Activate Relief System: Opens relief valve to reduce pressure.
Stop Heating System: Turns off heaters to prevent temperature rise.
Activate Cooling System: Engages cooling to lower temperature.
Issue Alarm: Activates audible/visual alarm to alert operators.
Initiate Safe Shutdown: Executes controlled reactor shutdown to safe state.


Assumptions:
Reactor has pressure, temperature, and level sensors with defined safety thresholds.
Interlocks are implemented via PLC (e.g., using Profibus DP or similar).
Actions are executed within seconds to ensure safety.
Safe shutdown is a last resort for critical conditions (e.g., sensor failures, low level).


Usage:
High Pressure: Triggers close feed valve, stop feed pump, activate relief system, stop heating, and issue alarm.
Pressure Sensor Failure: Triggers close feed valve, stop feed pump, activate relief system, stop heating, issue alarm, and initiate safe shutdown.



