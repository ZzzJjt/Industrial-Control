PROGRAM SteelShutdown
VAR
    // Inputs
    FurnaceTemp1: REAL;      // Temperature sensor 1 (°C)
    FurnaceTemp2: REAL;      // Temperature sensor 2 (°C)
    GasFlowActual: REAL;     // Current gas flow (m³/h)
    OxygenFlowActual: REAL;  // Current oxygen flow (m³/h)
    FlameStatus: BOOL;       // Flame detector status
    FurnacePressure: REAL;   // Furnace pressure (mbar)
    CO_Level: REAL;          // CO concentration (ppm)

    // Outputs
    GasFlowSetpoint: REAL;   // Gas flow setpoint (m³/h)
    OxygenFlowSetpoint: REAL;// Oxygen flow setpoint (m³/h)
    BurnerValve: BOOL;       // Burner isolation valve
    InertGasValve: BOOL;     // Inert gas purge valve
    VentDamper: BOOL;        // Ventilation damper

    // Control Variables
    ShutdownStep: INT := 0;  // Current shutdown step
    TempSetpoint: REAL;      // Temperature setpoint (°C)
    ElapsedTime: TIME;       // Time since step start
    Timer1: TON;             // Timer for steps
    EmergencyHold: BOOL;     // Emergency hold flag

    // Constants
    MAX_TEMP: REAL := 850.0; // Max allowable temp (°C)
    MAX_PRESSURE: REAL := 0.5; // Max pressure (mbar)
    MIN_OXYGEN: REAL := 100.0; // Min oxygen flow (m³/h)
    MAX_OXYGEN: REAL := 1500.0; // Max oxygen flow (m³/h)
END_VAR

// Safety Interlocks
IF (FurnaceTemp1 > MAX_TEMP OR FurnaceTemp2 > MAX_TEMP) 
   OR (FurnacePressure > MAX_PRESSURE OR FurnacePressure < -MAX_PRESSURE)
   OR (CO_Level > 200.0) OR NOT FlameStatus THEN
    EmergencyHold := TRUE;
ELSE
    EmergencyHold := FALSE;
END_IF;

// Shutdown Sequence
CASE ShutdownStep OF
    0: // Idle
        IF StartShutdown THEN
            ShutdownStep := 1;
            Timer1(IN := TRUE, PT := T#6h);
        END_IF;

    1: // Cooldown to 800°C
        TempSetpoint := 1600.0 - (800.0 * TIME_TO_REAL(Timer1.ET) / 21600.0);
        IF (FurnaceTemp1 <= 800.0 AND FurnaceTemp2 <= 800.0) OR Timer1.Q THEN
            ShutdownStep := 2;
            Timer1(IN := FALSE);
            Timer1(IN := TRUE, PT := T#12h);
        END_IF;

    2: // Fuel Gas Reduction
        IF NOT EmergencyHold THEN
            GasFlowSetpoint := FuelGasRamp(
                StartFlow := 500.0, 
                EndFlow := 0.0, 
                Duration := T#12h, 
                CurrentTime := Timer1.ET
            );
        END_IF;
        OxygenFlowSetpoint := OxygenRegulator(
            GasFlow := GasFlowSetpoint, 
            TargetRatio := 2.5, 
            CurrentTemp := FurnaceTemp1
        );
        IF Timer1.Q THEN
            ShutdownStep := 3;
            Timer1(IN := FALSE);
        END_IF;

    3: // Burner Isolation
        IF GasFlowActual <= 0.0 AND FurnaceTemp1 <= 400.0 THEN
            BurnerValve := FALSE;
            InertGasValve := TRUE;
            Timer1(IN := TRUE, PT := T#30m);
            ShutdownStep := 4;
        END_IF;

    4: // Inert Gas Purge
        IF Timer1.Q THEN
            InertGasValve := FALSE;
            VentDamper := TRUE;
            ShutdownStep := 5;
        END_IF;

    5: // Final Cooldown
        IF FurnaceTemp1 <= 50.0 THEN
            VentDamper := FALSE;
            ShutdownStep := 6; // Complete
        END_IF;
END_CASE;

// Reset Timer on Step Change
IF ShutdownStep <> Timer1.IN THEN
    Timer1(IN := FALSE);
END_IF;
END_PROGRAM
