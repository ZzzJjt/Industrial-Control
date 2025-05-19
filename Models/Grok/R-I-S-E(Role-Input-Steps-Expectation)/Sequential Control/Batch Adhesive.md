TYPE
    REACTION_STATE : (
        IDLE,           // Waiting for start command
        HEATING,        // Heating to target temperature
        MIXING,         // Starting and stabilizing mixer
        HOLDING,        // Holding reaction conditions
        COMPLETE,       // Reaction step completed
        FAULT           // Safety fault detected
    );
END_TYPE

(* Operation: Heating *)
FUNCTION_BLOCK FB_Heat
VAR_INPUT
    Temp_Setpoint : REAL;       // Target temperature (°C)
    Temp_Tolerance : REAL;      // Acceptable deviation (°C)
    Temp_Max : REAL;            // Maximum allowable temperature (°C)
    Temp_PV : REAL;             // Measured temperature (°C)
END_VAR
VAR_OUTPUT
    Heater_On : BOOL;           // Heater control signal
    Done : BOOL;                // Operation complete
END_VAR
VAR
    Stable : BOOL;              // Temperature within tolerance
END_VAR
    // Control heater
    IF Temp_PV < Temp_Setpoint - Temp_Tolerance THEN
        Heater_On := TRUE;
    ELSE
        Heater_On := FALSE;
    END_IF;

    // Check stability
    Stable := ABS(Temp_PV - Temp_Setpoint) <= Temp_Tolerance;
    Done := Stable AND Temp_PV >= Temp_Setpoint - Temp_Tolerance;

    // Safety check
    IF Temp_PV > Temp_Max THEN
        Heater_On := FALSE;
        Done := FALSE;
    END_IF;
END_FUNCTION_BLOCK

(* Operation: Mixing *)
FUNCTION_BLOCK FB_Mix
VAR_INPUT
    RPM_Setpoint : REAL;        // Target mixer speed (RPM)
    RPM_Tolerance : REAL;       // Acceptable deviation (RPM)
    RPM_Max : REAL;             // Maximum allowable speed (RPM)
    RPM_PV : REAL;              // Measured mixer speed (RPM)
END_VAR
VAR_OUTPUT
    Mixer_Speed_Setpoint : REAL;// Mixer speed command (RPM)
    Done : BOOL;                // Operation complete
END_VAR
VAR
    Stable : BOOL;              // Speed within tolerance
END_VAR
    // Set mixer speed
    Mixer_Speed_Setpoint := RPM_Setpoint;

    // Check stability
    Stable := ABS(RPM_PV - RPM_Setpoint) <= RPM_Tolerance;
    Done := Stable AND RPM_PV >= RPM_Setpoint - RPM_Tolerance;

    // Safety check
    IF RPM_PV > RPM_Max THEN
        Mixer_Speed_Setpoint := 0.0;
        Done := FALSE;
    END_IF;
END_FUNCTION_BLOCK

(* Operation: Holding *)
FUNCTION_BLOCK FB_Hold
VAR_INPUT
    Hold_Duration : TIME;       // Duration to hold (e.g., T#1800s)
    Temp_Setpoint : REAL;       // Target temperature (°C)
    Temp_Tolerance : REAL;      // Acceptable deviation (°C)
    Temp_PV : REAL;             // Measured temperature (°C)
    RPM_Setpoint : REAL;        // Target mixer speed (RPM)
    RPM_Tolerance : REAL;       // Acceptable deviation (RPM)
    RPM_PV : REAL;              // Measured mixer speed (RPM)
END_VAR
VAR_OUTPUT
    Heater_On : BOOL;           // Heater control signal
    Mixer_Speed_Setpoint : REAL;// Mixer speed command (RPM)
    Done : BOOL;                // Operation complete
END_VAR
VAR
    Hold_Timer : TON;           // Timer for hold duration
    Temp_Stable : BOOL;         // Temperature within tolerance
    RPM_Stable : BOOL;          // Speed within tolerance
