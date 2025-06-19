VAR
    // Step control
    ReactionStep      : INT := 0;           // 0: Idle, 1: Heat, 2: Mix, 3: Hold, 4: Complete
    StepComplete      : BOOL := FALSE;

    // Process parameters
    TargetTemperature : REAL := 85.0;       // °C
    MixingSpeed       : INT := 120;         // RPM
    HoldTime          : TIME := T#15m;      // Reaction hold duration

    // Sensors and status
    CurrentTemperature: REAL;
    HeatingComplete   : BOOL := FALSE;
    MixingActive      : BOOL := FALSE;

    // Timer for hold phase
    HoldTimer         : TON;
    HoldStart         : BOOL := FALSE;

    // Method interfaces (assumed implemented elsewhere)
    // PROCEDURE StartHeating(TargetTemp: REAL);
    // PROCEDURE StartMixing(Speed: INT);
    // PROCEDURE StopMixing();
END_VAR

// === Step B.2: Reaction ===

CASE ReactionStep OF

    0: // Idle - waiting for trigger
        StepComplete := FALSE;
        // ReactionStep := 1; // Set externally when ready to start

    1: // Start heating to target temperature
        StartHeating(TargetTemp := TargetTemperature);
        IF CurrentTemperature >= TargetTemperature THEN
            HeatingComplete := TRUE;
            ReactionStep := 2;
        END_IF;

    2: // Begin mixing at specified speed
        StartMixing(Speed := MixingSpeed);
        MixingActive := TRUE;
        ReactionStep := 3;

    3: // Hold time under controlled conditions
        IF NOT HoldStart THEN
            HoldTimer(IN := TRUE, PT := HoldTime);
            HoldStart := TRUE;
        END_IF;

        HoldTimer(IN := TRUE); // Continue timing
        IF HoldTimer.Q THEN
            StopMixing();
            MixingActive := FALSE;
            ReactionStep := 4;
        END_IF;

    4: // Complete
        HoldTimer(IN := FALSE);
        HoldStart := FALSE;
        StepComplete := TRUE;
        // ReactionStep := 0; // Reset externally if needed

END_CASE;
