PROGRAM PickAndPlaceControl
VAR
    (* Inputs *)
    BtnManual : BOOL; (* TRUE to activate manual mode *)
    BtnAuto : BOOL; (* TRUE to activate auto mode or trigger auto cycle *)
    CmdClip : BOOL; (* TRUE to command clip action in manual mode *)
    CmdTransfer : BOOL; (* TRUE to command transfer action in manual mode *)
    CmdRelease : BOOL; (* TRUE to command release action in manual mode *)
    
    (* Outputs *)
    Clip : BOOL; (* TRUE to activate clip (pick product from Conveyor A) *)
    Transfer : BOOL; (* TRUE to activate transfer (move to Conveyor B) *)
    Release : BOOL; (* TRUE to activate release (drop on Conveyor B) *)
    ManualMode : BOOL; (* TRUE if manual mode is active *)
    AutoMode : BOOL; (* TRUE if auto mode is active *)
    
    (* Internal variables *)
    AutoTrigger : BOOL := FALSE; (* TRUE to start auto cycle *)
    State : INT := 0; (* Auto mode state: 0=Idle, 1=Clip, 2=Transfer, 3=Release *)
    AutoTimer : TON; (* 2-second timer for transfer delay *)
    PrevBtnAuto : BOOL; (* Tracks previous BtnAuto state for edge detection *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid mode selection *)
END_VAR

(* Initialize outputs *)
Clip := FALSE;
Transfer := FALSE;
Release := FALSE;
ErrorCode := 0;

(* Mode interlock: ensure mutually exclusive modes *)
IF BtnManual AND BtnAuto THEN
    (* Invalid: both buttons pressed *)
    ManualMode := FALSE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;
    State := 0;
    AutoTimer(IN := FALSE);
    ErrorCode := 1;
ELSIF BtnManual THEN
    (* Activate manual mode *)
    ManualMode := TRUE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;
    State := 0;
    AutoTimer(IN := FALSE);
    ErrorCode := 0;
ELSIF BtnAuto THEN
    (* Activate auto mode *)
    ManualMode := FALSE;
    AutoMode := TRUE;
    ErrorCode := 0;
ELSE
    (* No mode selected: maintain current mode or default to none *)
    ErrorCode := 0;
END_IF;

(* Manual mode operation *)
IF ManualMode THEN
    (* Direct control based on command inputs *)
    Clip := CmdClip;
    Transfer := CmdTransfer;
    Release := CmdRelease;
ELSE
    (* Ensure outputs are off in auto mode or no mode *)
    IF NOT AutoMode THEN
        Clip := FALSE;
        Transfer := FALSE;
        Release := FALSE;
    END_IF;
END_IF;

(* Auto mode trigger: detect rising edge of BtnAuto *)
IF AutoMode AND BtnAuto AND NOT PrevBtnAuto THEN
    AutoTrigger := TRUE;
END_IF;

(* Auto mode sequence *)
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: (* Idle: start clip *)
            Clip := TRUE;
            Transfer := FALSE;
            Release := FALSE;
            State := 1;
        
        1: (* Clip: move to transfer with delay *)
            Clip := FALSE;
            Transfer := TRUE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;
        
        2: (* Transfer: move to release *)
            Transfer := FALSE;
            Release := TRUE;
            State := 3;
        
        3: (* Release: complete cycle *)
            Release := FALSE;
            AutoTrigger := FALSE; (* Reset trigger for next cycle *)
            State := 0;
        
        ELSE
            (* Invalid state: reset *)
            Clip := FALSE;
            Transfer := FALSE;
            Release := FALSE;
            AutoTrigger := FALSE;
            State := 0;
            AutoTimer(IN := FALSE);
    END_CASE;
END_IF;

(* Update previous button state *)
PrevBtnAuto := BtnAuto;

END_PROGRAM
