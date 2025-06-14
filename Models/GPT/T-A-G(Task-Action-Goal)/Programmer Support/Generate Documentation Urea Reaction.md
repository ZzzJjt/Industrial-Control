📌 Overview

The UreaReactionControl program automates the controlled synthesis of urea by managing the operation of ammonia and carbon dioxide (CO₂) inlet valves, monitoring reactor pressure and temperature, and enforcing timed reaction conditions. The logic is structured in two sequential steps:
	1.	Material Loading Phase – Ensures the valves open and material begins entering the reactor.
	2.	Reaction Control Phase – Monitors critical parameters (pressure, temperature), manages reaction timing, and verifies completion conditions.

This logic ensures safe and reliable chemical reaction progression in urea production.

⸻

🧩 Variable Definitions

🔹 Inputs

Name
Type
Description
Ammonia_Valve_In
BOOL
Feedback signal: Ammonia valve is open
CO2_Valve_In
BOOL
Feedback signal: CO₂ valve is open
Pressure_PV
REAL
Measured pressure in the reactor (bar)
Temperature_PV
REAL
Measured reactor temperature (°C)
CURRENT_TIME
TIME
System time used to track elapsed duration

🔸 Internal Variables

Name
Type
Description
Step
INT
Step control: 0 = idle, 1 = loading, 2 = reacting
Timer_Start
TIME
Timestamp when the reaction timer starts
Elapsed_Time
TIME
Time elapsed since reaction started

🔁 Sequential Logic Explanation

▶ Step 0: Idle Initialization
	•	Waits for system reset or start conditions.
	•	Reaction_Complete := FALSE.

▶ Step 1: Material Loading
	•	Actions:
	•	Open both Ammonia_Valve_Out and CO2_Valve_Out.
	•	Transition Conditions:
	•	Wait for Ammonia_Valve_In = TRUE and CO2_Valve_In = TRUE as confirmation.
	•	Once confirmed, advance to step 2.

▶ Step 2: Reaction Monitoring
	•	Monitoring:
	•	Continuously check:
	•	ABS(Pressure_PV - Pressure_Setpoint) ≤ Pressure_Tolerance
	•	ABS(Temperature_PV - Temp_Setpoint) ≤ Temp_Tolerance
	•	Timing:
	•	On first entry into step 2, store Timer_Start := CURRENT_TIME.
	•	Compute Elapsed_Time := CURRENT_TIME - Timer_Start.
	•	Completion:
	•	If all conditions are met and Elapsed_Time ≥ Reaction_Duration, set Reaction_Complete := TRUE.

⸻

⚙️ Control Conditions and Safety Handling
	•	Valve Confirmation: Ensures process safety by only proceeding when actual valve states confirm open status.
	•	Parameter Tolerances: Prevents premature reaction validation due to sensor noise.
	•	Reaction Timer: Uses CURRENT_TIME to guarantee a minimum sustained reaction window, avoiding undershoot.
	•	Failsafe: If at any point pressure or temperature deviates beyond limits, the reaction is not marked complete.

 Pressure_Setpoint := 160.0;          // bar
Temp_Setpoint := 185.0;              // °C
Pressure_Tolerance := 2.5;           // ± bar
Temp_Tolerance := 1.5;               // ± °C
Reaction_Duration := T#600s;         // 10 minutes


