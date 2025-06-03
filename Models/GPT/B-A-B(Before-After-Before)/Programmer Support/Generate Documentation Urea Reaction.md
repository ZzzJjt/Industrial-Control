Step 1: System Start and Prechecks
	•	Triggered by Start = TRUE.
	•	Program checks that Temp_PV and Press_PV are within safe operating limits:
	•	Temp_Min ≤ Temp_PV ≤ Temp_Max
	•	Press_Min ≤ Press_PV ≤ Press_Max
	•	If all checks pass:
	•	Open both Valve_NH3 and Valve_CO2
	•	Set ReactionStarted := TRUE
	•	Save StartTime := CURRENT_TIME

⸻

▶ Step 2: Reaction Monitoring
	•	While the reaction is ongoing:
	•	Continue monitoring pressure and temperature.
	•	If values move outside allowable bounds:
	•	Close both valves
	•	Set ReactionDone := TRUE
	•	Exit process

⸻

✅ Step 3: Reaction Completion
	•	Compute elapsed time:
	•	Elapsed := CURRENT_TIME - StartTime
	•	If Elapsed ≥ ReactionDuration:
	•	Close Valve_NH3 and Valve_CO2
	•	Set ReactionDone := TRUE
	•	Set ReactionStarted := FALSE
