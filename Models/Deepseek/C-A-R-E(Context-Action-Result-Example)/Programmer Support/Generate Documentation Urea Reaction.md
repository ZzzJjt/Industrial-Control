PROGRAM UreaReactionControl
VAR_INPUT
    stAmmoniaValve : BOOL;        // Status of the ammonia valve (read-only)
    stCO2Valve : BOOL;            // Status of the CO₂ valve (read-only)
    rCurrentPressure : REAL;      // Reactor pressure in bar
    rCurrentTemperature : REAL;   // Reactor temperature in °C
END_VAR

VAR_OUTPUT
    stAmmoniaValveControl : BOOL; // TRUE = open ammonia valve
    stCO2ValveControl : BOOL;     // TRUE = open CO₂ valve
    stReactionFinished : BOOL;    // TRUE = reaction process completed
END_VAR

VAR
    bMaterialLoaded : BOOL := FALSE;
    bConditionsMet : BOOL := FALSE;
    tStartTime : TIME_OF_DAY;
    reactionTimer : TON;
    pressureTolerance : REAL := 5.0;
    temperatureTolerance : REAL := 2.0;
    targetPressure : REAL := 175.0;
    targetTemperature : REAL := 185.0;
    reactionTime : TIME := T#30m;
END_VAR

// Initialize timers
reactionTimer(IN := FALSE);

CASE
    // Material Loading (Step 1)
    NOT bMaterialLoaded:
        // Open valves
        stAmmoniaValveControl := TRUE;
        stCO2ValveControl := TRUE;

        // Wait for confirmation that both valves are open
        IF stAmmoniaValve AND stCO2Valve THEN
            bMaterialLoaded := TRUE;
            // Begin timing the reaction
            tStartTime := CURRENT_TIME;
            reactionTimer(IN := TRUE);
        END_IF;

    // Reaction Control (Step 2)
    bMaterialLoaded AND NOT reactionTimer.Q:
        // Monitor and maintain reaction conditions
        bConditionsMet := (
            (rCurrentPressure >= targetPressure - pressureTolerance) AND
            (rCurrentPressure <= targetPressure + pressureTolerance) AND
            (rCurrentTemperature >= targetTemperature - temperatureTolerance) AND
            (rCurrentTemperature <= targetTemperature + temperatureTolerance)
        );

        // Continue until the timer reaches the reaction duration
        IF NOT bConditionsMet THEN
            // Handle condition failure (optional)
        END_IF;

    // Shutdown (Completion)
    ELSE:
        // Close all valves
        stAmmoniaValveControl := FALSE;
        stCO2ValveControl := FALSE;

        // Mark the process as finished
        stReactionFinished := TRUE;

        // Reset timers and flags
        reactionTimer(IN := FALSE);
        bMaterialLoaded := FALSE;
        bConditionsMet := FALSE;
END_CASE;



