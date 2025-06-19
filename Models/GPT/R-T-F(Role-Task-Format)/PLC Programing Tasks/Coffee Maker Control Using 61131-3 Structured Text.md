PROGRAM CoffeeMakerController
VAR_INPUT
    Start            : BOOL;
    EmergencyStop    : BOOL;
    CoffeeOnly       : BOOL;
    CoffeeMilk       : BOOL;
    MixerLevelFull   : BOOL;
END_VAR

VAR_OUTPUT
    CoffeeValve      : BOOL := FALSE;
    MilkValve        : BOOL := FALSE;
    Mixer            : BOOL := FALSE;
    OutputValve      : BOOL := FALSE;
END_VAR

VAR
    State            : INT := 0;         // 0=Idle, 1=Filling, 2=Mixing, 3=Dispensing
    MixTimer         : TON;              // IEC timer for 4s mixing
    MixTimerEnable   : BOOL := FALSE;
END_VAR

// ---------- Emergency Handling ----------
IF EmergencyStop THEN
    // Shut everything off immediately
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    MixTimerEnable := FALSE;
    State := 0; // Reset state machine to Idle
ELSE
    // ---------- State Machine ----------
    CASE State OF
        // ------------------ IDLE ------------------
        0:
            IF Start THEN
                IF CoffeeMilk THEN
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                ELSIF CoffeeOnly THEN
                    CoffeeValve := TRUE;
                    MilkValve := FALSE;
                END_IF;
                State := 1; // Move to FILLING
            END_IF;

        // ------------------ FILLING ------------------
        1:
            IF MixerLevelFull THEN
                CoffeeValve := FALSE;
                MilkValve := FALSE;
                Mixer := TRUE;
                MixTimerEnable := TRUE;
                MixTimer(IN := MixTimerEnable, PT := T#4s);
                State := 2; // Move to MIXING
            END_IF;

        // ------------------ MIXING ------------------
        2:
            MixTimer(IN := MixTimerEnable, PT := T#4s);
            IF MixTimer.Q THEN
                Mixer := FALSE;
                MixTimerEnable := FALSE;
                OutputValve := TRUE;
                State := 3; // Move to DISPENSING
            END_IF;

        // ------------------ DISPENSING ------------------
        3:
            // Optional auto-reset (e.g., after 2s dispense or sensor feedback)
            // For simplicity, return to Idle immediately
            OutputValve := FALSE;
            State := 0;
    END_CASE;
END_IF
