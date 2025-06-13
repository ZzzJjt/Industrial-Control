PROGRAM CoffeeMakerControl
VAR_INPUT
    Start : BOOL; // Start button pressed
    CoffeeMilk : BOOL; // Coffee with milk option selected
    CoffeeOnly : BOOL; // Coffee only option selected
    EmergencyStop : BOOL; // Emergency stop button pressed
    MixerLevelFull : BOOL; // Sensor indicating mixer tank is full
END_VAR

VAR_OUTPUT
    CoffeeValve : BOOL; // Valve for adding coffee
    MilkValve : BOOL; // Valve for adding milk
    OutputValve : BOOL; // Valve for dispensing the drink
    Mixer : BOOL; // Mixer motor
END_VAR

VAR
    State : INT := 0; // Current state of the state machine
    MixTimer : TON; // Timer for mixing duration
END_VAR

// Main execution loop
CASE State OF
    0: // Idle
        IF NOT EmergencyStop THEN
            IF Start THEN
                IF CoffeeMilk THEN
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                ELSIF CoffeeOnly THEN
                    CoffeeValve := TRUE;
                END_IF;
                State := 1;
            END_IF;
        ELSE
            // Handle emergency stop
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            OutputValve := FALSE;
            Mixer := FALSE;
            State := 0;
        END_IF;

    1: // Filling
        IF Not EmergencyStop THEN
            IF MixerLevelFull THEN
                CoffeeValve := FALSE;
                MilkValve := FALSE;
                Mixer := TRUE;
                MixTimer(IN := TRUE, PT := T#4s);
                State := 2;
            END_IF;
        ELSE
            // Handle emergency stop
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            OutputValve := FALSE;
            Mixer := FALSE;
            State := 0;
        END_IF;

    2: // Mixing
        IF Not EmergencyStop THEN
            IF MixTimer.Q THEN
                Mixer := FALSE;
                OutputValve := TRUE;
                State := 3;
            END_IF;
        ELSE
            // Handle emergency stop
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            OutputValve := FALSE;
            Mixer := FALSE;
            State := 0;
        END_IF;

    3: // Dispensing
        IF Not EmergencyStop THEN
            // Wait for the output valve to be closed manually or automatically
            IF NOT OutputValve THEN
                State := 0;
            END_IF;
        ELSE
            // Handle emergency stop
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            OutputValve := FALSE;
            Mixer := FALSE;
            State := 0;
        END_IF;
END_CASE;

// Emergency handling
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
    State := 0;
END_IF;



