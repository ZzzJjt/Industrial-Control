// IEC 61131-3 Structured Text: Steel Facility Shutdown Control (Steps 4–6)

VAR
    CurrentTemp         : REAL;    // Current furnace temperature (°C)
    TargetTemp          : REAL := 400.0;  // Safe shutdown temperature
    FuelGasFlow         : REAL;    // Current gas flow rate (Nm^3/h)
    TargetFuelFlow      : REAL := 0.0;
    MaxFuelFlow         : REAL := 100.0;  // Initial maximum fuel gas flow
    OxygenFlow          : REAL;    // Current oxygen supply
    FuelToAirRatio      : REAL := 2.5; // 1:2.5 target ratio (Fuel:Air)

    RampStartTime       : TIME;
    RampDuration        : TIME := T#12h0m0s;
    RampActive          : BOOL := FALSE;
    ShutdownStep        : INT := 4;

    Timer               : TON;
    CurrentTime         : TIME;
    ElapsedTime         : TIME;

    Fault               : BOOL;
END_VAR

// Step 4: Begin temperature reduction and start fuel ramp-down
IF ShutdownStep = 4 THEN
    IF NOT RampActive THEN
        RampStartTime := CURRENT_TIME;
        RampActive := TRUE;
    END_IF

    // Calculate elapsed time since ramp started
    ElapsedTime := CURRENT_TIME - RampStartTime;
    IF ElapsedTime < RampDuration THEN
        FuelGasFlow := MaxFuelFlow * (1.0 - (REAL_TO_TIME(ElapsedTime) / REAL_TO_TIME(RampDuration)));
    ELSE
        FuelGasFlow := TargetFuelFlow;
    END_IF

    // Check if temperature < 400°C to proceed
    IF CurrentTemp < TargetTemp THEN
        ShutdownStep := 5;
    END_IF
END_IF

// Step 5: Adjust oxygen for safe combustion ratio
IF ShutdownStep = 5 THEN
    OxygenFlow := FuelGasFlow * FuelToAirRatio;

    // Check for safe oxygen range (arbitrary bounds 50–300 Nm^3/h)
    IF (OxygenFlow < 50.0) OR (OxygenFlow > 300.0) THEN
        Fault := TRUE; // Trigger interlock
    END_IF

    // Proceed when fuel is minimal and oxygen is balanced
    IF FuelGasFlow <= 1.0 AND NOT Fault THEN
        ShutdownStep := 6;
    END_IF
END_IF

// Step 6: Confirm shutdown complete
IF ShutdownStep = 6 THEN
    FuelGasFlow := 0.0;
    OxygenFlow := 0.0;
    // Optional: Set facility status flag or log completion
END_IF
