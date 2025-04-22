```
PROGRAM ConveyorBeltControl
VAR
    ConveyorSpeed : REAL := 2.0; // Conveyor belt speed in m/s
    IsRunning : BOOL := FALSE; // Indicates if the conveyor is running
    IsStopped : BOOL := TRUE; // Indicates if the conveyor is stopped
    ObjectDetected : ARRAY [1..5] OF BOOL := (FALSE, FALSE, FALSE, FALSE, FALSE); // Array of sensor statuses
    ManualStopStation1 : BOOL := FALSE; // Manual stop command from station 1
    ManualStopStation2 : BOOL := FALSE; // Manual stop command from station 2
    ManualStopStation3 : BOOL := FALSE; // Manual stop command from station 3
    AutoMode : BOOL := TRUE; // Indicates if the system is in auto mode
    ManualMode : BOOL := FALSE; // Indicates if the system is in manual mode
    ConveyorMotor : BOOL := FALSE; // Indicates if the conveyor motor is powered
END_VAR

// Main control logic
IF AutoMode THEN
    // Automatic mode logic
    IF NOT IsStopped THEN
        // Conveyor is running
        FOR i := 1 TO 5 DO
            IF NOT ObjectDetected[i] THEN
                // If any sensor does not detect an object, stop the conveyor
                IsRunning := FALSE;
                IsStopped := TRUE;
                ConveyorMotor := FALSE;
                EXIT;
            END_IF;
        END_FOR;
        
        IF ManualStopStation1 OR ManualStopStation2 OR ManualStopStation3 THEN
            // If any station triggers a stop command, stop the conveyor
            IsRunning := FALSE;
            IsStopped := TRUE;
            ConveyorMotor := FALSE;
        END_IF;
        
        // If no stop conditions are met, keep the conveyor running
        IsRunning := TRUE;
        ConveyorMotor := TRUE;
    ELSE
        // Conveyor is stopped
        FOR i := 1 TO 5 DO
            IF ObjectDetected[i] THEN
                // If any sensor detects an object, start the conveyor
                IsRunning := TRUE;
                IsStopped := FALSE;
                ConveyorMotor := TRUE;
                EXIT;
            END_IF;
        END_FOR;
    END_IF;
ELSIF ManualMode THEN
    // Manual mode logic
    IF NOT ManualStopStation1 AND NOT ManualStopStation2 AND NOT ManualStopStation3 THEN
        // If no manual stop commands are active, start the conveyor
        IsRunning := TRUE;
        IsStopped := FALSE;
        ConveyorMotor := TRUE;
    ELSE
        // If any manual stop command is active, stop the conveyor
        IsRunning := FALSE;
        IsStopped := TRUE;
        ConveyorMotor := FALSE;
    END_IF;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Conveyor Running", IsRunning);
// Example: Write("Manual Stop Station 1", ManualStopStation1);
// Example: Write("Manual Stop Station 2", ManualStopStation2);
// Example: Write("Manual Stop Station 3", ManualStopStation3);
// Example: Write("Object Detected 1", ObjectDetected[1]);
// Example: Write("Object Detected 2", ObjectDetected[2]);
// Example: Write("Object Detected 3", ObjectDetected[3]);
// Example: Write("Object Detected 4", ObjectDetected[4]);
// Example: Write("Object Detected 5", ObjectDetected[5]);
// Example: Write("Auto Mode", AutoMode);
// Example: Write("Manual Mode", ManualMode);
// Example: Write("Conveyor Motor", ConveyorMotor);

END_PROGRAM
```
