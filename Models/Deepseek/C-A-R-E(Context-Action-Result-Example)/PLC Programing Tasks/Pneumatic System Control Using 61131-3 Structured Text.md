PROGRAM PneumaticControlSystem
VAR_INPUT
    FlowInput : REAL; // Flow rate in SLPM from flow sensor
    PressureInput : REAL; // Pressure in bar from pressure sensor
END_VAR

VAR_OUTPUT
    FlowValveOutput : BOOL := FALSE; // Output to control flow valve actuator
    PressureReliefValve : BOOL := FALSE; // Output to control pressure relief valve
    FlowError : BOOL := FALSE; // Error flag for flow deviation
    PressureError : BOOL := FALSE; // Error flag for pressure fault
END_VAR

VAR
    FlowSetpoint : REAL := 50.0; // Target flow rate in SLPM
    MinPressure : REAL := 5.5; // Minimum acceptable pressure in bar
    MaxPressure : REAL := 6.0; // Maximum acceptable pressure in bar
    LastFlowInput : REAL; // Previous flow input value
    LastPressureInput : REAL; // Previous pressure input value
END_VAR

// Flow Control Logic
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE; // Open valve to increase flow
ELSIF FlowInput > FlowSetpoint THEN
    FlowValveOutput := FALSE; // Close valve to decrease flow
ELSE
    FlowValveOutput := FALSE; // Maintain current flow
END_IF;

// Pressure Monitoring and Protection
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE; // Activate safety relief valve
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE; // Deactivate safety relief valve
END_IF;

// Flow Deviation Check
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF;

// Update previous input values
LastFlowInput := FlowInput;
LastPressureInput := PressureInput;

// Inline comments explaining the logic:
// The program regulates airflow to a setpoint of 50 SLPM and monitors pressure to stay between 5.5 and 6.0 bar.
// It includes safety mechanisms to detect and respond to over/under pressure and abnormal flow deviations.
// Outputs control the flow valve actuator and pressure relief valve, and error flags indicate fault conditions.



