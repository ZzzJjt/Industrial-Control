VAR
    SP           : REAL := 80.0;             // Desired tank level (cm)
    PV           : REAL;                     // Actual tank level from sensor
    OUT          : REAL;                     // Pump speed (0–100%)
    Kp           : REAL := 1.0;
    Ki           : REAL := 0.2;
    Kd           : REAL := 0.05;
    MANUAL       : BOOL := FALSE;
    LIMIT_HI     : REAL := 100.0;
    LIMIT_LO     : REAL := 0.0;
END_VAR

// Read sensor
PV := ReadLevelSensor();

// Call PID block
PID(SP := SP, PV := PV, Kp := Kp, Ki := Ki, Kd := Kd,
    MANUAL := MANUAL, LIMIT_HI := LIMIT_HI, LIMIT_LO := LIMIT_LO,
    OUT => OUT);
    
// Apply control signal
SetPumpSpeed(OUT);
