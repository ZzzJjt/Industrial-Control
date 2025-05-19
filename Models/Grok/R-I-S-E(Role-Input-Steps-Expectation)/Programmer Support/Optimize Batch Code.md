TYPE
    BATCH_STATE : (
        IDLE,               // Waiting for start command
        RAW_MATERIAL_PREP,  // Preparing raw materials (e.g., ethylene, catalyst)
        POLYMERIZATION,     // Polymerizing under high pressure/temperature
        QUENCHING,          // Cooling the polymer
        DRYING,             // Removing moisture
        PELLETIZING,        // Forming pellets
        QA,                 // Quality assurance checks
        PACKAGING           // Packaging final product
    );
END_TYPE

FUNCTION_BLOCK PolyethyleneBatchControl
VAR
    (* State and control *)
    Current_State       : BATCH_STATE := IDLE;   // Current batch step
    Phase_Timer         : TON;                   // Timer for step durations
    Start_Command       : BOOL;                  // Operator start signal
    Prev_Start_Command  : BOOL;                  // For edge detection
    Batch_Complete      : BOOL;                  // Batch completion flag
    Fault_Alarm         : BOOL;                  // Fault indicator

    (* Process parameters *)
    Temp_Setpoint       : REAL;                  // Temperature setpoint (°C)
    Press_Setpoint      : REAL;                  // Pressure setpoint (bar)
    Temp_PV             : REAL;                  // Measured temperature (°C)
    Press_PV            : REAL;                  // Measured pressure (bar)

    (* Step durations *)
    Prep_Duration       : TIME := T#120s;       // Raw material prep: 2 min
    Poly_Duration       : TIME := T#600s;       // Polymerization: 10 min
    Quench_Duration     : TIME := T#180s;       // Quenching: 3 min
    Dry_Duration        : TIME := T#300s;       // Drying: 5 min
    Pellet_Duration     : TIME := T#240s;       // Pelletizing: 4 min
    QA_Duration         : TIME := T#60s;        // QA: 1 min
    Pack_Duration       : TIME := T#120s;       // Packaging: 2 min

    (* Safety limits *)
    Temp_Max            : REAL := 250.0;        // Max allowable temperature (°C)
    Press_Max           : REAL := 1200.0;       // Max allowable pressure (bar)
END_VAR

(* Centralized method to update temperature and pressure setpoints *)
METHOD PRIVATE UpdateTemperaturesAndPressures : BOOL
VAR_INPUT
    targetTemp : REAL;      // Desired temperature setpoint (°C)
    targetPress : REAL;     // Desired pressure setpoint (bar)
END_VAR
    // Update setpoints and validate conditions
    Temp_Setpoint := targetTemp;
    Press_Setpoint := targetPress;

    // Check safety limits
    IF Temp_PV > Temp_Max OR Press_PV > Press_Max THEN
        Fault_Alarm := TRUE;
        UpdateTemperaturesAndPressures := FALSE;
    ELSE
        UpdateTemperaturesAndPressures := TRUE;
    END_IF;
END_METHOD

(* Step-specific logic methods *)
METHOD PRIVATE RunRawMaterialPrep : BOOL
    // Configure for raw material preparation
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Prep_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 50.0, targetPress := 10.0);
    END_IF;
    RunRawMaterialPrep := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunPolymerization : BOOL
    // Configure for polymerization
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Poly_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 200.0, targetPress := 1000.0);
    END_IF;
    RunPolymerization := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunQuenching : BOOL
    // Configure for quenching
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Quench_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 30.0, targetPress := 50.0);
    END_IF;
    RunQuenching := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunDrying : BOOL
    // Configure for drying
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Dry_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 80.0, targetPress := 1.0);
    END_IF;
    RunDrying := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunPelletizing : BOOL
    // Configure for pelletizing
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Pellet_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 150.0, targetPress := 10.0);
    END_IF;
    RunPelletizing := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunQA : BOOL
    // Configure for quality assurance
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := QA_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 25.0, targetPress := 1.0);
    END_IF;
    RunQA := Phase_Timer.Q;
END_METHOD

METHOD PRIVATE RunPackaging : BOOL
    // Configure for packaging
    IF Phase_Timer.IN = FALSE THEN
        Phase_Timer(IN := TRUE, PT := Pack_Duration);
        UpdateTemperaturesAndPressures(targetTemp := 25.0, targetPress := 1.0);
    END_IF;
    RunPackaging := Phase_Timer.Q;
END_METHOD

(* Main cyclic execution *)
CASE Current_State OF
    IDLE:
        // Reset outputs and wait for start
        Temp_Setpoint := 0.0;
        Press_Setpoint := 0.0;
        Batch_Complete := FALSE;
        Fault_Alarm := FALSE;
        Phase_Timer(IN := FALSE);

        // Detect rising edge of Start_Command
        IF Start_Command AND NOT Prev_Start_Command THEN
            Current_State := RAW_MATERIAL_PREP;
        END_IF;

    RAW_MATERIAL_PREP:
        // Run preparation step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunRawMaterialPrep() THEN
            Current_State := POLYMERIZATION;
            Phase_Timer(IN := FALSE);
        END_IF;

    POLYMERIZATION:
        // Run polymerization step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunPolymerization() THEN
            Current_State := QUENCHING;
            Phase_Timer(IN := FALSE);
        END_IF;

    QUENCHING:
        // Run quenching step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunQuenching() THEN
            Current_State := DRYING;
            Phase_Timer(IN := FALSE);
        END_IF;

    DRYING:
        // Run drying step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunDrying() THEN
            Current_State := PELLETIZING;
            Phase_Timer(IN := FALSE);
        END_IF;

    PELLETIZING:
        // Run pelletizing step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunPelletizing() THEN
            Current_State := QA;
            Phase_Timer(IN := FALSE);
        END_IF;

    QA:
        // Run quality assurance step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunQA() THEN
            Current_State := PACKAGING;
            Phase_Timer(IN := FALSE);
        END_IF;

    PACKAGING:
        // Run packaging step
        IF Fault_Alarm THEN
            Current_State := IDLE;
        ELSIF RunPackaging() THEN
            Current_State := IDLE;
            Batch_Complete := TRUE;
            Phase_Timer(IN := FALSE);
        END_IF;
END_CASE;

(* Update edge detection *)
Prev_Start_Command := Start_Command;

(* Outputs are sent to PID controllers or actuators *)
(* Example: Write Temp_Setpoint, Press_Setpoint to analog outputs *)
END_FUNCTION_BLOCK
