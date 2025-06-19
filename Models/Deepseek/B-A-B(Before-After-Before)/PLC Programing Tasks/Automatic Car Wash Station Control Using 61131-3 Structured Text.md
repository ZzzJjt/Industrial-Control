FUNCTION_BLOCK CarWashSafetyControl
{ SFC: Safe car wash control logic }
{ Author: Automation Engineer }
{ Date: 2025-05-13 }

(*
    Description:
    Implements safe control logic for automated car wash systems.
    Ensures the wash cycle runs only when it is safe to do so,
    based on presence of a vehicle and absence of a human.

    Safety Priority:
    Human detection takes precedence over all other conditions.
    Once triggered, the system locks out until manually reset.

    Inputs:
    - CarPresentSensor: TRUE if a vehicle is in the bay
    - HumanDetectedSensor: TRUE if a person is in the area

    Outputs:
    - WashActive: TRUE when the wash cycle is running
    - AlarmActive: TRUE when a human is detected
    - SafeToRun: Internal flag indicating system readiness
*)

VAR_INPUT
    CarPresentSensor : BOOL;     // Input from vehicle detection sensor
    HumanDetectedSensor : BOOL;  // Input from human presence detection sensor
END_VAR

VAR_OUTPUT
    WashActive : BOOL := FALSE;  // TRUE when wash cycle is active
    AlarmActive : BOOL := FALSE; // TRUE when alarm condition is active
    SafeToRun : BOOL := TRUE;    // State flag for safe operation
END_VAR

// Local variables for internal logic
VAR
    CarPresent : BOOL := FALSE;
    HumanDetected : BOOL := FALSE;
END_VAR

// Step 1: Read input sensors
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// Step 2: Evaluate safety and control states
IF HumanDetected THEN
    // Immediate stop and alarm if human is detected
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;

ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    // Start wash if conditions are safe
    WashActive := TRUE;
    AlarmActive := FALSE;

ELSE
    // Default: keep wash off unless conditions change
    WashActive := FALSE;
    AlarmActive := FALSE;
END_IF;

// Step 3: Reset safe-to-run flag only when area is clear and wash is not active
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;

END_FUNCTION_BLOCK


PROGRAM PLC_PRG
VAR
    CarPresentInput AT %IX100.0 : BOOL; // Simulated input bits
    HumanDetectedInput AT %IX100.1 : BOOL;

    CarWashCtrl: CarWashSafetyControl;

    // Output signals to actuators/alarm
    WashMotor AT %QX0.0 : BOOL;
    AlarmLight AT %QX0.1 : BOOL;
    SystemReady AT %QX0.2 : BOOL;
END_VAR

// Run the safety control logic
CarWashCtrl(
    CarPresentSensor := CarPresentInput,
    HumanDetectedSensor := HumanDetectedInput
);

// Map outputs
WashMotor := CarWashCtrl.WashActive;
AlarmLight := CarWashCtrl.AlarmActive;
SystemReady := CarWashCtrl.SafeToRun;
