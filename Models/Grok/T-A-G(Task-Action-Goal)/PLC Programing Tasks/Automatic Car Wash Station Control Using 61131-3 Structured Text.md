(* Program: Single-Bay Automatic Car Wash Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages car wash operation with safety interlocks for human and car presence *)
PROGRAM PRG_CarWashControl
VAR
    (* Inputs *)
    CarPresentSensor : BOOL;          (* TRUE if car is detected in wash bay *)
    HumanDetectedSensor : BOOL;       (* TRUE if person is detected in wash zone *)
    Reset : BOOL;                     (* Manual reset input for alarm and safe state *)
    
    (* Outputs *)
    WashActive : BOOL;                (* TRUE to start wash process *)
    AlarmActive : BOOL;               (* TRUE if human detected during wash *)
    SafeToRun : BOOL;                 (* TRUE if wash can safely begin *)
    
    (* Internal Variables *)
    LastReset : BOOL;                 (* For rising edge detection on Reset *)
    HumanInterruption : BOOL;         (* Latch human detection to prevent auto-restart *)
END_VAR

(* Initialize outputs *)
WashActive := FALSE;
AlarmActive := FALSE;
SafeToRun := TRUE;                    (* Initial state: Safe to run until human detected *)

(* Main logic *)
(* Interlock: Human presence takes precedence, stopping wash and requiring manual reset *)
IF HumanDetectedSensor THEN
    (* Human detected: Stop wash, activate alarm, lock out operation *)
    WashActive := FALSE;              (* Immediately halt wash process *)
    AlarmActive := TRUE;              (* Activate alarm to alert operators *)
    SafeToRun := FALSE;               (* Prevent wash until reset *)
    HumanInterruption := TRUE;        (* Latch interruption to enforce manual reset *)
ELSE
    (* No human detected: Evaluate wash conditions *)
    IF CarPresentSensor AND SafeToRun THEN
        (* Car present, safe to run: Start wash *)
        WashActive := TRUE;           (* Activate wash process *)
        AlarmActive := FALSE;         (* Clear any previous alarm *)
    ELSIF NOT CarPresentSensor THEN
        (* No car present: Stop wash *)
        WashActive := FALSE;          (* Deactivate wash process *)
        (* Maintain AlarmActive if human interruption latched *)
        IF NOT HumanInterruption THEN
            AlarmActive := FALSE;     (* Clear alarm if no interruption *)
        END_IF;
    END_IF;
    
    (* Safe state restoration: Only reset SafeToRun when no human and wash stopped *)
    (* Prevents automatic restart after human interruption *)
    IF NOT HumanDetectedSensor AND NOT WashActive AND NOT HumanInterruption THEN
        SafeToRun := TRUE;            (* Restore safe state for next cycle *)
    END_IF;
END_IF;

(* Manual reset logic *)
(* Allows clearing HumanInterruption and AlarmActive after operator verification *)
IF Reset AND NOT LastReset AND NOT HumanDetectedSensor THEN
    (* Rising edge on Reset, no human present: Clear latched state *)
    HumanInterruption := FALSE;       (* Allow future wash cycles *)
    AlarmActive := FALSE;             (* Reset alarm *)
    IF NOT WashActive THEN
        SafeToRun := TRUE;            (* Restore safe state if wash stopped *)
    END_IF;
END_IF;
LastReset := Reset;

(* Ensure safe state on power-up or PLC stop *)
(* WashActive and SafeToRun remain FALSE until conditions are met *)
IF NOT CarPresentSensor THEN
    WashActive := FALSE;              (* Ensure wash is off if no car *)
END_IF;

END_PROGRAM
