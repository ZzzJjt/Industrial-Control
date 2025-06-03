PROGRAM PressSectionStartup
VAR
    phase                : INT := 0;
    timer                : TON;
    startupComplete      : BOOL := FALSE;

    rollSpeedSetpoint    : REAL := 0.0;        // m/min
    nipPressureSetpoint  : REAL := 0.0;        // kN/m
    temperatureSetpoint  : REAL := 0.0;        // °C

    allDrivesReady       : BOOL;
    guardsClosed         : BOOL;
    pressFeltOK          : BOOL;
    tempFeedback         : REAL;
    pressureFeedback     : REAL;
    speedFeedback        : REAL;
END_VAR

// Safety check function
FUNCTION CheckPreconditions : BOOL
VAR
    result : BOOL;
END_VAR
result := allDrivesReady AND guardsClosed AND pressFeltOK;
RETURN result;
END_FUNCTION

// Main startup sequence
CASE phase OF
    0: // Pre-checks
        IF CheckPreconditions() THEN
            phase := 1;
            timer(IN := FALSE);
        END_IF;

    1: // Roll Activation
        (* Activate individual press rolls and conveyors *)
        // Simulated output actions can be set here
        timer(IN := TRUE, PT := T#5s);
        IF timer.Q THEN
            phase := 2;
            timer(IN := FALSE);
        END_IF;

    2: // Pressure Build-Up
        nipPressureSetpoint := 250.0;  // kN/m
        // Pressure control logic (could be a PID block here)
        IF ABS(pressureFeedback - nipPressureSetpoint) < 5.0 THEN
            phase := 3;
        END_IF;

    3: // Temperature Stabilization
        temperatureSetpoint := 90.0;  // °C
        // Temperature control logic
        IF ABS(tempFeedback - temperatureSetpoint) < 2.0 THEN
            phase := 4;
        END_IF;

    4: // Speed Ramp-Up
        IF rollSpeedSetpoint < 100.0 THEN
            rollSpeedSetpoint := rollSpeedSetpoint + 5.0;
        ELSE
            phase := 5;
        END_IF;

    5: // Final checks and transition to normal operation
        IF ABS(speedFeedback - 100.0) < 1.0 THEN
            startupComplete := TRUE;
        END_IF;
END_CASE;
