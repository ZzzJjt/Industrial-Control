FUNCTION_BLOCK FB_CoffeeMachineController
VAR_INPUT
    Start            : BOOL;  // Start button
    EmergencyStop    : BOOL;  // Emergency stop button
    CoffeeMilk       : BOOL;  // Mode: Coffee with milk
    CoffeeOnly       : BOOL;  // Mode: Coffee only
    MixerLevelFull   : BOOL;  // TRUE when mixer reaches 130 ml
END_VAR

VAR_OUTPUT
    CoffeeValve      : BOOL;  // Control coffee valve
    MilkValve        : BOOL;  // Control milk valve
    OutputValve      : BOOL;  // Control output valve
    Mixer            : BOOL;  // Control mixer motor
    ErrorFlag        : BOOL;  // Optional for future diagnostics
END_VAR

VAR
    State            : INT := 0;        // 0=IDLE, 1=FILLING, 2=MIXING, 3=DISPENSING
    MixingTimer      : TON;            // 4-second mixer timer
    MixingStart      : BOOL := FALSE;  // Internal trigger for TON
END_VAR

// Emergency Stop override: immediate shutdown
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
    MixingStart := FALSE;
    MixingTimer(IN := FALSE);
    State := 0; // Reset to IDLE
    RETURN;
END_IF

// Main State Machine
CASE State OF

    // === IDLE STATE ===
    0:
        CoffeeValve := FALSE;
        MilkValve := FALSE;
        OutputValve := FALSE;
        Mixer := FALSE;
        MixingStart := FALSE;
        MixingTimer(IN := FALSE);

        IF Start AND (CoffeeOnly OR CoffeeMilk) THEN
            State := 1; // Proceed to FILLING
        END_IF;

    // === FILLING STATE ===
    1:
        CoffeeValve := TRUE;
        MilkValve := CoffeeMilk; // Only open MilkValve if CoffeeMilk selected

        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            MixingStart := TRUE;
            MixingTimer(IN := TRUE, PT := T#4s);
            State := 2; // Proceed to MIXING
        END_IF;

    // === MIXING STATE ===
    2:
        Mixer := TRUE;
        MixingTimer(IN := MixingStart);

        IF MixingTimer.Q THEN
            Mixer := FALSE;
            MixingStart := FALSE;
            MixingTimer(IN := FALSE);
            State := 3; // Proceed to DISPENSING
        END_IF;

    // === DISPENSING STATE ===
    3:
        OutputValve := TRUE;

        // Optional: Add sensor or time limit here to auto-close OutputValve
        // For now, return to IDLE manually via Start button or next cycle
        IF NOT Start THEN
            OutputValve := FALSE;
            State := 0;
        END_IF;

END_CASE
