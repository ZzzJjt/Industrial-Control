PROGRAM UreaReactionControl
VAR_INPUT
    AmmoniaValveOpen : BOOL; // Input state of ammonia valve
    CO2ValveOpen : BOOL;     // Input state of CO2 valve
    CurrentPressure : REAL;  // Measured pressure
    CurrentTemperature : REAL; // Measured temperature
END_VAR

VAR_OUTPUT
    ControlAmmoniaValve : BOOL; // Control signal for ammonia valve
    ControlCO2Valve : BOOL;     // Control signal for CO2 valve
END_VAR

VAR
    MaterialLoaded : BOOL := FALSE;
    ReactionStarted : BOOL := FALSE;
    ReactionComplete : BOOL := FALSE;
    ReactionTimer : TON;
    Step : INT := 0;
END_VAR

CONST
    PressureTarget : REAL := 150.0;
    TemperatureTarget : REAL := 180.0;
    PressureTolerance : REAL := 5.0;
    TemperatureTolerance : REAL := 5.0;
    ReactionDuration : TIME := T#30m;

// Initialize timers
ReactionTimer(IN := FALSE);

CASE Step OF
    0: // Idle
        ControlAmmoniaValve := FALSE;
        ControlCO2Valve := FALSE;
        MaterialLoaded := FALSE;
        ReactionStarted := FALSE;
        ReactionComplete := FALSE;
        ReactionTimer(IN := FALSE);
        Step := 1; // Move to Step 1: Load Materials

    1: // Load Materials
        ControlAmmoniaValve := TRUE;
        ControlCO2Valve := TRUE;

        IF AmmoniaValveOpen AND CO2ValveOpen THEN
            MaterialLoaded := TRUE;
            Step := 2; // Move to Step 2: Controlled Reaction
        END_IF;

    2: // Controlled Reaction
        IF MaterialLoaded THEN
            // Check pressure and temperature conditions
            IF ((CurrentPressure >= PressureTarget - PressureTolerance) AND (CurrentPressure <= PressureTarget + PressureTolerance)) AND
               ((CurrentTemperature >= TemperatureTarget - TemperatureTolerance) AND (CurrentTemperature <= TemperatureTarget + TemperatureTolerance)) THEN
                IF NOT ReactionStarted THEN
                    ReactionTimer(IN := TRUE);
                    ReactionTimer(PresetTime := ReactionDuration);
                    ReactionStarted := TRUE;
                END_IF;

                // Check if reaction timer expired
                IF ReactionTimer.Q THEN
                    ReactionComplete := TRUE;
                    Step := 0; // Return to Idle
                END_IF;
            ELSE
                // Handle out-of-tolerance conditions
                // Trigger alarm or take corrective action
                ReactionTimer(IN := FALSE);
                ReactionStarted := FALSE;
                Step := 0; // Return to Idle
            END_IF;
        END_IF;

    ELSE
        Step := 0; // Default to Idle
END_CASE;



