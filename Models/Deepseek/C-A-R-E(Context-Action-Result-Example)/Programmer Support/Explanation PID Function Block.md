PROGRAM WaterTreatmentPlantChlorineControl
VAR_INPUT
    MeasuredChlorine : REAL; // Measured chlorine concentration (ppm)
END_VAR

VAR_OUTPUT
    DosingRate : REAL;        // Dosing rate output to the pump (% open)
END_VAR

VAR
    Chlorine_PID : FB_PID;    // Instance of the PID function block
    SP_Chlorine : REAL := 3.0; // Desired chlorine concentration (ppm)
    Kp : REAL := 1.2;
    Ki : REAL := 0.5;
    Kd : REAL := 0.0;
    OUT_MIN : REAL := 0.0;
    OUT_MAX : REAL := 100.0;
    MANUAL : BOOL := FALSE;
    AUTO : BOOL := TRUE;
END_VAR

// Configure and execute the PID function block
Chlorine_PID(
    PV := MeasuredChlorine,
    SP := SP_Chlorine,
    Kp := Kp,
    Ki := Ki,
    Kd := Kd,
    MAN := MANUAL,
    AUT := AUTO,
    OUT_MIN := OUT_MIN,
    OUT_MAX := OUT_MAX,
    OUT => DosingRate,
    ERR => ERROR,
    SAT => SATURATED,
    LIM => LIMIT_REACHED
);

// Output the control signal to the dosing pump
DosingRate := Chlorine_PID.OUT;

// Optional: Monitor error and saturation flags
IF Chlorine_PID.SATURATED THEN
    // Handle saturation case
END_IF;

IF Chlorine_PID.LIMIT_REACHED THEN
    // Handle limit reached case
END_IF;

// Inline comments explaining the logic:
// The program uses the OSCAT PID function block to control the chlorine dosing pump in a water treatment plant.
// It maintains the desired chlorine concentration (SP_Chlorine) by adjusting the dosing rate (DosingRate).
// The PID parameters (Kp, Ki, Kd) are configured to achieve stable and responsive control.
// Output clamping ensures the dosing rate remains within safe limits (0.0–100.0%).
// Manual override and auto mode handling provide flexibility in operation.



