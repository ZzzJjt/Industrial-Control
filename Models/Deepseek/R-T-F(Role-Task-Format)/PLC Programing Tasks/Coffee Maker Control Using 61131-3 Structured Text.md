FUNCTION_BLOCK CoffeeMakerController
VAR_INPUT
    Start : BOOL;              // Initiates the brewing process
    EmergencyStop : BOOL;      // Stops all operations immediately
    CoffeeMilk : BOOL;         // Selects drink with both coffee and milk
    CoffeeOnly : BOOL;         // Selects drink with only coffee
    MixerLevelFull : BOOL;     // Detects when the mixer tank is full
END_VAR

VAR_OUTPUT
    CoffeeValve : BOOL;        // Opens to add coffee to mixer
    MilkValve : BOOL;          // Opens to add milk to mixer
    OutputValve : BOOL;        // Opens to dispense final mixed drink
    Mixer : BOOL;              // Runs for 4 seconds once tank is full
END_VAR

VAR
    State : INT := 0;           // Current state of the machine
    MixTimer : TON;             // Timer for mixing duration (4 seconds)
    ResetRequired : BOOL := FALSE; // Flag to indicate if reset is required after emergency stop
END_VAR

METHOD Execute : BOOL
BEGIN
    // Handle Emergency Stop condition
    IF EmergencyStop THEN
        CoffeeValve := FALSE;
        MilkValve := FALSE;
        Mixer := FALSE;
        OutputValve := FALSE;
        State := 0; // Go to idle state
        ResetRequired := TRUE; // Set flag to require manual reset
        RETURN TRUE;
    END_IF;

    // Check if reset is required before proceeding
    IF ResetRequired THEN
        RETURN TRUE; // Stay in idle state until manually reset
    END_IF;

    // State Machine
    CASE State OF
        0: // Idle
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := FALSE;
            OutputValve := FALSE;

            IF Start THEN
                IF CoffeeMilk THEN
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                ELSIF CoffeeOnly THEN
                    CoffeeValve := TRUE;
                END_IF;
                State := 1; // Move to Filling state
            END_IF;

        1: // Filling
            IF MixerLevelFull THEN
                CoffeeValve := FALSE;
                MilkValve := FALSE;
                Mixer := TRUE;
                MixTimer(IN := TRUE, PT := T#4s);
                State := 2; // Move to Mixing state
            END_IF;

        2: // Mixing
            IF MixTimer.Q THEN
                Mixer := FALSE;
                OutputValve := TRUE;
                State := 3; // Move to Dispensing state
            END_IF;

        3: // Dispensing
            IF NOT MixerLevelFull THEN // Assuming MixerLevelFull goes low when dispensing is complete
                OutputValve := FALSE;
                State := 0; // Return to Idle state
            END_IF;

        OTHERS:
            State := 0; // Default to Idle state in case of unexpected state
    END_CASE;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
