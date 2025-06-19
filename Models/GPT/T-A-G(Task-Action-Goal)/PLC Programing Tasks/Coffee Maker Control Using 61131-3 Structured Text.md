FUNCTION_BLOCK FB_CoffeeMaker
VAR_INPUT
    Start           : BOOL;    // User starts the coffee maker
    CoffeeMilk      : BOOL;    // Select coffee + milk
    CoffeeOnly      : BOOL;    // Select coffee only
    EmergencyStop   : BOOL;    // Emergency stop
    MixerLevelFull  : BOOL;    // Mixer tank is full
END_VAR

VAR_OUTPUT
    CoffeeValve     : BOOL;    // Opens to add coffee
    MilkValve       : BOOL;    // Opens to add milk
    OutputValve     : BOOL;    // Dispense drink
    Mixer           : BOOL;    // Stirring mixer
END_VAR

VAR
    State           : INT := 0;           // 0: Idle, 1: Filling, 2: Mixing, 3: Dispensing
    MixTimer        : TON;                // Mixing timer
END_VAR

// --- Emergency Stop Logic: Always takes priority ---
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
    State := 0;
    MixTimer(IN := FALSE);
ELSE
    CASE State OF

    // --- State 0: Idle / Waiting for Start ---
    0:
        IF Start THEN
            IF CoffeeMilk THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
            ELSIF CoffeeOnly THEN
                CoffeeValve := TRUE;
            END_IF;
            State := 1; // Move to filling state
        END_IF;

    // --- State 1: Filling until MixerLevelFull ---
    1:
        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := TRUE;
            MixTimer(IN := TRUE, PT := T#4s);
            State := 2; // Move to mixing
        END_IF;

    // --- State 2: Mixing (4 seconds) ---
    2:
        MixTimer(IN := TRUE);
        IF MixTimer.Q THEN
            Mixer := FALSE;
            MixTimer(IN := FALSE);
            OutputValve := TRUE;
            State := 3; // Move to dispensing
        END_IF;

    // --- State 3: Dispensing (waits for manual reset) ---
    3:
        // System remains in this state until reset or Stop
        // Optional: Add automatic timeout or additional logic
    END_CASE
END_IF
