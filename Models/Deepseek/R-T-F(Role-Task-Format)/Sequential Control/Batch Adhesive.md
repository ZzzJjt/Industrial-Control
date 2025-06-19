PROGRAM AdhesiveProduction_Reaction
VAR
    // State management variables
    currentState : INT := 0; // 0: Idle, 1: Heating, 2: Mixing, 3: Holding, 4: Complete

    // Process parameters
    TargetTemperature : REAL := 85.0; // Target temperature in Celsius
    MixingSpeed : REAL := 100.0; // Mixing speed in RPM
    HoldTime : TIME := T#15m; // Hold time for reaction

    // Timers
    HeatingTimer : TON;
    MixingTimer : TON;
    HoldTimer : TON;

    // Flags
    HeatingComplete : BOOL := FALSE;
    MixingComplete : BOOL := FALSE;
    HoldingComplete : BOOL := FALSE;
END_VAR

// Method declarations
METHOD StartHeating(this : REFERENCE TO AdhesiveProduction_Reaction) : BOOL
BEGIN
    // Implement heating logic here
    // Example: SetTemperature(TargetTemperature);
    RETURN TRUE;
END_METHOD

METHOD StartMixing(this : REFERENCE TO AdhesiveProduction_Reaction) : BOOL
BEGIN
    // Implement mixing logic here
    // Example: SetMixingSpeed(MixingSpeed);
    RETURN TRUE;
END_METHOD

METHOD MaintainEnvironment(this : REFERENCE TO AdhesiveProduction_Reaction) : BOOL
BEGIN
    // Implement environment maintenance logic here
    // Example: CheckTemperatureAndSpeed();
    RETURN TRUE;
END_METHOD

// Main control loop
CASE currentState OF
    0: // Idle state
        IF StartReaction THEN
            currentState := 1; // Transition to Heating state
        END_IF;

    1: // Heating state
        StartHeating(); // Start heating process
        HeatingTimer(IN := TRUE, PT := T#10m); // Assume heating takes 10 minutes
        IF HeatingTimer.Q THEN
            HeatingTimer(IN := FALSE);
            HeatingComplete := TRUE;
            currentState := 2; // Transition to Mixing state
        END_IF;

    2: // Mixing state
        StartMixing(); // Start mixing process
        MixingTimer(IN := TRUE, PT := T#5m); // Assume mixing takes 5 minutes
        IF MixingTimer.Q THEN
            MixingTimer(IN := FALSE);
            MixingComplete := TRUE;
            currentState := 3; // Transition to Holding state
        END_IF;

    3: // Holding state
        MaintainEnvironment(); // Maintain reaction environment
        HoldTimer(IN := TRUE, PT := HoldTime);
        IF HoldTimer.Q THEN
            HoldTimer(IN := FALSE);
            HoldingComplete := TRUE;
            currentState := 4; // Transition to Complete state
        END_IF;

    4: // Complete state
        // Reaction complete, transition back to Idle
        currentState := 0;

    ELSE:
        currentState := 0; // Reset to Idle state in case of unexpected state
END_CASE;

// In-line comments explaining transitions and control logic
// - The program starts in the Idle state and transitions to Heating when StartReaction is triggered.
// - During Heating, a timer is used to ensure the heating duration is met before transitioning to Mixing.
// - During Mixing, another timer ensures the mixing duration is met before transitioning to Holding.
// - During Holding, the environment is maintained for the specified hold time using a timer.
// - Once all phases are complete, the program transitions back to the Idle state.
