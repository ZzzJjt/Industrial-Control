PROGRAM ConveyorFeedforwardControl
VAR
    (* Inputs *)
    Predicted_Load : REAL; (* Predicted material load (kg) from upstream sensor *)
    Base_Speed : REAL := 1.0; (* Base conveyor speed (m/s) *)
    Gain_FF : REAL := 0.02; (* Feedforward gain: speed increase per unit load (m/s per kg) *)
    
    (* Constraints *)
    Speed_Min : REAL := 0.5; (* Minimum safe conveyor speed (m/s) *)
    Speed_Max : REAL := 2.0; (* Maximum safe conveyor speed (m/s) *)
    Load_Max : REAL := 100.0; (* Maximum expected load (kg) for validation *)
    
    (* Outputs *)
    Conveyor_Speed : REAL; (* Computed conveyor speed (m/s) *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
END_VAR

(* Initialize outputs *)
Conveyor_Speed := Speed_Min; (* Default to minimum speed for safety *)
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(Predicted_Load) OR Predicted_Load < 0.0 OR Predicted_Load > Load_Max THEN
    (* Invalid or out-of-range load: use minimum speed *)
    Conveyor_Speed := Speed_Min;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Feedforward control logic *)
(* Speed = Base_Speed + Gain_FF * Predicted_Load *)
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

(* Clamp speed within safe operational bounds *)
IF Conveyor_Speed > Speed_Max THEN
    Conveyor_Speed := Speed_Max;
ELSIF Conveyor_Speed < Speed_Min THEN
    Conveyor_Speed := Speed_Min;
END_IF;

(* Validate output *)
IF NOT IS_VALID_REAL(Conveyor_Speed) THEN
    Conveyor_Speed := Speed_Min;
    ErrorCode := 1;
END_IF;

(* Helper function to check if a REAL value is valid *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

END_PROGRAM
