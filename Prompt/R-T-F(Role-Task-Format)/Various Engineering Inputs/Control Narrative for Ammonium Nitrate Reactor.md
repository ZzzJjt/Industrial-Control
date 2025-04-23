**Control Narrative for Ammonium Nitrate Reactor**

Create a control narrative for a reactor producing ammonium nitrates including concrete setpoints and ranges.

**R-I-S-E:**

🟥 R (Role) – Define Your Role

You are a chemical process control engineer responsible for drafting a control narrative for the ammonium nitrate reactor used in continuous production. Your narrative will support PLC/DCS programming and operator training.

⸻

🟩 I (Input) – Information Provided

You need to develop a control narrative that includes:
	•	Setpoints and acceptable operating ranges for temperature, pressure, pH, and flow ratios
	•	A list of required instrumentation and equipment, including sensors, valves, and controllers
	•	Clear operational procedures: startup, steady-state control, shutdown
	•	Defined control logic, including alarms, interlocks, and safety functions

⸻

🟧 S (Steps) – Steps to Perform
	1.	Identify key control parameters with setpoints and ranges:
	•	Reactor temperature: 175 °C ± 2 °C
	•	Reactor pressure: 4.8 bar ± 0.2 bar
	•	Ammonia:nitric acid molar ratio: 1.01:1
	•	Outlet pH: 6.2 ± 0.3
	2.	List critical equipment and instrumentation:
	•	TIC-101: Temperature control loop
	•	PIC-102: Pressure control loop
	•	FIC-103/FIC-104: Ammonia and acid flow controllers
	•	AIC-105: Inline pH analyzer
	•	ESD-106: Emergency shutdown valve
	3.	Define control procedures:
	•	Startup: Ensure all control loops are in AUTO, initiate flow with ramp-up control, preheat reactor, verify level and pressure
	•	Steady-state operation: Use ratio control between FIC-103 and FIC-104, monitor TIC-101 and PIC-102 with PID logic, adjust ammonia flow to maintain pH
	•	Shutdown: Initiate safe ramp-down, isolate feeds, depressurize reactor, open vent paths as needed
	4.	Add interlocks and alarms:
	•	ESD if temperature > 185 °C or pressure > 5.2 bar
	•	Low-pH trip if pH < 5.5 sustained for >30 seconds
	•	High-level switch disables acid feed if LSH-107 = TRUE

⸻

🟦 E (Expectation) – Expected Outcome

Deliver a structured, detailed control narrative that:
	•	Provides a clear operational guideline for reactor automation
	•	Ensures safe, efficient, and regulatory-compliant ammonium nitrate production
	•	Enables translation into ladder logic, function blocks, or ISA-88 control modules
	•	Supports integration with alarm management and HMI/SCADA systems
