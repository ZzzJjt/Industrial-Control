FUNCTION_BLOCK CoffeeMachineControl
VAR_INPUT
    EmergencyStop  : BOOL;  // Master stop button
    Start          : BOOL;  // Start button
    CoffeeMilk     : BOOL;  // Selects coffee + milk mode
    CoffeeOnly     : BOOL;  // Selects coffee only mode
    MixerLevelFull : BOOL;  // TRUE when mixer tank is full (130 ml sensor)
END_VAR

VAR_OUTPUT
    CoffeeValve    : BOOL;  // Opens coffee valve
    MilkValve      : BOOL;  // Opens milk valve
    Mixer          : BOOL;  // Activates mixer
    OutputValve    : BOOL;  // Opens output valve
END_VAR

VAR
    State          : INT := 0; // State machine variable
    MixTimer       : TON;      // Timer for 4s mixing
END_VAR

// Emergency Stop overrides all logic
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    MixTimer(IN := FALSE);
    State := 0; // Reset to Idle
ELSE
    CASE State OF

    // 🟠 0: Idle – Wait for start and mode selection
    0:
        IF Start THEN
            IF CoffeeMilk THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
                State := 1;
            ELSIF CoffeeOnly THEN
                CoffeeValve := TRUE;
                MilkValve := FALSE;
                State := 1;
            END_IF;
        END_IF;

    // 🟡 1: Filling – Wait until mixer tank reaches full level
    1:
        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := TRUE;
            MixTimer(IN := TRUE, PT := T#4s);
            State := 2;
        END_IF;

    // 🟢 2: Mixing – Wait 4 seconds then dispense
    2:
        MixTimer(IN := TRUE);
        IF MixTimer.Q THEN
            Mixer := FALSE;
            OutputValve := TRUE;
            State := 3;
        END_IF;

    // 🔵 3: Dispensing – Stay here until reset condition
    3:
        // Optional: implement a timeout or output confirmation sensor
        // Here, auto-reset after 2 seconds for simplicity
        MixTimer(IN := TRUE, PT := T#2s);
        IF MixTimer.Q THEN
            OutputValve := FALSE;
            State := 0; // Back to Idle
        END_IF;

    END_CASE;
END_IF;
