(* PickAndPlace in IEC 61131-3 Instruction List *)
(* Controls a pick-and-place system with manual and automatic modes *)
(* States: 0=Idle, 1=Picking, 2=PickWait, 3=Placing, 4=PlaceWait *)
(* Uses TON for 2s delays and R_TRIG for edge detection *)

PROGRAM PickAndPlace
VAR
    (* Inputs *)
    BtnManual : BOOL; (* Manual mode button *)
    BtnAuto : BOOL; (* Auto mode button *)
    CmdPick : BOOL; (* Manual pick command *)
    CmdPlace : BOOL; (* Manual place command *)
    
    (* Outputs *)
    PickActuator : BOOL; (* Pick actuator control *)
    PlaceActuator : BOOL; (* Place actuator control *)
    
    (* Internal variables *)
    ManualMode : BOOL; (* TRUE if manual mode *)
    AutoMode : BOOL; (* TRUE if auto mode *)
    CycleActive : BOOL; (* TRUE during auto cycle *)
    State : INT := 0; (* Auto state *)
    AutoTimer : TON; (* Timer for delays *)
    AutoTrigger : R_TRIG; (* Edge detection for BtnAuto *)
    ModeTrigger : R_TRIG; (* Edge detection for mode buttons *)
END_VAR

(* Main IL program *)
(* Edge detection *)
CAL ModeTrigger(CLK := BtnManual OR BtnAuto)
CAL AutoTrigger(CLK := BtnAuto)

(* Mode selection *)
LD ModeTrigger.Q
JMPC ModeCheck
JMP ModeSkip
ModeCheck:
  LD BtnManual
  AND NOT BtnAuto
  JMPC SetManual
  LD BtnAuto
  AND NOT BtnManual
  JMPC SetAuto
  JMP ModeSkip
SetManual:
  LD TRUE
  ST ManualMode
  LD FALSE
  ST AutoMode
  ST CycleActive
  LD 0
  ST State
  LD FALSE
  ST AutoTimer.IN
  JMP ModeSkip
SetAuto:
  LD FALSE
  ST ManualMode
  LD TRUE
  ST AutoMode
ModeSkip:

(* Manual mode logic *)
LD ManualMode
JMPC ManualLogic
JMP AutoLogic
ManualLogic:
  LD CmdPick
  JMPC ManualPick
  LD CmdPlace
  JMPC ManualPlace
  LD FALSE
  ST PickActuator
  ST PlaceActuator
  JMP EndLogic
ManualPick:
  LD TRUE
  ST PickActuator
  LD FALSE
  ST PlaceActuator
  JMP EndLogic
ManualPlace:
  LD FALSE
  ST PickActuator
  LD TRUE
  ST PlaceActuator
  JMP EndLogic

(* Automatic mode logic *)
AutoLogic:
  LD AutoMode
  AND AutoTrigger.Q
  AND NOT CycleActive
  JMPC StartCycle
  JMP StateMachine
StartCycle:
  LD TRUE
  ST CycleActive
  LD 1
  ST State
StateMachine:
  LD State
  EQ 0
  JMPC State0
  LD State
  EQ 1
  JMPC State1
  LD State
  EQ 2
  JMPC State2
  LD State
  EQ 3
  JMPC State3
  LD State
  EQ 4
  JMPC State4
  JMP EndLogic

State0: (* Idle *)
  LD FALSE
  ST PickActuator
  ST PlaceActuator
  JMP EndLogic

State1: (* Picking *)
  LD TRUE
  ST PickActuator
  LD FALSE
  ST PlaceActuator
  CAL AutoTimer(IN := TRUE, PT := T#2s)
  LD AutoTimer.Q
  JMPC NextState1
  JMP EndLogic
NextState1:
  LD FALSE
  ST AutoTimer.IN
  LD 2
  ST State
  JMP EndLogic

State2: (* Pick Wait *)
  LD FALSE
  ST PickActuator
  ST PlaceActuator
  CAL AutoTimer(IN := TRUE, PT := T#2s)
  LD AutoTimer.Q
  JMPC NextState2
  JMP EndLogic
NextState2:
  LD FALSE
  ST AutoTimer.IN
  LD 3
  ST State
  JMP EndLogic

State3: (* Placing *)
  LD FALSE
  ST PickActuator
  LD TRUE
  ST PlaceActuator
  CAL AutoTimer(IN := TRUE, PT := T#2s)
  LD AutoTimer.Q
  JMPC NextState3
  JMP EndLogic
NextState3:
  LD FALSE
  ST AutoTimer.IN
  LD 4
  ST State
  JMP EndLogic

State4: (* Place Wait *)
  LD FALSE
  ST PickActuator
  ST PlaceActuator
  CAL AutoTimer(IN := TRUE, PT := T#2s)
  LD AutoTimer.Q
  JMPC NextState4
  JMP EndLogic
NextState4:
  LD FALSE
  ST AutoTimer.IN
  ST CycleActive
  LD 0
  ST State

EndLogic:
END_PROGRAM
