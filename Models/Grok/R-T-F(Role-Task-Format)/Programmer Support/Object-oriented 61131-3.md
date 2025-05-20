(* Base class for generic motor control *)
CLASS Motor
    VAR
        Speed : REAL; (* Motor speed in RPM *)
        Running : BOOL; (* TRUE if motor is running *)
    END_VAR
    
    (* Virtual method to start the motor *)
    METHOD PUBLIC Start
        Running := TRUE;
        Speed := 1000.0; (* Default speed *)
    END_METHOD
    
    (* Virtual method to stop the motor *)
    METHOD PUBLIC Stop
        Running := FALSE;
        Speed := 0.0;
    END_METHOD
    
    (* Abstract method for status reporting *)
    METHOD PUBLIC VIRTUAL GetStatus : STRING
        GetStatus := 'Motor: Speed=' + REAL_TO_STRING(Speed) + ', Running=' + BOOL_TO_STRING(Running);
    END_METHOD
END_CLASS

(* Derived class for servo motor, extending Motor *)
CLASS ServoMotor EXTENDS Motor
    VAR
        Position : REAL; (* Servo position in degrees *)
    END_VAR
    
    (* Override Start method *)
    METHOD PUBLIC Start
        SUPER.Start(); (* Call base class Start *)
        Position := 0.0; (* Reset position *)
        Speed := 1500.0; (* Higher speed for servo *)
    END_METHOD
    
    (* Override GetStatus method *)
    METHOD PUBLIC GetStatus : STRING
        GetStatus := 'ServoMotor: Speed=' + REAL_TO_STRING(Speed) + 
                     ', Running=' + BOOL_TO_STRING(Running) + 
                     ', Position=' + REAL_TO_STRING(Position);
    END_METHOD
    
    (* Method specific to ServoMotor *)
    METHOD PUBLIC SetPosition
        VAR_INPUT
            NewPosition : REAL; (* Target position in degrees *)
        END_VAR
        IF Running THEN
            Position := LIMIT(NewPosition, 0.0, 360.0);
        END_IF;
    END_METHOD
END_CLASS

(* Main program demonstrating polymorphism *)
PROGRAM MotorControlDemo
VAR
    Motor1 : Motor; (* Base class instance *)
    Motor2 : ServoMotor; (* Derived class instance *)
    MotorRef : REF_TO Motor; (* Reference for polymorphism *)
    Status : STRING; (* Status output *)
    StartCmd : BOOL; (* Command to start motor *)
    StopCmd : BOOL; (* Command to stop motor *)
    ServoPosition : REAL := 90.0; (* Servo target position *)
END_VAR

(* Control logic *)
IF StartCmd THEN
    Motor1.Start();
    Motor2.Start();
    Motor2.SetPosition(ServoPosition); (* Servo-specific method *)
END_IF;

IF StopCmd THEN
    Motor1.Stop();
    Motor2.Stop();
END_IF;

(* Polymorphic behavior *)
MotorRef := REF(Motor1); (* Point to base motor *)
Status := MotorRef^.GetStatus(); (* Calls Motor.GetStatus *)

MotorRef := REF(Motor2); (* Point to servo motor *)
Status := MotorRef^.GetStatus(); (* Calls ServoMotor.GetStatus *)
END_PROGRAM