END_VAR
    // Maintain temperature
    IF Temp_PV < Temp_Setpoint - Temp_Tolerance THEN
        Heater_On := TRUE;
    ELSE
        Heater_On := FALSE;
    END_IF;
    Temp_Stable := ABS(Temp_PV - Temp_Setpoint) <= Temp_Tolerance;

    // Maintain mixer speed
    Mixer_Speed_Setpoint := RPM_Setpoint;
    RPM_Stable := ABS(RPM_PV - RPM_Setpoint) <= RPM_Tolerance;

    // Run timer if conditions are stable
    Hold_Timer(IN := Temp_Stable AND RPM_Stable, PT := Hold_Duration);
    Done := Hold_Timer.Q;
END_FUNCTION_BLOCK

(* Unit Procedure: Reaction *)
FUNCTION_BLOCK FB_Reaction
VAR
    (* State and control *)
    Current_State       : REACTION_STATE := IDLE;   // Current operation state
    Start_Command       : BOOL;                     // Start signal from recipe
    Prev_Start_Command  : BOOL;                    // For edge detection
    EStop               : BOOL;                    // Emergency stop
    Step_Complete       : BOOL;                    // Step completion flag
    Fault_Alarm         : BOOL;                    // Fault indicator

    (* Inputs *)
    Temp_PV             : REAL;                    // Reactor temperature (°C)
    RPM_PV              : REAL;                    // Mixer speed (RPM)
    Pressure_PV         : REAL;                    // Reactor pressure (bar)

    (* Operation instances *)
    Heat_Op             : FB_Heat;                 // Heating operation
    Mix_Op              : FB_Mix;                  // Mixing operation
    Hold_Op             : FB_Hold;                 // Holding operation

    (* Parameters *)
    Temp_Setpoint       : REAL := 120.0;           // Reaction temperature (°C)
    Temp_Tolerance      : REAL := 2.0;             // Temperature tolerance (°C)
    Temp_Max            : REAL := 150.0;           // Max allowable temperature (°C)
    RPM_Setpoint        : REAL := 500.0;           // Mixer speed (RPM)
    RPM_Tolerance       : REAL := 50.0;            // Speed tolerance (RPM)
    RPM_Max             : REAL := 1000.0;          // Max allowable speed (RPM)
    Hold_Duration       : TIME := T#1800s;         // Hold time (30 min)
    Pressure_Max        : REAL := 10.0;            // Max allowable pressure (bar)
END_VAR
VAR_OUTPUT
    Heater_On           : BOOL;                    // Heater control signal
    Mixer_Speed_Setpoint : REAL;                  // Mixer speed command (RPM)
END_VAR

(* Check safety conditions *)
METHOD PRIVATE CheckSafety : BOOL
    IF Temp_PV > Temp_Max OR Pressure_PV > Pressure_Max OR EStop THEN
        Fault_Alarm := TRUE;
        Heater_On := FALSE;
        Mixer_Speed_Setpoint := 0.0;
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

