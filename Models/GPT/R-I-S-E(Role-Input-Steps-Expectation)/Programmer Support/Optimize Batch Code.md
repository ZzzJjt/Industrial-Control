FUNCTION_BLOCK PolyethyleneBatchControl
VAR
    Step : INT := 0; // Batch state machine
    Timer : TON; // Step duration timer
    Timer_Duration : TIME;

    // Process conditions
    Temperature_SP : REAL;
    Pressure_SP : REAL;

    // Internal flag to detect state entry
    StepChanged : BOOL := TRUE;
END_VAR

METHOD PRIVATE UpdateTemperaturesAndPressures
VAR_INPUT
    StepIdx : INT;
END_VAR
CASE StepIdx OF
    0: // Raw Material Prep
        Temperature_SP := 30.0;
        Pressure_SP := 1.0;
    1: // Polymerization
        Temperature_SP := 160.0;
        Pressure_SP := 5.0;
    2: // Quenching
        Temperature_SP := 80.0;
        Pressure_SP := 2.0;
    3: // Drying
        Temperature_SP := 110.0;
        Pressure_SP := 1.2;
    4: // Pelletizing
        Temperature_SP := 90.0;
        Pressure_SP := 1.0;
    5: // Quality Check
        Temperature_SP := 25.0;
        Pressure_SP := 1.0;
    6: // Packaging
        Temperature_SP := 22.0;
        Pressure_SP := 1.0;
END_CASE
END_METHOD

METHOD PRIVATE GetStepDuration : TIME
VAR_INPUT
    StepIdx : INT;
END_VAR
CASE StepIdx OF
    0: GetStepDuration := T#30S;  // Prep
    1: GetStepDuration := T#5M;   // Polymerization
    2: GetStepDuration := T#1M;   // Quenching
    3: GetStepDuration := T#3M;   // Drying
    4: GetStepDuration := T#2M;   // Pelletizing
    5: GetStepDuration := T#1M;   // QA
    6: GetStepDuration := T#1M;   // Packaging
ELSE
    GetStepDuration := T#0S;
END_CASE
END_METHOD

// Main cyclic logic
IF StepChanged THEN
    // Configure process settings
    UpdateTemperaturesAndPressures(Step);
    Timer_Duration := GetStepDuration(Step);

    // Initialize timer
    Timer(IN := TRUE, PT := Timer_Duration);
    StepChanged := FALSE;
ELSE
    Timer(IN := TRUE); // Continue timing
END_IF

Timer(); // Evaluate timer

IF Timer.Q THEN
    // Advance to next step
    Step := Step + 1;
    StepChanged := TRUE;
    Timer(IN := FALSE); // Stop current timer
END_IF

IF Step > 6 THEN
    // Process complete, reset
    Step := 0;
    StepChanged := TRUE;
    Timer(IN := FALSE);
END_IF
