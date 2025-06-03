PROGRAM AdhesiveReactionControl
VAR
    // State machine variable
    reactionState : INT := 0;

    // Process parameters (could be loaded from recipe management system)
    targetTemp : REAL := 85.0; // °C
    reactionTime : TIME := T#30m;

    // Inputs
    currentTemp : REAL;
    tempSensorOK : BOOL;

    // Outputs
    HeaterOn : BOOL := FALSE;
    MixerOn  : BOOL := FALSE;
    ReactionComplete : BOOL := FALSE;

    // Timer
    tReaction : TON;

END_VAR

// Main control logic
CASE reactionState OF

    0: // Idle / Waiting to start
        IF tempSensorOK THEN
            reactionState := 1;
        END_IF;

    1: // Start heating
        HeaterOn := TRUE;
        IF currentTemp >= targetTemp THEN
            HeaterOn := FALSE;
            reactionState := 2;
        END_IF;

    2: // Start mixing
        MixerOn := TRUE;
        tReaction(IN := TRUE, PT := reactionTime);
        reactionState := 3;

    3: // Hold reaction
        IF tReaction.Q THEN
            reactionState := 4;
        END_IF;

    4: // End reaction
        MixerOn := FALSE;
        ReactionComplete := TRUE;
        reactionState := 5;

    5: // Final state – maintain end condition
        // Optionally wait for external reset or next batch trigger
END_CASE;