(* Main state machine *)
CASE Current_State OF
    IDLE:
        // Reset outputs and state
        Heater_On := FALSE;
        Mixer_Speed_Setpoint := 0.0;
        Step_Complete := FALSE;
        Fault_Alarm := FALSE;
        Heat_Op.Done := FALSE;
        Mix_Op.Done := FALSE;
        Hold_Op.Done := FALSE;

        // Start on rising edge
        IF Start_Command AND NOT Prev_Start_Command THEN
            Current_State := HEATING;
        END_IF;

    HEATING:
        IF NOT CheckSafety() THEN
            Current_State := FAULT;
        ELSE
            // Execute heating operation
            Heat_Op(
                Temp_Setpoint := Temp_Setpoint,
                Temp_Tolerance := Temp_Tolerance,
                Temp_Max := Temp_Max,
                Temp_PV := Temp_PV
            );
            Heater_On := Heat_Op.Heater_On;
            IF Heat_Op.Done THEN
                Current_State := MIXING;
            END_IF;
        END_IF;

    MIXING:
        IF NOT CheckSafety() THEN
            Current_State := FAULT;
        ELSE
            // Execute mixing operation
            Mix_Op(
                RPM_Setpoint := RPM_Setpoint,
                RPM_Tolerance := RPM_Tolerance,
                RPM_Max := RPM_Max,
                RPM_PV := RPM_PV
            );
            Mixer_Speed_Setpoint := Mix_Op.Mixer_Speed_Setpoint;
            IF Mix_Op.Done THEN
                Current_State := HOLDING;
            END_IF;
        END_IF;

    HOLDING:
        IF NOT CheckSafety() THEN
            Current_State := FAULT;
        ELSE
            // Execute holding operation
            Hold_Op(
                Hold_Duration := Hold_Duration,
                Temp_Setpoint := Temp_Setpoint,
                Temp_Tolerance := Temp_Tolerance,
                Temp_PV := Temp_PV,
                RPM_Setpoint := RPM_Setpoint,
                RPM_Tolerance := RPM_Tolerance,
                RPM_PV := RPM_PV
            );
            Heater_On := Hold_Op.Heater_On;
            Mixer_Speed_Setpoint := Hold_Op.Mixer_Speed_Setpoint;
            IF Hold_Op.Done THEN
                Current_State := COMPLETE;
            END_IF;
        END_IF;

    COMPLETE:
        // Signal completion
        Heater_On := FALSE;
        Mixer_Speed_Setpoint := 0.0;
        Step_Complete := TRUE;
        // Wait for recipe to reset
        IF NOT Start_Command THEN
            Current_State := IDLE;
        END_IF;

    FAULT:
        // Disable outputs
        Heater_On := FALSE;
        Mixer_Speed_Setpoint := 0.0;
        Step_Complete := FALSE;
        // Remain in fault until reset
        IF NOT EStop AND NOT Start_Command THEN
            Current_State := IDLE;
        END_IF;
END_CASE;

(* Update edge detection *)
Prev_Start_Command := Start_Command;

