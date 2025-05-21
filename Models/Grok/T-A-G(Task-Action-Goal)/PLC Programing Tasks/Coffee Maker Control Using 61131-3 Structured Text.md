(* Program: Automated Coffee Maker Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages tank filling, mixing, and dispensing with emergency stop handling *)
PROGRAM PRG_CoffeeMakerControl
VAR
    (* Inputs *)
    Start : BOOL;                     (* Start button to initiate cycle *)
    CoffeeMilk : BOOL;                (* Select coffee with milk *)
    CoffeeOnly : BOOL;                (* Select coffee only *)
    EmergencyStop : BOOL;             (* Emergency stop button *)
    MixerLevelFull : BOOL;            (* Sensor: TRUE if mixer tank is full (130 ml) *)
    
    (* Outputs *)
    CoffeeValve : BOOL;               (* Coffee valve: TRUE = Open *)
    MilkValve : BOOL;                 (* Milk valve: TRUE = Open *)
    OutputValve : BOOL;               (* Output valve: TRUE = Open *)
    Mixer : BOOL;                     (* Mixer motor: TRUE = Running *)
    
    (* Internal Variables *)
    State : UINT;                     (* State machine: 0=Idle, 1=Filling, 2=Mixing, 3=Dispensing *)
    MixTimer : TON;                   (* Timer for 4-second mixing *)
    LastStart : BOOL;                 (* For rising edge detection on Start *)
    SafeToRun : BOOL;                 (* Internal flag: TRUE if safe to start *)
END_VAR

(* Initialize outputs and state *)
CoffeeValve := FALSE;
MilkValve := FALSE;
OutputValve := FALSE;
Mixer := FALSE;
State := 0;                           (* Start in Idle state *)
SafeToRun := TRUE;                    (* Initially safe to run *)
MixTimer(IN := FALSE, PT := T#4s);    (* 4-second mixing timer *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Immediately halt all operations and reset to safe state *)
    CoffeeValve := FALSE;             (* Close coffee valve *)
    MilkValve := FALSE;               (* Close milk valve *)
    OutputValve := FALSE;             (* Close output valve *)
    Mixer := FALSE;                   (* Stop mixer *)
    MixTimer(IN := FALSE);            (* Reset timer *)
    State := 0;                       (* Return to Idle *)
    SafeToRun := FALSE;               (* Lock out until reset *)
ELSE
    (* Normal operation: State machine *)
    CASE State OF
        0: (* Idle *)
            (* Wait for Start button and valid drink selection *)
            (* Use rising edge to prevent continuous triggering *)
            IF Start AND NOT LastStart AND SafeToRun THEN
                IF CoffeeMilk AND NOT CoffeeOnly THEN
                    (* Coffee with milk: Open both valves *)
                    CoffeeValve := TRUE;
                    MilkValve := TRUE;
                    State := 1;           (* Move to Filling *)
                ELSIF CoffeeOnly AND NOT CoffeeMilk THEN
                    (* Coffee only: Open coffee valve *)
                    CoffeeValve := TRUE;
                    MilkValve := FALSE;
                    State := 1;           (* Move to Filling *)
                ELSE
                    (* Invalid selection: Do nothing *)
                    (* Prevents operation if both or neither selected *)
                END_IF;
            END_IF;

        1: (* Filling *)
            (* Monitor mixer tank level *)
            IF MixerLevelFull THEN
                (* Tank full: Close input valves, start mixing *)
                CoffeeValve := FALSE;      (* Close coffee valve *)
                MilkValve := FALSE;        (* Close milk valve *)
                Mixer := TRUE;             (* Start mixer *)
                MixTimer(IN := TRUE);      (* Start 4-second timer *)
                State := 2;                (* Move to Mixing *)
            END_IF;

        2: (* Mixing *)
            (* Wait for mixing to complete *)
            IF MixTimer.Q THEN
                (* Mixing done: Stop mixer, open output valve *)
                Mixer := FALSE;            (* Stop mixer *)
                MixTimer(IN := FALSE);     (* Reset timer *)
                OutputValve := TRUE;       (* Open output valve *)
                State := 3;                (* Move to Dispensing *)
            END_IF;

        3: (* Dispensing *)
            (* Assume dispensing completes when tank is empty *)
            (* Simplification: No explicit empty sensor *)
            IF NOT MixerLevelFull THEN
                (* Tank empty: Close output valve, reset to Idle *)
                OutputValve := FALSE;      (* Close output valve *)
                State := 0;                (* Return to Idle *)
            END_IF;
    END_CASE;
END_IF;

(* Reset SafeToRun after emergency stop *)
(* Requires Start button release and no emergency condition *)
IF NOT EmergencyStop AND NOT Start AND NOT LastStart THEN
    SafeToRun := TRUE;                (* Restore safe state *)
END_IF;

(* Update LastStart for rising edge detection *)
LastStart := Start;

(* Ensure safe state on power-up or PLC stop *)
(* All outputs FALSE unless explicitly activated *)
IF NOT SafeToRun THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
END_IF;

END_PROGRAM
