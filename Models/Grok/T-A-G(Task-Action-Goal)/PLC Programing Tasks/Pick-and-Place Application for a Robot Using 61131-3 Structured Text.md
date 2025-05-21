(* Program: Pick-and-Place Robot Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Controls robot between two conveyors in Manual and Auto Modes *)
(* Ensures safe mode interlocking and timed Auto sequence *)
PROGRAM PRG_PickAndPlaceRobot
VAR
    (* Inputs *)
    BtnManual : BOOL;                 (* TRUE to select Manual Mode *)
    BtnAuto : BOOL;                   (* TRUE to select Auto Mode *)
    CmdClip : BOOL;                   (* Manual command: Clip product from Conveyor A *)
    CmdTransfer : BOOL;               (* Manual command: Transfer product to Conveyor B *)
    CmdRelease : BOOL;                (* Manual command: Release product onto Conveyor B *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Clip : BOOL;                      (* TRUE to activate clip action *)
    Transfer : BOOL;                  (* TRUE to activate transfer action *)
    Release : BOOL;                   (* TRUE to activate release action *)
    ManualMode : BOOL;                (* TRUE if Manual Mode active *)
    AutoMode : BOOL;                  (* TRUE if Auto Mode active *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or mode error *)
    
    (* Internal Variables *)
    State : UINT;                     (* Auto Mode state: 0=Idle, 1=Clipping, 2=Waiting, 3=Releasing *)
    AutoTrigger : BOOL;               (* TRUE to start Auto sequence *)
    LastBtnAuto : BOOL;               (* For rising edge detection on BtnAuto *)
    AutoTimer : TON;                  (* 2-second timer for Auto sequence delay *)
    ModeError : BOOL;                 (* TRUE if invalid mode selection *)
END_VAR

(* Initialize outputs and state *)
Clip := FALSE;                        (* Clip action off *)
Transfer := FALSE;                    (* Transfer action off *)
Release := FALSE;                     (* Release action off *)
ManualMode := FALSE;                  (* Manual Mode off *)
AutoMode := FALSE;                    (* Auto Mode off *)
AlarmActive := FALSE;                 (* No initial alarm *)
State := 0;                           (* Start in Idle *)
AutoTrigger := FALSE;                 (* No Auto sequence *)
AutoTimer(IN := FALSE, PT := T#2s);   (* 2-second timer *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt all operations and reset to safe state *)
    Clip := FALSE;                    (* Stop clip action *)
    Transfer := FALSE;                (* Stop transfer action *)
    Release := FALSE;                 (* Stop release action *)
    ManualMode := FALSE;              (* Disable Manual Mode *)
    AutoMode := FALSE;                (* Disable Auto Mode *)
    AutoTrigger := FALSE;             (* Clear Auto sequence *)
    AutoTimer(IN := FALSE);           (* Reset timer *)
    State := 0;                       (* Return to Idle *)
    AlarmActive := TRUE;              (* Activate alarm *)
    ModeError := FALSE;               (* Clear mode error *)
    RETURN;
END_IF;

(* Mode interlock: Ensure only one mode active *)
(* BtnManual or BtnAuto toggles modes mutually exclusively *)
IF BtnManual AND NOT BtnAuto THEN
    ManualMode := TRUE;               (* Activate Manual Mode *)
    AutoMode := FALSE;                (* Deactivate Auto Mode *)
    AutoTrigger := FALSE;             (* Clear Auto sequence *)
    AutoTimer(IN := FALSE);           (* Reset timer *)
    State := 0;                       (* Reset to Idle *)
    ModeError := FALSE;               (* Clear mode error *)
ELSIF BtnAuto AND NOT BtnManual THEN
    ManualMode := FALSE;              (* Deactivate Manual Mode *)
    AutoMode := TRUE;                 (* Activate Auto Mode *)
    ModeError := FALSE;               (* Clear mode error *)
ELSE
    (* Invalid mode: Both or neither selected *)
    ManualMode := FALSE;              (* Disable Manual Mode *)
    AutoMode := FALSE;                (* Disable Auto Mode *)
    Clip := FALSE;                    (* Stop clip action *)
    Transfer := FALSE;                (* Stop transfer action *)
    Release := FALSE;                 (* Stop release action *)
    AutoTrigger := FALSE;             (* Clear Auto sequence *)
    AutoTimer(IN := FALSE);           (* Reset timer *)
    State := 0;                       (* Reset to Idle *)
    ModeError := TRUE;                (* Flag mode error *)
    AlarmActive := TRUE;              (* Activate alarm *)
END_IF;

(* Manual Mode logic *)
IF ManualMode THEN
    (* Respond to individual commands one at a time *)
    (* Commands are mutually exclusive to prevent conflicting actions *)
    IF CmdClip AND NOT CmdTransfer AND NOT CmdRelease THEN
        Clip := TRUE;                 (* Activate clip action *)
        Transfer := FALSE;            (* Ensure transfer off *)
        Release := FALSE;             (* Ensure release off *)
    ELSIF CmdTransfer AND NOT CmdClip AND NOT CmdRelease THEN
        Clip := FALSE;                (* Ensure clip off *)
        Transfer := TRUE;             (* Activate transfer action *)
        Release := FALSE;             (* Ensure release off *)
    ELSIF CmdRelease AND NOT CmdClip AND NOT CmdTransfer THEN
        Clip := FALSE;                (* Ensure clip off *)
        Transfer := FALSE;            (* Ensure transfer off *)
        Release := TRUE;              (* Activate release action *)
    ELSE
        Clip := FALSE;                (* No valid command: All off *)
        Transfer := FALSE;
        Release := FALSE;
    END_IF;
ELSE
    (* Ensure manual actions off in Auto Mode or no mode *)
    Clip := FALSE;
    Transfer := FALSE;
    Release := FALSE;
END_IF;

(* Auto Mode trigger *)
(* Detect rising edge on BtnAuto to start sequence *)
IF AutoMode AND BtnAuto AND NOT LastBtnAuto THEN
    AutoTrigger := TRUE;               (* Trigger Auto sequence *)
END_IF;
LastBtnAuto := BtnAuto;

(* Auto Mode sequence *)
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: (* Idle *)
            (* Start sequence: Clip product *)
            Clip := TRUE;                 (* Activate clip action *)
            Transfer := FALSE;            (* Ensure transfer off *)
            Release := FALSE;             (* Ensure release off *)
            State := 1;                   (* Move to Clipping *)

        1: (* Clipping *)
            (* Wait for clip action, then start delay *)
            Clip := FALSE;                (* Stop clip action *)
            AutoTimer(IN := TRUE);        (* Start 2-second timer *)
            State := 2;                   (* Move to Waiting *)

        2: (* Waiting *)
            (* Wait 2 seconds, then release *)
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);   (* Reset timer *)
                Transfer := TRUE;         (* Activate transfer action *)
                State := 3;               (* Move to Transferring *)
            END_IF;

        3: (* Transferring *)
            (* Perform transfer, then release *)
            Transfer := FALSE;            (* Stop transfer action *)
            Release := TRUE;              (* Activate release action *)
            State := 4;                   (* Move to Releasing *)

        4: (* Releasing *)
            (* Complete sequence: Reset *)
            Release := FALSE;             (* Stop release action *)
            AutoTrigger := FALSE;         (* Clear trigger *)
            State := 0;                   (* Return to Idle *)
    END_CASE;
ELSE
    (* Ensure Auto actions off if not triggered *)
    IF AutoMode AND NOT AutoTrigger THEN
        Clip := FALSE;
        Transfer := FALSE;
        Release := FALSE;
        State := 0;
    END_IF;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* All actions off unless explicitly activated *)
IF EmergencyStop OR ModeError THEN
    Clip := FALSE;
    Transfer := FALSE;
    Release := FALSE;
END_IF;

END_PROGRAM
