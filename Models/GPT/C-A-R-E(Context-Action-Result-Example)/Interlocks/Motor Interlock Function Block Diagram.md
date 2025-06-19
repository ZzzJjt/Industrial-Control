FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;   (* TRUE if equipment 1 is running *)
    Equipment2Running : BOOL;   (* TRUE if equipment 2 is running *)
    Equipment3Running : BOOL;   (* TRUE if equipment 3 is running *)
END_VAR

VAR_OUTPUT
    AllowStart : BOOL;          (* TRUE only when all equipment is stopped *)
END_VAR

(* === Interlock Logic === *)
AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;


VAR
    MixMotorInterlock : MotorInterlock;
    AgitatorRunning   : BOOL;
    FeedPumpRunning   : BOOL;
    ValveRunning      : BOOL;
    MotorCanStart     : BOOL;
END_VAR

(* Apply input conditions *)
MixMotorInterlock.Equipment1Running := AgitatorRunning;
MixMotorInterlock.Equipment2Running := FeedPumpRunning;
MixMotorInterlock.Equipment3Running := ValveRunning;

(* Output *)
MotorCanStart := MixMotorInterlock.AllowStart;
