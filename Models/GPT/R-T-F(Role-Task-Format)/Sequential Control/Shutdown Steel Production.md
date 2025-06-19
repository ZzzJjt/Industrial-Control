PROGRAM FurnaceShutdown

VAR
    // Phase control
    ShutdownStep : INT := 1;

    // Process variables
    FurnaceTemp : REAL;
    GasFlowRate : REAL;
    OxygenFlowRate : REAL;

    // Target parameters
    TargetTemp : REAL := 150.0; // °C for cooldown
    TargetGasFlow : REAL := 0.0; // Fully off
    TargetO2Ratio : REAL := 2.5; // Fuel:O2 = 1:2.5

    // Timing
    RampTimer : TON;
    StepTimer : TON;
    StepDuration : TIME := T#30m;

    // Function block outputs
    NewGasFlow : REAL;
    NewO2Flow : REAL;
END_VAR

// --- Read real-time process variables (to be linked to sensors) ---
// FurnaceTemp := ReadFurnaceTemp();
// GasFlowRate := ReadGasFlowRate();

CASE ShutdownStep OF

    1: // Step 1 – Begin cooldown
        StartCooling(); // User-defined function
        StepTimer(IN := TRUE, PT := StepDuration);
        IF StepTimer.Q THEN
            ShutdownStep := 2;
            StepTimer(IN := FALSE);
        END_IF;

    2: // Step 2 – Ramp down gas over 12 hours
        RampTimer(IN := TRUE, PT := T#12h);
        NewGasFlow := RampDownGas(InitialFlow := GasFlowRate, Timer := RampTimer);
        SetGasFlow(NewGasFlow);
        IF RampTimer.Q THEN
            ShutdownStep := 3;
            RampTimer(IN := FALSE);
        END_IF;

    3: // Step 3 – Adjust oxygen for safe ratio
        NewO2Flow := AdjustOxygen(GasFlow := NewGasFlow, Temp := FurnaceTemp);
        SetOxygenFlow(NewO2Flow);
        IF FurnaceTemp <= TargetTemp THEN
            ShutdownStep := 4;
        END_IF;

    4: // Step 4 – Maintain safety idle for 30 min
        MaintainIdle(); // hold valves, monitor temps
        StepTimer(IN := TRUE, PT := StepDuration);
        IF StepTimer.Q THEN
            ShutdownStep := 5;
            StepTimer(IN := FALSE);
        END_IF;

    5: // Step 5 – Final isolation
        CloseAllValves();
        IsolateFurnace();
        ShutdownStep := 6;

    6: // Step 6 – Shutdown complete
        SetSystemState(SHUTDOWN_COMPLETE);
END_CASE

FUNCTION AdjustOxygen : REAL
VAR_INPUT
    GasFlow : REAL;
    Temp : REAL; // Optional: can include temp compensation if needed
END_VAR
VAR
    O2Ratio : REAL := 2.5;
BEGIN
    AdjustOxygen := GasFlow * O2Ratio;
END_FUNCTION
