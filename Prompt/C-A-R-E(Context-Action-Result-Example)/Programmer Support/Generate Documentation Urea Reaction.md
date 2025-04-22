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

**C-A-R-E:**

🟥 C (Context) – The Situation

You have an IEC 61131-3 Structured Text (ST) program named UreaReactionControl, which manages a two-step urea synthesis reaction. It handles raw material intake (ammonia and CO₂), checks for temperature and pressure within target ranges, and times the reaction process. While the code is functional, it lacks clear developer documentation, which hinders onboarding, maintenance, and compliance.

⸻

🟩 A (Action) – What to Do

Generate detailed developer documentation for the program. The documentation should include:
	•	Program purpose
	•	Descriptions of variables: inputs, outputs, internal flags, and parameter constants
	•	Step-by-step logic flow:
	•	Step 1: Load ammonia and CO₂
	•	Step 2: Monitor and maintain reaction conditions (pressure & temperature)
	•	Completion: Final shutdown and marking the process as finished
	•	Clarify conditions for valve control and reaction timing
	•	Highlight use of CURRENT_TIME and how it tracks duration

⸻

🟨 R (Result) – What You Get

This will result in well-structured documentation that:
	•	Allows developers and control engineers to quickly understand the logic
	•	Serves as a technical reference for debugging and upgrades
	•	Supports regulatory compliance and internal quality control
	•	Increases team collaboration and reduces learning curves

⸻

🟦 E (Example) – Sample Output Format

## Developer Documentation: UreaReactionControl

### Purpose
This program automates the urea synthesis process by controlling input valves and monitoring reactor conditions. It ensures the process completes only when temperature and pressure are within tolerance for a sustained period.

### Variable Overview

#### Inputs
| Name                | Type  | Description                           |
|---------------------|--------|---------------------------------------|
| stAmmoniaValve      | BOOL | Status of the ammonia valve (read-only) |
| stCO2Valve          | BOOL | Status of the CO₂ valve (read-only)     |
| rCurrentPressure    | REAL | Reactor pressure in bar                |
| rCurrentTemperature | REAL | Reactor temperature in °C              |

#### Outputs
| Name                  | Type  | Description                              |
|-----------------------|--------|------------------------------------------|
| stAmmoniaValveControl | BOOL | TRUE = open ammonia valve                |
| stCO2ValveControl     | BOOL | TRUE = open CO₂ valve                    |

#### Parameters
| Parameter             | Value | Description                               |
|-----------------------|--------|-------------------------------------------|
| rTargetPressure       | 175.0 | Target pressure in bar                    |
| rPressureTolerance    | 5.0   | Allowed ± range for pressure              |
| rTargetTemperature    | 185.0 | Target temperature in °C                  |
| rTemperatureTolerance | 2.0   | Allowed ± range for temperature           |
| tReactionTime         | 30m   | Total reaction duration                   |

### Program Logic

1. **Material Loading (Step 1)**
   - Opens both valves
   - Waits for confirmation that both are open
   - Begins timing the reaction

2. **Reaction Control (Step 2)**
   - Monitors current pressure & temperature
   - Ensures both are within tolerance
   - Continues until the timer reaches 30 minutes

3. **Shutdown**
   - Closes all valves
   - Marks the process as `stReactionFinished := TRUE`
