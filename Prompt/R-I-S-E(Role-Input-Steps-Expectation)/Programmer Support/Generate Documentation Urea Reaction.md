**Generate Documentation Urea Reaction:**

Generate a developer documentation for the following code: PROGRAM UreaReactionControl VAR // Inputs stAmmoniaValve : BOOL; // TRUE when ammonia valve is open stCO2Valve : BOOL; // TRUE when CO2 valve is open rCurrentPressure : REAL; rCurrentTemperature : REAL;
// Outputs
stAmmoniaValveControl : BOOL; // TRUE to open ammonia valve
stCO2ValveControl : BOOL; // TRUE to open CO2 valve

// Internal variables
stStep1 : BOOL := FALSE; // Load raw materials
stStep2 : BOOL := FALSE; // Control reaction
stReactionFinished : BOOL := FALSE; // Reaction finished flag

// Parameters
rTargetPressure : REAL := 175.0; // Target reactor pressure in bars
rPressureTolerance : REAL := 5.0; // Pressure tolerance in bars
rTargetTemperature : REAL := 185.0; // Target reactor temperature in �C
rTemperatureTolerance : REAL := 2.0; // Temperature tolerance in �C
tReactionTime : TIME := T#30m; // Total reaction time
tReactionTimer : TIME; // Reaction timer
END_VAR
// Main sequence control
IF NOT stReactionFinished THEN

    // Step 1: Load raw materials
    IF NOT stStep1 THEN
        stAmmoniaValveControl := TRUE;  // Control ammonia valve
        stCO2ValveControl := TRUE;      // Control CO2 valve
        
        // Check valve status
        IF stAmmoniaValve AND stCO2Valve THEN
            stStep1 := TRUE;             // Step 1 complete
            tReactionTimer := CURRENT_TIME;  // Record current time to start reaction timing
        END_IF

    // Step 2: Control reaction
    ELSIF NOT stStep2 THEN
        // Check if current pressure and temperature are within the target range
        IF (rCurrentPressure >= rTargetPressure - rPressureTolerance) AND (rCurrentPressure <= rTargetPressure + rPressureTolerance) AND 
           (rCurrentTemperature >= rTargetTemperature - rTemperatureTolerance) AND (rCurrentTemperature <= rTargetTemperature + rTemperatureTolerance) THEN
           
            // Check if the reaction time has been reached
            IF CURRENT_TIME >= tReactionTimer + tReactionTime THEN
                stStep2 := TRUE;  // Step 2 complete
            END_IF
        
        ELSE
            // Adjust valves based on pressure and temperature
            stAmmoniaValveControl := (rCurrentPressure < rTargetPressure) OR (rCurrentTemperature < rTargetTemperature);
            stCO2ValveControl := (rCurrentPressure < rTargetPressure) OR (rCurrentTemperature < rTargetTemperature);
        END_IF
    END_IF

ELSE
    // Close all valves, mark reaction as finished
    stAmmoniaValveControl := FALSE;
    stCO2ValveControl := FALSE;
    stReactionFinished := TRUE;
END_IF

END_PROGRAM

**R-I-S-E:**

🟥 R (Role) – Your Role

You are a PLC control system developer responsible for documenting an IEC 61131-3 Structured Text (ST) program titled UreaReactionControl. The goal is to create a clear, professional-level technical documentation that will help other engineers understand, maintain, and verify the logic of the urea reaction process.

⸻

🟩 I (Input) – What You’re Given

You are provided with a full ST program that:
	•	Opens ammonia and CO₂ valves to load raw materials
	•	Monitors pressure and temperature conditions inside a reactor
	•	Times the urea reaction process over a fixed period
	•	Contains internal states for tracking progress (e.g., step flags, timer, and finished flag)
	•	Uses threshold values and tolerances to determine safe operation

⸻

🟧 S (Steps) – What You Need to Do
	1.	Summarize the program’s purpose: Explain that it controls a two-phase urea reaction involving material loading and monitored reaction timing.
	2.	Describe all variables:
	•	Inputs (e.g., valve status, sensor readings)
	•	Outputs (e.g., valve control signals)
	•	Internal control flags
	•	Reaction parameters like pressure, temperature, and timing
	3.	Outline the control flow:
	•	Step 1: Open valves and wait until both confirm open
	•	Step 2: Check whether pressure and temperature are within range
	•	Start the timer and keep conditions steady until the reaction time completes
	•	Final: Close valves and set the finished flag
	4.	Clarify timer logic: Highlight how the CURRENT_TIME is used to track the duration of the reaction process.

⸻

🟦 E (Expectation) – What the Output Should Look Like

The final output should be a well-organized, developer-friendly documentation that includes:
	•	A purpose section
	•	A variable list (with types and descriptions)
	•	A logic flow explanation (step-by-step control behavior)
	•	Notes on how and when valves are triggered
	•	Description of reaction success criteria (temperature/pressure/timer)
	•	A ready-to-use reference for future development, safety checks, and compliance
