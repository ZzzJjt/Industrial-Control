VAR
    // === Equipment Phase States ===
    Phase_Reaction     : INT := 0; // 0:Idle, 1:Heating, 2:Mixing, 3:Hold, 4:Done
    Phase_Crystallize  : INT := 0;
    Phase_Drying       : INT := 0;

    // === Parameters ===
    ReactionTemp       : REAL := 70.0;         // °C
    ReactionPressure   : REAL := 1.2;          // bar
    ReactionHoldTime   : TIME := T#30m;

    CrystallizeTemp    : REAL := 10.0;         // °C
    CrystallizeTime    : TIME := T#20m;

    DryingTemp         : REAL := 90.0;         // °C
    DryingTime         : TIME := T#45m;

    // === Process Values ===
    CurrentTemp        : REAL;
    CurrentPressure    : REAL;

    // === Control Flags ===
    ReactionComplete   : BOOL := FALSE;
    CrystallizeComplete: BOOL := FALSE;
    DryingComplete     : BOOL := FALSE;

    // === Timers ===
    ReactionTimer      : TON;
    CrystallizeTimer   : TON;
    DryingTimer        : TON;

    // === Internal Control ===
    ReactionTimerStart     : BOOL := FALSE;
    CrystallizeTimerStart  : BOOL := FALSE;
    DryingTimerStart       : BOOL := FALSE;

    // === External Method Interfaces (Assumed Implemented Elsewhere) ===
    // PROCEDURE StartHeating(TargetTemp: REAL);
    // PROCEDURE StartMixing();
    // PROCEDURE StopMixing();
    // PROCEDURE SetDryingHeater(Enable: BOOL);
END_VAR

// === REACTION PHASE ===
CASE Phase_Reaction OF

    0: // Idle - waiting to start
        ReactionComplete := FALSE;

    1: // Heat to target temperature
        StartHeating(TargetTemp := ReactionTemp);
        IF CurrentTemp >= ReactionTemp THEN
            Phase_Reaction := 2;
        END_IF;

    2: // Begin mixing
        StartMixing();
        Phase_Reaction := 3;

    3: // Hold for reaction time
        IF NOT ReactionTimerStart THEN
            ReactionTimer(IN := TRUE, PT := ReactionHoldTime);
            ReactionTimerStart := TRUE;
        END_IF;

        IF ReactionTimer.Q THEN
            StopMixing();
            ReactionTimer(IN := FALSE);
            ReactionTimerStart := FALSE;
            Phase_Reaction := 4;
        END_IF;

    4: // Done
        ReactionComplete := TRUE;

END_CASE;

// === CRYSTALLIZATION PHASE ===
IF ReactionComplete THEN
    CASE Phase_Crystallize OF

        0: // Start crystallization
            StartHeating(TargetTemp := CrystallizeTemp); // Use cooling
            Phase_Crystallize := 1;

        1: // Hold at low temp
            IF CurrentTemp <= CrystallizeTemp THEN
                CrystallizeTimer(IN := TRUE, PT := CrystallizeTime);
                Phase_Crystallize := 2;
            END_IF;

        2: // Wait for crystal formation
            IF CrystallizeTimer.Q THEN
                CrystallizeTimer(IN := FALSE);
                CrystallizeComplete := TRUE;
                Phase_Crystallize := 3;
            END_IF;

        3: // Done
            // Optional centrifuge call can be placed here
    END_CASE;
END_IF

// === DRYING PHASE ===
IF CrystallizeComplete THEN
    CASE Phase_Drying OF

        0: // Start dryer
            SetDryingHeater(Enable := TRUE);
            Phase_Drying := 1;

        1: // Wait until drying temp is reached
            IF CurrentTemp >= DryingTemp THEN
                DryingTimer(IN := TRUE, PT := DryingTime);
                Phase_Drying := 2;
            END_IF;

        2: // Maintain drying temperature
            IF DryingTimer.Q THEN
                DryingTimer(IN := FALSE);
                SetDryingHeater(Enable := FALSE);
                DryingComplete := TRUE;
                Phase_Drying := 3;
            END_IF;

        3: // Done
            // Batch complete flag can be set here
    END_CASE;
END_IF
