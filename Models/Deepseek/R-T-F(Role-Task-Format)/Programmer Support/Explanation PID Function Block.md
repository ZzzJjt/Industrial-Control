IF OUT > LIMIT_HI THEN
    OUT := LIMIT_HI;
    LIMIT_ACTIVE := TRUE;
ELSIF OUT < LIMIT_LO THEN
    OUT := LIMIT_LO;
    LIMIT_ACTIVE := TRUE;
ELSE
    LIMIT_ACTIVE := FALSE;
END_IF;

IF MANUAL THEN
    OUT := Manual_Output; // Alternative control signal
    MANUAL_MODE := TRUE;
ELSE
    MANUAL_MODE := FALSE;
END_IF;

PROGRAM TankLevelControl
VAR
    // Inputs
    SP : REAL := 80.0; // Desired level (cm)
    PV : REAL;        // Real-time level measurement
    Kp : REAL := 1.0; // Proportional gain
    Ki : REAL := 0.2; // Integral gain
    Kd : REAL := 0.05; // Derivative gain
    MANUAL : BOOL := FALSE; // Manual override flag
    LIMIT_HI : REAL := 100.0; // Upper limit for output
    LIMIT_LO : REAL := 0.0; // Lower limit for output
    Manual_Output : REAL := 50.0; // Alternative control signal in manual mode

    // Outputs
    OUT : REAL; // Control signal sent to pump speed
    ERROR : REAL; // Live difference between SP and PV
    LIMIT_ACTIVE : BOOL; // Flag indicating output was clamped
    MANUAL_MODE : BOOL; // Flag indicating manual mode is active

    // PID function block instance
    FB_PID : FB_PID;
END_VAR

// Read real-time level measurement from sensor
PV := ReadLevelSensor();

// Call the PID function block
FB_PID(
    SP := SP,
    PV := PV,
    Kp := Kp,
    Ki := Ki,
    Kd := Kd,
    MANUAL := MANUAL,
    LIMIT_HI := LIMIT_HI,
    LIMIT_LO := LIMIT_LO,
    Manual_Output := Manual_Output,
    OUT => OUT,
    ERROR => ERROR,
    LIMIT_ACTIVE => LIMIT_ACTIVE,
    MANUAL_MODE => MANUAL_MODE
);

// Send control signal to pump speed
SetPumpSpeed(OUT);
