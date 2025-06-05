VAR
    PID_CTRL : PID;
    SP       : REAL := 3.0;           // Desired ppm
    PV       : REAL;                  // Sensor: chlorine ppm
    MV       : REAL;                  // Output to dosing pump
END_VAR

// Update PV from sensor
PV := MeasuredChlorine();

// Call PID function block
PID_CTRL(
    PV := PV,
    SP := SP,
    Kp := 1.2,
    Ki := 0.5,
    Kd := 0.0,
    MAN := FALSE,
    MV_MAN := 0.0,
    MV_MIN := 0.0,
    MV_MAX := 100.0,
    ENABLE := TRUE,
    DT := 0.1,
    MV => MV
);

// Use MV to set pump speed
SetPumpSpeed(MV);
