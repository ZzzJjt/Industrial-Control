VAR
    // Process state
    state : INT := 0;

    // Timer
    stepTimer : TON;
    timerEnable : BOOL := FALSE;

    // Temperature and pressure targets for each step
    rawMatPrepTemp     : REAL := 90.0;
    rawMatPrepPressure : REAL := 5.0;

    reactorTemp        : REAL := 160.0;
    reactorPressure    : REAL := 15.0;

    coolingTemp        : REAL := 60.0;
    coolingPressure    : REAL := 2.0;

    // Current measured values
    currentTemp    : REAL;
    currentPressure: REAL;

    // Output controls
    Heater         : BOOL;
    Cooler         : BOOL;
    PressureValve  : BOOL;

    // Timer preset
    stepDuration : TIME := T#5s;
END_VAR

// === Helper Methods ===

METHOD UpdateTemperaturesAndPressures : VOID
    // Control heating, cooling, and valve based on target values
    IF currentTemp < rawMatPrepTemp THEN
        Heater := TRUE;
        Cooler := FALSE;
    ELSIF currentTemp > rawMatPrepTemp THEN
        Heater := FALSE;
        Cooler := TRUE;
    ELSE
        Heater := FALSE;
        Cooler := FALSE;
    END_IF;

    IF currentPressure < rawMatPrepPressure THEN
        PressureValve := TRUE;
    ELSE
        PressureValve := FALSE;
    END_IF;
END_METHOD


METHOD SetTargets : VOID
VAR_INPUT
    targetTemp : REAL;
    targetPress : REAL;
END_VAR
    rawMatPrepTemp := targetTemp;
    rawMatPrepPressure := targetPress;
END_METHOD


// === Timer Logic ===
stepTimer(IN := timerEnable, PT := stepDuration);

// === Main State Machine ===

CASE state OF

0: // Initialization
    Heater := FALSE;
    Cooler := FALSE;
    PressureValve := FALSE;
    timerEnable := FALSE;
    state := 1;

1: // Raw Material Preparation
    SetTargets(targetTemp := 90.0, targetPress := 5.0);
    UpdateTemperaturesAndPressures();
    timerEnable := TRUE;
    IF stepTimer.Q THEN
        timerEnable := FALSE;
        state := 2;
    END_IF;

2: // Reactor Polymerization
    SetTargets(targetTemp := 160.0, targetPress := 15.0);
    UpdateTemperaturesAndPressures();
    timerEnable := TRUE;
    IF stepTimer.Q THEN
        timerEnable := FALSE;
        state := 3;
    END_IF;

3: // Cooling
    SetTargets(targetTemp := 60.0, targetPress := 2.0);
    UpdateTemperaturesAndPressures();
    timerEnable := TRUE;
    IF stepTimer.Q THEN
        timerEnable := FALSE;
        state := 4;
    END_IF;

4: // End of Batch
    Heater := FALSE;
    Cooler := FALSE;
    PressureValve := FALSE;
    // Batch complete, optionally reset or go to idle
    state := 0;

ELSE
    // Fallback state
    state := 0;
END_CASE;
