VAR
    (* Input variables *)
    Predicted_Load  : REAL;             (* Predicted load from upstream sensor [kg] *)
    
    (* Control parameters *)
    Base_Speed      : REAL := 1.0;      (* Base conveyor speed [m/s] *)
    Gain_FF         : REAL := 0.02;     (* Feedforward gain [m/s per kg] *)
    
    (* Output variable *)
    Conveyor_Speed  : REAL;             (* Speed reference for motor controller [m/s] *)
    
    (* Constants for operational limits *)
    Max_Speed       : REAL := 2.0;      (* Maximum allowable speed [m/s] *)
    Min_Speed       : REAL := 0.5;      (* Minimum allowable speed [m/s] *)
END_VAR

(* Feedforward control for conveyor speed *)
(* Calculate speed based on predicted load *)
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

(* Clamp speed to safe operational limits *)
IF Conveyor_Speed > Max_Speed THEN
    Conveyor_Speed := Max_Speed;
ELSIF Conveyor_Speed < Min_Speed THEN
    Conveyor_Speed := Min_Speed;
END_IF;

(* Conveyor_Speed is sent to the motor controller *)
(* Example: Write Conveyor_Speed to analog output for motor drive *)
