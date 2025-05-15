(* Pick-and-Place Robot Control in IEC 61131-3 Structured Text *)
(* Purpose: Manage Manual and Auto modes for safe and reliable robotic operation *)

PROGRAM PickAndPlaceControl
VAR
    (* Inputs *)
    BtnManual : BOOL;               (* TRUE to select Manual Mode *)
    BtnAuto : BOOL;                 (* TRUE to select or trigger Auto Mode *)
    CmdClip : BOOL;                 (* Manual command to clip from conveyor A *)
    CmdTransfer : BOOL;             (* Manual command to transfer to conveyor B *)
    CmdRelease : BOOL;              (* Manual command to release on conveyor B *)

    (* Outputs *)
    ActClip : BOOL;                 (* Activates clip mechanism *)
    ActTransfer : BOOL;             (* Activates transfer mechanism *)
    ActRelease : BOOL;              (* Activates release mechanism *)

    (* Internal Variables *)
    ManualMode : BOOL;              (* TRUE when Manual Mode is active *)
    AutoMode : BOOL;                (* TRUE when Auto Mode is active *)
    AutoTrigger : BOOL;             (* TRUE to start Auto Mode sequence *)
    State : INT := 0;               (* Auto Mode state: 0=Clip, 1=Transfer, 2=Release *)
    AutoTimer : TON;                (* 2-second timer for transfer delay *)
END_VAR

(* Main Control Logic *)
(* 1. Mode Interlock *)
(* Ensure only one mode is active at a time *)
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;  (* Reset Auto sequence when switching modes *)
    State := 0;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF;

(* 2. Manual Mode Operations *)
IF ManualMode THEN
    ActClip := CmdClip;        (* Directly map clip command to actuator *)
    ActTransfer := CmdTransfer; (* Directly map transfer command to actuator *)
    ActRelease := CmdRelease;   (* Directly map release command to actuator *)
ELSE
    ActClip := FALSE;
    ActTransfer := FALSE;
    ActRelease := FALSE;       (* Disable manual actuators in Auto Mode *)
END_IF;

(* 3. Auto Mode Sequence *)
IF AutoMode AND BtnAuto AND NOT AutoTrigger THEN
    AutoTrigger := TRUE;       (* Trigger Auto sequence on BtnAuto press *)
END_IF;

IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: (* Clip State *)
            ActClip := TRUE;
            ActTransfer := FALSE;
            ActRelease := FALSE;
            State := 1;        (* Move to Transfer *)
            
        1: (* Transfer State *)
            ActClip := FALSE;
            ActTransfer := TRUE;
            ActRelease := FALSE;
            AutoTimer(IN := TRUE, PT := T#2s);  (* Start 2-second delay *)
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;    (* Move to Release *)
            END_IF;
            
        2: (* Release State *)
            ActClip := FALSE;
            ActTransfer := FALSE;
            ActRelease := TRUE;
            AutoTrigger := FALSE;  (* Reset trigger *)
            State := 0;           (* Return to Clip for next cycle *)
    END_CASE;
END_IF;

(* Notes:
   - Mode Interlock:
     - BtnManual and BtnAuto ensure ManualMode and AutoMode are mutually exclusive
     - Switching modes resets Auto sequence (State and AutoTrigger)
   - Manual Mode:
     - Directly maps CmdClip, CmdTransfer, CmdRelease to actuators
     - Only active when ManualMode is TRUE
   - Auto Mode:
     - Triggered by BtnAuto, runs full sequence: Clip → Transfer (2s delay) → Release
     - AutoTrigger ensures one-shot execution until re-triggered
   - Physical Integration:
     - Inputs: Push-buttons or HMI for BtnManual, BtnAuto, CmdClip, CmdTransfer, CmdRelease
     - Outputs: Relays or signals for ActClip (gripper), ActTransfer (arm), ActRelease (gripper release)
   - Safety:
     - Mode exclusivity prevents conflicting commands
     - Actuators are disabled when not in use or in wrong mode
   - Scalability: Add more states or commands by extending State machine
   - Maintenance: Add HMI to display Mode, State, and actuator status
*)
END_PROGRAM
