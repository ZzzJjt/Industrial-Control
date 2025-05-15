(* Coffee Machine Control Logic in IEC 61131-3 Structured Text *)
(* Purpose: Automate coffee machine with safety interlocks, multiple drink modes, and precise state control *)

PROGRAM CoffeeMachineControl
VAR
    (* Inputs *)
    EmergencyStop : BOOL;         (* TRUE when emergency stop button is pressed *)
    StartButton : BOOL;           (* TRUE to start brewing process *)
    CoffeeMilkButton : BOOL;      (* TRUE to select coffee + milk mode *)
    CoffeeOnlyButton : BOOL;      (* TRUE to select coffee only mode *)
    MixerLevelFull : BOOL;        (* TRUE when mixer tank reaches 130 ml *)
    MixerFault : BOOL;            (* TRUE if mixer motor or sensor fault detected *)

    (* Outputs *)
    CoffeeValve : BOOL;           (* Controls coffee tank valve *)
    MilkValve : BOOL;             (* Controls milk tank valve *)
    OutputValve : BOOL;           (* Controls mixer tank output valve *)
    Mixer : BOOL;                 (* Controls mixer motor *)

    (* Internal Variables *)
    State : INT := 0;             (* State machine: 0=Idle, 1=Filling, 2=Mixing, 3=Dispensing *)
    MixTimer : TON;               (* 4-second timer for mixing process *)
    DispenseTimer : TON;          (* Timer for dispensing duration, e.g., 2 seconds *)
    
    (* Safety Flag *)
    SafeToOperate : BOOL := TRUE; (* FALSE if emergency stop or fault detected *)
END_VAR

(* Main Control Logic *)
(* 1. Emergency Stop and Fault Check *)
IF EmergencyStop OR MixerFault THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    Mixer := FALSE;
    OutputValve := FALSE;
    SafeToOperate := FALSE;
    State := 0;  (* Force return to Idle *)
ELSE
    SafeToOperate := TRUE;  (* Allow operation if no faults *)
END_IF;

(* 2. State Machine *)
IF SafeToOperate THEN
    CASE State OF
        0: (* Idle State *)
            IF StartButton THEN
                IF CoffeeMilkButton AND NOT CoffeeOnlyButton THEN
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                    State := 1;  (* Move to Filling *)
                ELSIF CoffeeOnlyButton AND NOT CoffeeMilkButton THEN
                    CoffeeValve := TRUE;
                    MilkValve := FALSE;
                    State := 1;  (* Move to Filling *)
                END_IF;
            END_IF;

        1: (* Filling State *)
            IF MixerLevelFull THEN
                CoffeeValve := FALSE;
                MilkValve := FALSE;
                Mixer := TRUE;
                MixTimer(IN := TRUE, PT := T#4s);  (* Start 4-second mixing *)
                State := 2;  (* Move to Mixing *)
            END_IF;

        2: (* Mixing State *)
            IF MixTimer.Q THEN
                Mixer := FALSE;
                OutputValve := TRUE;
                DispenseTimer(IN := TRUE, PT := T#2s);  (* Dispense for 2 seconds *)
                State := 3;  (* Move to Dispensing *)
            END_IF;

        3: (* Dispensing State *)
            IF DispenseTimer.Q THEN
                OutputValve := FALSE;
                State := 0;  (* Return to Idle *)
                MixTimer(IN := FALSE);  (* Reset timers *)
                DispenseTimer(IN := FALSE);
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - EmergencyStop and MixerFault take precedence, shutting down all outputs and resetting to Idle
   - SafeToOperate ensures no operation during fault conditions
   - State machine ensures sequential operation: Idle -> Filling -> Mixing -> Dispensing -> Idle
   - MixTimer (4s) controls mixing duration; DispenseTimer (2s) ensures controlled dispensing
   - CoffeeMilkButton and CoffeeOnlyButton are mutually exclusive for mode selection
   - Physical integration:
     - Inputs: Push-buttons (StartButton, CoffeeMilkButton, CoffeeOnlyButton, EmergencyStop)
     - MixerLevelFull: Level sensor in mixer tank (e.g., float switch)
     - MixerFault: Fault signal from mixer motor or sensor
     - Outputs: Relays for CoffeeValve, MilkValve, OutputValve, and Mixer motor
   - For maintenance: Add HMI display for State and fault status
*)
END_PROGRAM
