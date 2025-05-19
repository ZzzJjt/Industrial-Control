VAR
    (* PID Function Block Instance *)
    PID_Temp       : OSCAT_PID;        (* OSCAT PID block instance *)

    (* Inputs *)
    Temp_PV        : REAL;             (* Measured temperature [°C] *)
    Temp_SP        : REAL := 75.0;     (* Target temperature [°C] *)
    Kp             : REAL := 2.0;      (* Proportional gain *)
    Ki             : REAL := 0.5;      (* Integral gain *)
    Kd             : REAL := 0.1;      (* Derivative gain *)
    Manual_Mode    : BOOL := FALSE;    (* Manual override flag *)
    Manual_Output  : REAL := 0.0;      (* Manual output value [0-100%] *)

    (* Output *)
    Valve_Position : REAL;             (* Control signal to valve [0-100%] *)
    Error          : REAL;             (* Current error [°C] *)

    (* Limits *)
    Limit_High     : REAL := 100.0;    (* Max valve position [%] *)
    Limit_Low      : REAL := 0.0;      (* Min valve position [%] *)

    (* Timing *)
    Sample_Time    : TIME := T#100ms;  (* Sampling period *)
END_VAR

(* Configure and execute PID block *)
PID_Temp(
    SP := Temp_SP,                 (* Setpoint: 75°C *)
    PV := Temp_PV,                 (* Measured temperature *)
    Kp := Kp,                      (* Proportional gain *)
    Ki := Ki,                      (* Integral gain *)
    Kd := Kd,                      (* Derivative gain *)
    MANUAL := Manual_Mode,         (* Manual mode flag *)
    MAN_OUT := Manual_Output,      (* Manual output value *)
    LIMIT_HI := Limit_High,        (* Max output limit *)
    LIMIT_LO := Limit_Low,         (* Min output limit *)
    T_Sample := Sample_Time        (* Sampling period *)
);

(* Retrieve outputs *)
Valve_Position := PID_Temp.OUT;    (* Valve control signal *)
Error := PID_Temp.ERROR;           (* Error for monitoring *)

(* Valve_Position is sent to the heating valve *)
(* Example: Write Valve_Position to analog output for valve control *)
