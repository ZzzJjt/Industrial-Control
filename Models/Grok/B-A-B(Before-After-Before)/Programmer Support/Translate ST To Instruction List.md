(* IEC 61131-3 Structured Text: PickAndPlace Program *)
(* Controls a robotic arm and conveyor with manual and automatic modes *)

PROGRAM PickAndPlace
VAR
    (* Inputs *)
    ModeSwitch : BOOL;              (* TRUE=Auto, FALSE=Manual *)
    ManualArmBtn : BOOL;            (* Manual arm movement button *)
    ManualConveyorBtn : BOOL;       (* Manual conveyor button *)
    EmergencyStop : BOOL;           (* TRUE to stop all operations *)
    
    (* Outputs *)
    ArmMove : BOOL;                 (* TRUE to move robotic arm *)
    ConveyorRun : BOOL;             (* TRUE to run conveyor *)
    CycleComplete : BOOL;           (* TRUE when auto cycle completes *)
    
    (* Internal *)
    AutoStep : INT := 0;            (* Auto mode state: 0=Idle, 1=Pick, 2=Move, 3=Place *)
    CycleTimer : TON;               (* Timer for auto cycle steps *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    ArmMove := FALSE;
    ConveyorRun := FALSE;
    AutoStep := 0;
    CycleComplete := FALSE;
    CycleTimer(IN := FALSE);
ELSE
    IF ModeSwitch THEN (* Automatic Mode *)
        CASE AutoStep OF
            0: (* Idle *)
                CycleTimer(IN := FALSE);
                ArmMove := FALSE;
                ConveyorRun := FALSE;
                IF ManualArmBtn THEN (* Trigger auto cycle *)
                    AutoStep := 1;
                    CycleTimer(IN := TRUE, PT := T#3s);
                END_IF;
            1: (* Pick *)
                ArmMove := TRUE;
                ConveyorRun := FALSE;
                IF CycleTimer.Q THEN
                    AutoStep := 2;
                    CycleTimer(IN := TRUE, PT := T#2s);
                END_IF;
            2: (* Move *)
                ArmMove := TRUE;
                ConveyorRun := TRUE;
                IF CycleTimer.Q THEN
                    AutoStep := 3;
                    CycleTimer(IN := TRUE, PT := T#3s);
                END_IF;
            3: (* Place *)
                ArmMove := FALSE;
                ConveyorRun := TRUE;
                IF CycleTimer.Q THEN
                    AutoStep := 0;
                    CycleComplete := TRUE;
                    CycleTimer(IN := FALSE);
                END_IF;
        END_CASE;
    ELSE (* Manual Mode *)
        AutoStep := 0;
        CycleTimer(IN := FALSE);
        CycleComplete := FALSE;
        ArmMove := ManualArmBtn;
        ConveyorRun := ManualConveyorBtn;
    END_IF;
END_IF;

(* Notes:
   - EmergencyStop overrides all operations, resetting outputs and state.
   - Manual mode directly maps button inputs to outputs.
   - Auto mode uses a state machine (AutoStep) with timed steps (3s, 2s, 3s).
*)
END_PROGRAM

(* IEC 61131-3 Instruction List: PickAndPlace Translation *)
(* Equivalent logic to ST program, using IL instructions *)

PROGRAM PickAndPlace_IL
VAR
    ModeSwitch : BOOL;
    ManualArmBtn : BOOL;
    ManualConveyorBtn : BOOL;
    EmergencyStop : BOOL;
    ArmMove : BOOL;
    ConveyorRun : BOOL;
    CycleComplete : BOOL;
    AutoStep : INT := 0;
    CycleTimer : TON;
END_VAR

(* Main IL Logic *)
CAL     CycleTimer              (* Update timer state each cycle *)
LD      EmergencyStop           (* Check emergency stop *)
JMPC    EmergencyHandling       (* Jump to emergency logic if TRUE *)

LD      ModeSwitch              (* Check mode *)
JMPC    AutoMode                (* Jump to auto mode if TRUE *)

(* Manual Mode *)
LD      0
ST      AutoStep                (* Reset state *)
LD      FALSE
ST      CycleTimer.IN           (* Stop timer *)
ST      CycleComplete           (* Clear completion flag *)
LD      ManualArmBtn
ST      ArmMove                 (* Map button to arm *)
LD      ManualConveyorBtn
ST      ConveyorRun             (* Map button to conveyor *)
JMP     EndCycle

AutoMode:
LD      AutoStep
EQ      0
JMPC    AutoIdle
LD      AutoStep
EQ      1
JMPC    AutoPick
LD      AutoStep
EQ      2
JMPC    AutoMove
LD      AutoStep
EQ      3
JMPC    AutoPlace
JMP     EndCycle                (* Fallback *)

AutoIdle:
LD      FALSE
ST      CycleTimer.IN           (* Stop timer *)
ST      ArmMove                 (* Arm off *)
ST      ConveyorRun             (* Conveyor off *)
LD      ManualArmBtn            (* Check trigger *)
JMPC    StartAutoCycle
JMP     EndCycle

StartAutoCycle:
LD      1
ST      AutoStep                (* Move to Pick *)
LD      TRUE
ST      CycleTimer.IN           (* Start timer *)
LD      T#3s
ST      CycleTimer.PT
JMP     EndCycle

AutoPick:
LD      TRUE
ST      ArmMove                 (* Arm on *)
LD      FALSE
ST      ConveyorRun             (* Conveyor off *)
LD      CycleTimer.Q            (* Check timer *)
JMPC    NextStepPick
JMP     EndCycle

NextStepPick:
LD      2
ST      AutoStep                (* Move to Move *)
LD      TRUE
ST      CycleTimer.IN           (* Restart timer *)
LD      T#2s
ST      CycleTimer.PT
JMP     EndCycle

AutoMove:
LD      TRUE
ST      ArmMove                 (* Arm on *)
ST      ConveyorRun             (* Conveyor on *)
LD      CycleTimer.Q            (* Check timer *)
JMPC    NextStepMove
JMP     EndCycle

NextStepMove:
LD      3
ST      AutoStep                (* Move to Place *)
LD      TRUE
ST      CycleTimer.IN           (* Restart timer *)
LD      T#3s
ST      CycleTimer.PT
JMP     EndCycle

AutoPlace:
LD      FALSE
ST      ArmMove                 (* Arm off *)
LD      TRUE
ST      ConveyorRun             (* Conveyor on *)
LD      CycleTimer.Q            (* Check timer *)
JMPC    CompleteCycle
JMP     EndCycle

CompleteCycle:
LD      0
ST      AutoStep                (* Back to Idle *)
LD      FALSE
ST      CycleTimer.IN           (* Stop timer *)
LD      TRUE
ST      CycleComplete           (* Signal completion *)
JMP     EndCycle

EmergencyHandling:
LD      FALSE
ST      ArmMove                 (* Stop all *)
ST      ConveyorRun
ST      CycleTimer.IN
LD      0
ST      AutoStep
LD      FALSE
ST      CycleComplete
JMP     EndCycle

EndCycle:
(* End of cycle, PLC scan continues *)

(* Notes:
   - IL preserves ST logic using LD, ST, JMPC, and JMP for control flow.
   - Timer (CycleTimer) updated with CAL each cycle, IN/PT set per state.
   - Labels (AutoIdle, AutoPick, etc.) map to ST CASE branches.
   - EmergencyStop handled first to ensure safety.
   - Manual mode directly maps inputs to outputs, auto mode uses state machine.
*)
END_PROGRAM
