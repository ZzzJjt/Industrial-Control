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

**R-T-F:**

🟥 R (Role) – Your Role

You are a PLC automation engineer tasked with creating developer-level documentation for an IEC 61131-3 Structured Text (ST) program named UreaReactionControl. This documentation will help future engineers and integrators understand the control logic and operational workflow of the urea reaction process.

⸻

🟩 T (Task) – What You Need to Do

Develop a clear and structured developer documentation for the UreaReactionControl program that includes:
	1.	A program overview describing the purpose and sequence of the urea synthesis process.
	2.	A description of all variables, including:
	•	Inputs (sensor and valve status signals)
	•	Outputs (control signals for valves)
	•	Internal flags for step tracking
	•	Configurable parameters (pressure, temperature, timing)
	3.	A step-by-step explanation of the logic:
	•	Step 1: Raw material loading
	•	Step 2: Reaction control based on process variables
	•	Final step: Automatic shutdown upon successful completion
	4.	Clarification on how timing is managed using CURRENT_TIME and the reaction timer.

⸻

🟦 F (Format) – Output Format

Prepare the documentation in a format suitable for:
	•	Developer handbooks (Markdown, Word, or PDF)
	•	Technical audits or validation reports
	•	Inline code comments and header blocks

Include the following sections in the documentation:
	•	Program Purpose
	•	Variable Table (Inputs, Outputs, Internals, Parameters)
	•	Control Sequence Diagram or Description
	•	Timing Logic Explanation
	•	Safety and Termination Conditions
