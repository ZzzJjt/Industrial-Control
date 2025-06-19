Gas Turbine Interlock Matrix
This matrix defines 10 interlocks for a gas turbine, each designed to prevent hazardous conditions and protect critical components. Each interlock specifies the monitored condition, threshold, and response action.



Interlock
Safety Scenario
Monitored Condition
Threshold
Response Action



1
Overtemperature
Exhaust Gas Temperature (EGT)
>650°C
Shut down turbine, close fuel valve, trigger alarm


2
Overspeed
Turbine Shaft Speed
>3600 RPM
Shut down turbine, close fuel valve, trigger alarm


3
Flame Failure
Flame Detector Signal
No flame detected
Close fuel valve, trigger alarm, initiate purge cycle


4
Low Lube Oil Pressure
Lubrication Oil Pressure
<1.5 bar
Shut down turbine, trigger alarm


5
High Vibration
Turbine Vibration Level
>7 mm/s
Reduce load, trigger alarm; if persists >10 mm/s, shut down turbine


6
Low Fuel Pressure
Fuel Supply Pressure
<2 bar
Close fuel valve, trigger alarm, switch to backup fuel (if available)


7
High Compressor Discharge Pressure
Compressor Discharge Pressure
>20 bar
Reduce load, trigger alarm; if persists, shut down turbine


8
Low Inlet Air Pressure
Inlet Air Pressure
<0.95 bar
Trigger alarm, initiate filter cleaning; if persists, shut down turbine


9
High Bearing Temperature
Bearing Temperature
>120°C
Shut down turbine, trigger alarm


10
Emergency Stop
Manual Emergency Stop Button
Activated
Shut down turbine, close fuel valve, trigger alarm


Notes

Thresholds: Based on typical gas turbine specifications; adjust per manufacturer guidelines.
Response Actions:
Shut Down Turbine: Safely stops turbine operation via controlled ramp-down.
Close Fuel Valve: Halts fuel supply to prevent combustion.
Trigger Alarm: Alerts operators for manual intervention.
Reduce Load: Lowers turbine output to mitigate stress.
Purge Cycle: Clears residual fuel vapors to prevent explosions.
Switch to Backup Fuel: Maintains operation if primary fuel supply fails.
Filter Cleaning: Restores inlet air flow to prevent compressor issues.


Manual Override: Emergency stop (Interlock 10) allows operators to override automated logic for immediate shutdown.

