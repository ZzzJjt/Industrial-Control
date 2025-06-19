(* IEC 61131-3 Version 3.0 OOP Example: Actuator Control *)
(* Demonstrates classes, methods, inheritance, and polymorphism *)

(* Base Class: Generic Actuator *)
FUNCTION_BLOCK Actuator
VAR_INPUT
    Enable : BOOL;              (* TRUE to enable actuator *)
END_VAR
VAR_OUTPUT
    IsActive : BOOL;            (* TRUE if actuator is active *)
    Status : STRING[80];        (* Current status message *)
END_VAR
VAR
    Position : REAL := 0.0;     (* Actuator position, 0.0 to 100.0% *)
END_VAR

(* Virtual Method: Start *)
METHOD PUBLIC VIRTUAL Start : BOOL
VAR_INPUT
    TargetPosition : REAL;      (* Desired position, 0.0 to 100.0 *)
END_VAR
IF Enable AND TargetPosition >= 0.0 AND TargetPosition <= 100.0 THEN
    Position := TargetPosition;
    IsActive := TRUE;
    Status := 'Actuator Started';
    Start := TRUE;
ELSE
    IsActive := FALSE;
    Status := 'Start Failed: Invalid Input or Disabled';
    Start := FALSE;
END_IF;
END_METHOD

(* Virtual Method: Stop *)
METHOD PUBLIC VIRTUAL Stop : BOOL
IsActive := FALSE;
Position := 0.0;
Status := 'Actuator Stopped';
Stop := TRUE;
END_METHOD

(* Method: GetPosition *)
METHOD PUBLIC GetPosition : REAL
GetPosition := Position;
END_METHOD
END_FUNCTION_BLOCK

(* Derived Class: ValveController, Extends Actuator *)
FUNCTION_BLOCK ValveController EXTENDS Actuator
VAR
    FlowRate : REAL := 0.0;     (* Current flow rate in L/min *)
    MaxFlow : REAL := 100.0;    (* Maximum flow rate in L/min *)
END_VAR

(* Override Start Method *)
METHOD PUBLIC OVERRIDE Start : BOOL
VAR_INPUT
    TargetPosition : REAL;      (* Desired valve opening, 0.0 to 100.0% *)
END_VAR
IF Enable AND TargetPosition >= 0.0 AND TargetPosition <= 100.0 THEN
    Position := TargetPosition;
    FlowRate := (TargetPosition / 100.0) * MaxFlow; (* Calculate flow *)
    IsActive := TRUE;
    Status := 'Valve Opened, Flow Rate: ' + REAL_TO_STRING(FlowRate) + ' L/min';
    Start := TRUE;
ELSE
    IsActive := FALSE;
    FlowRate := 0.0;
    Status := 'Valve Start Failed: Invalid Input or Disabled';
    Start := FALSE;
END_IF;
END_METHOD

(* Override Stop Method *)
METHOD PUBLIC OVERRIDE Stop : BOOL
IsActive := FALSE;
Position := 0.0;
FlowRate := 0.0;
Status := 'Valve Closed';
Stop := TRUE;
END_METHOD

(* Additional Method: GetFlowRate *)
METHOD PUBLIC GetFlowRate : REAL
GetFlowRate := FlowRate;
END_METHOD
END_FUNCTION_BLOCK

(* Interface for Polymorphism *)
INTERFACE IActuator
METHOD Start : BOOL
VAR_INPUT
    TargetPosition : REAL;
END_VAR
END_METHOD
METHOD Stop : BOOL
END_METHOD
METHOD GetPosition : REAL
END_METHOD
END_INTERFACE

(* Example Program Using Polymorphism *)
PROGRAM ActuatorControlDemo
VAR
    GenericActuator : Actuator;                     (* Instance of base class *)
    Valve : ValveController;                        (* Instance of derived class *)
    ActuatorRef : REFERENCE TO IActuator;           (* Reference for polymorphism *)
    CurrentPosition : REAL;                         (* To store position *)
    CurrentFlow : REAL;                             (* To store valve flow rate *)
    StartSuccess : BOOL;                            (* To track start result *)
END_VAR

(* Demo Logic *)
GenericActuator.Enable := TRUE;
StartSuccess := GenericActuator.Start(50.0);     (* Start generic actuator at 50% *)
CurrentPosition := GenericActuator.GetPosition();

Valve.Enable := TRUE;
StartSuccess := Valve.Start(75.0);               (* Start valve at 75% *)
CurrentPosition := Valve.GetPosition();
CurrentFlow := Valve.GetFlowRate();

(* Polymorphic Behavior *)
ActuatorRef := Valve;                            (* Reference valve as IActuator *)
StartSuccess := ActuatorRef.Start(25.0);         (* Calls ValveController.Start *)
CurrentPosition := ActuatorRef.GetPosition();    (* Calls ValveController.GetPosition *)
StartSuccess := ActuatorRef.Stop();              (* Calls ValveController.Stop *)

(* Note: ActuatorRef can also reference GenericActuator for generic behavior *)
END_PROGRAM
