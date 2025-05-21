(* Program: Reactor Temperature Control with OOP *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Demonstrates classes, inheritance, and polymorphism in IEC 61131-3 *)
PROGRAM PRG_ReactorTempControl
VAR
    ReactorInstance : Reactor;        (* Reactor instance *)
    PIDCtrl : PIDController;          (* PID controller instance *)
    OnOffCtrl : OnOffController;      (* On-Off controller instance *)
    Temp_PV : REAL := 70.0;           (* Simulated process variable, °C *)
    UsePID : BOOL := TRUE;            (* TRUE to use PID, FALSE for On-Off *)
END_VAR

(* Base Controller Class *)
CLASS Controller ABSTRACT
    METHOD PUBLIC ABSTRACT ControlLoop : REAL
        VAR_INPUT
            Setpoint : REAL;
            ProcessValue : REAL;
        END_VAR
    END_METHOD
END_CLASS

(* PID Controller Class *)
CLASS PIDController EXTENDS Controller
    VAR
        Kp : REAL := 2.0;             (* Proportional gain *)
        Ki : REAL := 0.5;             (* Integral gain *)
        Integral : REAL;              (* Accumulated integral *)
    END_VAR
    METHOD PUBLIC ControlLoop : REAL
        VAR_INPUT
            Setpoint : REAL;
            ProcessValue : REAL;
        END_VAR
        VAR
            Error : REAL;
        END_VAR
        Error := Setpoint - ProcessValue;
        Integral := Integral + Error * 0.1; (* 100 ms sample time *)
        ControlLoop := Kp * Error + Ki * Integral;
        IF ControlLoop > 100.0 THEN
            ControlLoop := 100.0;
        ELSIF ControlLoop < 0.0 THEN
            ControlLoop := 0.0;
        END_IF;
    END_METHOD
END_CLASS

(* On-Off Controller Class *)
CLASS OnOffController EXTENDS Controller
    VAR
        Hysteresis : REAL := 2.0;     (* Hysteresis band, °C *)
    END_VAR
    METHOD PUBLIC ControlLoop : REAL
        VAR_INPUT
            Setpoint : REAL;
            ProcessValue : REAL;
        END_VAR
        IF ProcessValue < (Setpoint - Hysteresis) THEN
            ControlLoop := 100.0;     (* Full output *)
        ELSIF ProcessValue > (Setpoint + Hysteresis) THEN
            ControlLoop := 0.0;       (* No output *)
        END_IF;
    END_METHOD
END_CLASS

(* Reactor Class *)
CLASS Reactor
    VAR
        Ctrl : REF_TO Controller;     (* Reference to base Controller *)
        Setpoint : REAL := 75.0;      (* Target temperature, °C *)
        ValvePosition : REAL;         (* Valve position, 0.0–100.0% *)
    END_VAR
    METHOD PUBLIC Update
        VAR_INPUT
            ProcessValue : REAL;      (* Current temperature *)
        END_VAR
        (* Polymorphic call to ControlLoop *)
        ValvePosition := Ctrl^.ControlLoop(Setpoint, ProcessValue);
    END_METHOD
END_CLASS

(* Main program logic *)
IF UsePID THEN
    ReactorInstance.Ctrl := REF(PIDCtrl); (* Assign PID controller *)
ELSE
    ReactorInstance.Ctrl := REF(OnOffCtrl); (* Assign On-Off controller *)
END_IF;
ReactorInstance.Update(Temp_PV); (* Update with current temperature *)
END_PROGRAM
