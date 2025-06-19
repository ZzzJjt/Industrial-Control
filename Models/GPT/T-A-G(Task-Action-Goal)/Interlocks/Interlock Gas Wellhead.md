Gas Turbine Interlock System for Power Plant Safety

⸻

1. Overtemperature Interlock
	•	Monitored Parameter: Exhaust Gas Temperature (EGT)
	•	Threshold: > 650°C
	•	Action: Trip turbine, close fuel valve, initiate cooling sequence
	•	Manual Override: Emergency Stop (E-Stop) button

2. Overspeed Interlock
	•	Monitored Parameter: Turbine Shaft Speed
	•	Threshold: > 110% of nominal RPM
	•	Action: Trip turbine, close inlet guide vanes, raise alarm
	•	Manual Override: Mechanical centrifugal trip or E-Stop

3. Flame Failure Interlock
	•	Monitored Parameter: Flame Detector Signal
	•	Threshold: No flame detected within 5 seconds of ignition
	•	Action: Close fuel valves, lock out restart
	•	Manual Override: Manual reset after cause investigation

4. Lubrication Oil Pressure Interlock
	•	Monitored Parameter: Lube Oil Pressure
	•	Threshold: < 1.5 bar
	•	Action: Trip turbine, activate oil backup pump
	•	Manual Override: Not permitted; automatic reset only when safe

5. Bearing Temperature Interlock
	•	Monitored Parameter: Bearing Metal Temperature
	•	Threshold: > 120°C
	•	Action: Trip turbine, initiate bearing cooling fans
	•	Manual Override: Manual review before restart

6. Generator Differential Protection Interlock
	•	Monitored Parameter: Current imbalance between generator windings
	•	Threshold: > preset differential current value
	•	Action: Trip turbine-generator set, isolate electrical output
	•	Manual Override: Lockout reset via operator panel

7. Vibration Level Interlock
	•	Monitored Parameter: Shaft Vibration (overall RMS)
	•	Threshold: > 10 mm/s
	•	Action: Trip turbine, notify maintenance
	•	Manual Override: Requires vibration analysis and supervisor approval

8. Fuel Gas Pressure Low Interlock
	•	Monitored Parameter: Fuel Gas Supply Pressure
	•	Threshold: < 3 bar
	•	Action: Close main fuel valve, trip turbine
	•	Manual Override: E-Stop applicable if pressure surge risk

9. Air Intake Filter Differential Pressure Interlock
	•	Monitored Parameter: Filter dP
	•	Threshold: > 250 mmH2O
	•	Action: Raise alarm, switch to redundant filter bank
	•	Manual Override: Maintenance mode can inhibit alarm temporarily

10. Hydraulic Control Oil Contamination Interlock
	•	Monitored Parameter: Oil particle count or water content
	•	Threshold: Above ISO 4406 code limit or > 500 ppm water
	•	Action: Isolate hydraulic control system, raise critical alarm
	•	Manual Override: Not permitted until oil is replaced and retested

⸻

Integration with PLC/DCS:
Each interlock is implemented as part of a fail-safe logic group in the turbine’s PLC or DCS. Structured Text routines in IEC 61131-3 define signal comparisons, time delays, and action triggers. Redundant sensing and diagnostic confirmation are employed to reduce false trips. All interlocks write to a safety log with timestamps and alarm codes.

Safety Outcome:
This interlock system ensures that hazardous deviations in gas turbine operation are detected and acted upon in real-time, preventing equipment failure, fire hazards, or personnel injury. The system design complies with ISO 61285 and IEC 61508 safety standards.
