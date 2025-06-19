(* Bottle Ejection Control Logic in IEC 61131-3 Structured Text *)
(* Purpose: Automate detection and removal of empty bottles to ensure packaging line integrity *)

PROGRAM BottleEjectionControl
VAR
    (* Inputs *)
    BottlePresentSensor : BOOL;      (* TRUE when any bottle is detected on conveyor *)
    EmptyBottleSensor : BOOL;        (* TRUE when an empty bottle is detected *)

    (* Outputs *)
    ConveyorMotor : BOOL;            (* Controls conveyor motor, always running *)
    EjectCylinder : BOOL;            (* Controls pneumatic cylinder for ejecting empty bottles *)

    (* Internal Variables *)
    EjectTimer : TON;                (* Timer for controlling ejection duration *)
    EjectionActive : BOOL;           (* Tracks ejection state to ensure proper timing *)

END_VAR

(* Main Control Logic *)
(* 1. Conveyor Control *)
ConveyorMotor := TRUE;  (* Conveyor runs continuously to maintain production flow *)

(* 2. Empty Bottle Detection and Ejection *)
IF BottlePresentSensor AND EmptyBottleSensor THEN
    (* Empty bottle detected: trigger ejection *)
    EjectionActive := TRUE;
    EjectTimer(IN := TRUE, PT := T#500ms);  (* Activate cylinder for 0.5 seconds *)
ELSE
    (* No empty bottle: reset timer if not active *)
    EjectTimer(IN := FALSE);
    EjectionActive := FALSE;
END_IF;

(* 3. Cylinder Control *)
IF EjectionActive AND NOT EjectTimer.Q THEN
    EjectCylinder := TRUE;  (* Extend cylinder during ejection *)
ELSE
    EjectCylinder := FALSE;  (* Retract cylinder after timer or no ejection *)
END_IF;

(* Notes:
   - ConveyorMotor runs continuously to ensure seamless bottle flow
   - Ejection occurs only when both BottlePresentSensor and EmptyBottleSensor are TRUE
   - EjectTimer ensures the pneumatic cylinder extends for exactly 500ms to avoid jams
   - EjectCylinder retracts after timer completion or when no empty bottle is detected
   - Safety:
     - Ejection is precise to avoid affecting filled bottles
     - Timer prevents prolonged or repeated cylinder activation
   - Physical Integration:
     - BottlePresentSensor: Proximity sensor to detect any bottle
     - EmptyBottleSensor: Specialized sensor (e.g., weight or optical) for empty bottles
     - ConveyorMotor: Relay or VFD control for conveyor
     - EjectCylinder: Solenoid valve for pneumatic cylinder
   - Scalability: Add more sensors or adjust EjectTimer duration for different bottle sizes
   - Maintenance: Add HMI to monitor sensor states and ejection frequency
*)
END_PROGRAM
