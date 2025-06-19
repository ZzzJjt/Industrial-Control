FUNCTION_BLOCK FB_GasTurbine_Interlocks
VAR_INPUT
    ExhaustGasTemp      : REAL;     // °C
    RotorSpeed          : REAL;     // % of nominal
    CombustionPressure  : REAL;     // bar
    OilPressure         : REAL;     // bar
    VibrationLevel      : REAL;     // mm/s
    FlameDetected       : BOOL;     // TRUE = flame present
    FuelGasPressure     : REAL;     // bar
    WaterFlowRate       : REAL;     // L/min
    CompressorSurge     : BOOL;     // TRUE = surge detected
    EmergencyStop       : BOOL;     // Manual E-Stop button
END_VAR

VAR_OUTPUT
    ShutdownTurbine     : BOOL;
    OpenReliefValve     : BOOL;
    CloseFuelValve      : BOOL;
    StopRecirculation   : BOOL;
    AlarmFlameFailure   : BOOL;
    OpenBypassValve     : BOOL;
END_VAR

VAR
    OverTemp            : BOOL;
    OverSpeed           : BOOL;
    OverPressure        : BOOL;
    LowOilPressure      : BOOL;
    HighVibration       : BOOL;
    FlameFailure        : BOOL;
    LowFuelPressure     : BOOL;
    LowWaterFlow        : BOOL;
END_VAR

// --- Interlock Logic ---

OverTemp         := ExhaustGasTemp > 650.0;
OverSpeed        := RotorSpeed > 105.0;
OverPressure     := CombustionPressure > 30.0;
LowOilPressure   := OilPressure < 1.5;
HighVibration    := VibrationLevel > 10.0;
FlameFailure     := NOT FlameDetected;
LowFuelPressure  := FuelGasPressure < 2.0;
LowWaterFlow     := WaterFlowRate < 200.0;

// --- Safety Actions ---

ShutdownTurbine := OverTemp OR OverSpeed OR LowOilPressure OR
                   HighVibration OR FlameFailure OR LowFuelPressure OR
                   LowWaterFlow OR CompressorSurge OR EmergencyStop;

OpenReliefValve := OverPressure;

CloseFuelValve  := FlameFailure OR LowFuelPressure OR EmergencyStop;

StopRecirculation := ShutdownTurbine;

AlarmFlameFailure := FlameFailure;

OpenBypassValve := CompressorSurge;
