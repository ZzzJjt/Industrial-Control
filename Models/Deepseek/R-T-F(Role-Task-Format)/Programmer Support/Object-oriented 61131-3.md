TYPE Device :
CLASS
VAR PUBLIC
    Name : STRING;
    Status : BOOL;
END_VAR

METHOD SetName(this : REFERENCE TO Device; newName : STRING) : BOOL
    this.Name := newName;
    RETURN TRUE;
END_METHOD

METHOD GetStatus(this : REFERENCE TO Device) : BOOL
    RETURN this.Status;
END_METHOD

METHOD StartDevice(this : REFERENCE TO Device) : BOOL
    this.Status := TRUE;
    RETURN TRUE;
END_METHOD

METHOD StopDevice(this : REFERENCE TO Device) : BOOL
    this.Status := FALSE;
    RETURN TRUE;
END_METHOD
END_CLASS
END_TYPE

TYPE Motor EXTENDS Device :
CLASS
VAR PUBLIC
    Speed : INT;
END_VAR

METHOD SetSpeed(this : REFERENCE TO Motor; newSpeed : INT) : BOOL
    this.Speed := newSpeed;
    RETURN TRUE;
END_METHOD

METHOD GetSpeed(this : REFERENCE TO Motor) : INT
    RETURN this.Speed;
END_METHOD

METHOD StartMotor(this : REFERENCE TO Motor) : BOOL
    IF this.StartDevice() THEN
        this.Speed := 1000; // Default speed
        RETURN TRUE;
    END_IF;
    RETURN FALSE;
END_METHOD

METHOD StopMotor(this : REFERENCE TO Motor) : BOOL
    IF this.StopDevice() THEN
        this.Speed := 0;
        RETURN TRUE;
    END_IF;
    RETURN FALSE;
END_METHOD
END_CLASS
END_TYPE

PROGRAM MainProgram
VAR
    dev : Device;
    motor : Motor;
    baseRef : REFERENCE TO Device;
    motorRef : REFERENCE TO Motor;
END_VAR

// Initialize references
baseRef := dev;
motorRef := motor;

// Using base class reference
baseRef.SetName('Generic Device');
baseRef.StartDevice();

// Using derived class reference
motorRef.SetName('Electric Motor');
motorRef.SetSpeed(1500);
motorRef.StartMotor();

// Polymorphic behavior
baseRef := motorRef;
baseRef.StartDevice(); // Calls Motor.StartMotor()

// Output results
IF baseRef.GetStatus() THEN
    // Handle device started
ELSE
    // Handle device stopped
END_IF;

IF motorRef.GetSpeed() > 0 THEN
    // Handle motor running
ELSE
    // Handle motor stopped
END_IF;
END_PROGRAM
