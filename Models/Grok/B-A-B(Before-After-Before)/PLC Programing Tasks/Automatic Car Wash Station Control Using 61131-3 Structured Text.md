(* Car Wash Safety Control Logic in IEC 61131-3 Structured Text *)
(* Purpose: Ensure safe operation by monitoring car and human presence, controlling wash cycle, and managing alarms *)

PROGRAM CarWashSafety
VAR
    (* Input Sensors *)
    CarPresentSensor : BOOL;      (* TRUE when a car is detected in the wash bay *)
    HumanDetectedSensor : BOOL;   (* TRUE when a human is detected in the wash zone *)

    (* Control States *)
    CarPresent : BOOL;            (* Latched state of car presence *)
    HumanDetected : BOOL;         (* Latched state of human presence *)
    WashActive : BOOL;            (* TRUE when wash cycle is running *)
    AlarmActive : BOOL;           (* TRUE when safety alarm is triggered *)
    SafeToRun : BOOL := TRUE;     (* Safety interlock; FALSE after human detection until reset *)
    ResetConfirm : BOOL;          (* Operator confirmation to reset SafeToRun after interruption *)

    (* Outputs *)
    WashMotor : BOOL;             (* Controls wash system actuator *)
    AlarmSiren : BOOL;            (* Controls audible/visual alarm *)
END_VAR

(* Main Safety Logic *)
(* 1. Latch sensor inputs for processing *)
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

(* 2. Safety Check: Human detection takes absolute priority *)
(* If a human is detected, immediately halt wash and trigger alarm *)
IF HumanDetected THEN
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;  (* Lock out system until reset *)
END_IF;

(* 3. Wash Activation Logic *)
(* Wash only starts if: car is present, no human detected, and system is safe *)
IF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;
    AlarmActive := FALSE;
END_IF;

(* 4. Safe State Reset Logic *)
(* System can only return to SafeToRun after human is gone, wash is stopped, and operator confirms *)
IF NOT HumanDetected AND NOT WashActive AND ResetConfirm THEN
    SafeToRun := TRUE;
    ResetConfirm := FALSE;  (* Reset confirmation flag after use *)
END_IF;

(* 5. Output Assignments *)
(* Map internal states to physical outputs *)
WashMotor := WashActive;    (* Actuate wash system only when WashActive is TRUE *)
AlarmSiren := AlarmActive;  (* Activate alarm when AlarmActive is TRUE *)

(* Notes:
   - HumanDetected takes precedence to ensure safety over all other conditions
   - SafeToRun prevents automatic restart after human interruption, requiring manual confirmation
   - WashMotor and AlarmSiren are directly tied to actuators (e.g., relays for motor control, siren)
   - ResetConfirm should be wired to an operator interface (e.g., HMI button or physical switch)
   - For physical integration:
     - CarPresentSensor: Typically a photoelectric or proximity sensor at bay entrance
     - HumanDetectedSensor: Motion detector or safety mat in wash zone
     - WashMotor: Relay output to control wash system power
     - AlarmSiren: Relay output to audible/visual alarm system
*)
END_PROGRAM
