PROGRAM CoffeeMachineControl
VAR_INPUT
    EmergencyStop : BOOL; // Instantly stops all operations
    Start : BOOL;         // Begins the coffee-making process
    CoffeeMilk : BOOL;    // Prepares coffee with milk
    CoffeeOnly : BOOL;    // Prepares coffee without milk
    MixerLevelFull : BOOL; // Indicates if the mixer tank is full (130ml)
END_VAR

VAR_OUTPUT
    CoffeeValve : BOOL;   // Controls the coffee valve
    MilkValve : BOOL;     // Controls the milk valve
    Mixer : BOOL;         // Controls the mixer unit
    OutputValve : BOOL;   // Controls the output valve
END_VAR

VAR
    State : INT := 0;      // State variable to manage the state transitions
    MixTimer : TON;        // Timer for mixing duration
END_VAR

// Reset valves and mixer in case of emergency stop
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    State := 0; // Go back to idle state
ELSE
    CASE State OF
        0: // Idle
            IF Start THEN
                IF CoffeeMilk THEN
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                    State := 1; // Move to Filling state
                ELSIF CoffeeOnly THEN
                    CoffeeValve := TRUE;
                    State := 1; // Move to Filling state
                END_IF;
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
            // Optional: Reset after dispensing
            OutputValve := FALSE;
            State := 0; // Return to Idle state
    END_CASE;
END_IF;

// Inline comments explaining the logic:
// The program uses a state machine to control the coffee-making process.
// States include Idle, Filling, Mixing, and Dispensing.
// The EmergencyStop button overrides all states, halting all operations immediately.
// Valves and mixer are controlled based on the current state and sensor inputs.
// The MixTimer ensures the mixer runs for exactly 4 seconds before moving to the Dispensing state.



