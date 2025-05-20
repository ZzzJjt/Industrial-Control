FUNCTION_BLOCK CarWashController
VAR_INPUT
    CarPresentSensor : BOOL; (* TRUE if car is detected in wash bay *)
    HumanDetectedSensor : BOOL; (* TRUE if human is detected in danger zone *)
    EmergencyStop : BOOL; (* TRUE if emergency stop is pressed *)
END_VAR
VAR_OUTPUT
    WashActive : BOOL; (* TRUE when wash cycle is running *)
    AlarmActive : BOOL; (* TRUE when human is detected or emergency stop is active *)
    SafeToRun : BOOL; (* TRUE when system is safe to start a new wash cycle *)
END_VAR
VAR
    PrevHumanDetected : BOOL; (* Tracks previous state of HumanDetectedSensor *)
END_VAR

(* Initialize outputs *)
WashActive := FALSE;
AlarmActive := FALSE;
SafeToRun := TRUE; (* Initially safe until conditions change *)

(* Store previous human detection state for edge detection *)
PrevHumanDetected := HumanDetectedSensor;

(* Safety interlock: Check for human presence or emergency stop *)
IF HumanDetectedSensor OR EmergencyStop THEN
    (* Human detected or emergency stop pressed: halt wash and trigger alarm *)
    WashActive := FALSE; (* Immediately stop wash cycle *)
    AlarmActive := TRUE; (* Activate alarm to alert operators *)
    SafeToRun := FALSE; (* Lock out new wash cycles until cleared *)
ELSE
    (* No human detected and no emergency stop *)
    AlarmActive := FALSE; (* Clear alarm *)
    
    (* Check if safe to start or continue wash *)
    IF CarPresentSensor AND SafeToRun THEN
        (* Car is present and system is safe: activate wash *)
        WashActive := TRUE;
    ELSE
        (* No car or not safe: ensure wash is off *)
        WashActive := FALSE;
    END_IF;
END_IF;

(* Reset SafeToRun flag *)
IF NOT HumanDetectedSensor AND NOT PrevHumanDetected AND 
   NOT EmergencyStop AND NOT WashActive THEN
    (* Area is clear, no recent human detection, no emergency stop, and wash is off *)
    SafeToRun := TRUE; (* Allow new wash cycles *)
END_IF;

(* Update previous human detection state *)
PrevHumanDetected := HumanDetectedSensor;

END_FUNCTION_BLOCK
