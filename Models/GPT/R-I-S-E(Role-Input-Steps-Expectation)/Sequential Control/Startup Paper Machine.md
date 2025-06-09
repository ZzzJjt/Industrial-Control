VAR
    Phase : INT := 0;

    // Inputs
    DriveReady : BOOL;
    TempSensor : REAL;
    PressureSensor : REAL;
    SpeedSensor : REAL;
    EmergencyStop : BOOL;

    // Outputs
    ConveyorStart : BOOL;
    RollStart : BOOL;
    HeaterON : BOOL;
    NipPressCtrl : REAL;
    SpeedCtrl : REAL;

    // Setpoints
    TargetTemp : REAL := 88.0;
    TargetPressure : REAL := 250.0; // kN/m
    TargetSpeed : REAL := 500.0;    // m/min

    // Timers
    tPreheat : TON;
    tNipRamp : TON;
    tSpeedRamp : TON;

    StartTime : TIME;

    // Internal Flags
    PreheatDone : BOOL := FALSE;
    NipRampDone : BOOL := FALSE;
    SpeedRampDone : BOOL := FALSE;
END_VAR