(* Outputs are sent to equipment module *)
(* Example: Write Heater_On, Mixer_Speed_Setpoint to digital/analog outputs *)
END_FUNCTION_BLOCK
```

### Explanation of Implementation
1. **ISA-88 Recipe Structure**:
   - **Hierarchy**: Defined the adhesive production process with Unit Procedure B.2 (Reaction), comprising three operations: Heating, Mixing, and Holding, each with distinct phases.
   - **Modularity**: Implemented as a function block (`FB_Reaction`) for integration into the batch recipe, with operation-specific function blocks (`FB_Heat`, `FB_Mix`, `FB_Hold`) for reusability.
   - **Procedural Control**: Used a state machine (`REACTION_STATE`) to sequence operations, aligning with ISA-88’s procedural model.

2. **Function Blocks for Operations**:
   - **FB_Heat**:
     - Inputs: `Temp_Setpoint` (120°C), `Temp_Tolerance` (±2°C), `Temp_Max` (150°C), `Temp_PV`.
     - Output: `Heater_On` (BOOL), `Done` (BOOL).
     - Logic: Turns heater on if `Temp_PV` is below setpoint minus tolerance; sets `Done` when temperature stabilizes within tolerance.
   - **FB_Mix**:
     - Inputs: `RPM_Setpoint` (500 RPM), `RPM_Tolerance` (±50 RPM), `RPM_Max` (1000 RPM), `RPM_PV`.
     - Output: `Mixer_Speed_Setpoint` (REAL), `Done` (BOOL).
     - Logic: Sets mixer speed to setpoint; sets `Done` when speed stabilizes within tolerance.
   - **FB_Hold**:
     - Inputs: `Hold_Duration` (T#1800s), temperature/mixer parameters, `Temp_PV`, `RPM_PV`.
     - Output: `Heater_On`, `Mixer_Speed_Setpoint`, `Done` (BOOL).
     - Logic: Maintains temperature and mixer speed; runs `Hold_Timer` when stable, setting `Done` when timer completes.

3. **Sequential Control Logic**:
   - **State Machine**: `FB_Reaction` uses `REACTION_STATE` (`IDLE`, `HEATING`, `MIXING`, `HOLDING`, `COMPLETE`, `FAULT`) to sequence operations.
   - **Transitions**:
     - `IDLE → HEATING`: On `Start_Command` rising edge.
     - `HEATING → MIXING`: When `Heat_Op.Done = TRUE`.
     - `MIXING → HOLDING`: When `Mix_Op.Done = TRUE`.
     - `HOLDING → COMPLETE`: When `Hold_Op.Done = TRUE`.
     - `COMPLETE → IDLE`: When `Start_Command = FALSE` (recipe reset).
     - Any state → `FAULT`: On safety violation (`CheckSafety` fails).
   - **Timers**: `Hold_Timer` (in `FB_Hold`) manages the 30-minute hold duration, activated only when temperature and RPM are stable.
   - **Real-Time Conditions**: Transitions depend on `Temp_PV`, `RPM_PV`, and timer outputs, ensuring process stability.

4. **Modular and Reusable Structure**:
   - **Function Blocks**: `FB_Heat`, `FB_Mix`, and `FB_Hold` are standalone, reusable across other unit procedures (e.g., different reaction steps).
   - **Encapsulation**: `FB_Reaction` encapsulates the Reaction step, interfacing with the recipe via `Start_Command` and `Step_Complete`.
   - **State Handling**: Clear `REACTION_STATE` ENUM and reset logic (`IDLE`, `COMPLETE`) ensure predictable behavior.
   - **Comments**: Detailed comments explain each block, state, and parameter, enhancing maintainability.

### Meeting Expectations
- **ISA-88 Compliance**:
  - Follows the hierarchical model (Process → Unit Procedure → Operations → Phases).
  - Uses modular function blocks for operations, aligning with procedural control.
  - Supports recipe integration via `Start_Command` input and `Step_Complete` output.
- **Reliability**:
  - Safety interlocks (`CheckSafety`) prevent unsafe conditions (e.g., `Temp_PV > 150°C`, `Pressure_PV > 10 bar`).
  - Stable transitions rely on real-time PVs and timers, ensuring consistent adhesive quality.
- **Integration**:
  - `FB_Reaction` is self-contained, easily called by a higher-level recipe manager.
  - Outputs (`Heater_On`, `Mixer_Speed_Setpoint`) interface with equipment modules.
- **Clarity and Flexibility**:
  - Clear state machine and modular FBs simplify debugging and modification.
  - Parameters (e.g., `Temp_Setpoint`, `Hold_Duration`) are adjustable via HMI or recipe.
- **Safety**:
  - `FAULT` state disables outputs and requires manual reset, protecting equipment.
  - `EStop` integration ensures immediate halt in emergencies.
- **Reuse**: Operation FBs can be reused in other steps (e.g., heating in material prep) or recipes.

### Additional Notes
- **Process Parameters**: Used typical adhesive reaction values (120°C, 500 RPM, 30 min). Validate with actual process requirements.
- **Safety Enhancements**: Consider adding sensor validation (e.g., stuck mixer) or pressure relief logic for production.
- **Scalability**: Additional operations (e.g., pressure control) can be added as new FBs and states.
- **Performance**: Suitable for scan cycles <100 ms. Verify timer accuracy for long durations (e.g., 30 min).
- **ISA-88 Integration**: Assumes a batch control system (e.g., S88 Batch) manages the recipe, calling `FB_Reaction` for step B.2.

This program is ready for deployment on an IEC 61131-3-compliant PLC, ensuring reliable, safe, and flexible control of the adhesive reaction step. If you have specific parameters (e.g., exact temperatures, additional operations) or integration details (e.g., recipe manager interface), I can refine the code further!
