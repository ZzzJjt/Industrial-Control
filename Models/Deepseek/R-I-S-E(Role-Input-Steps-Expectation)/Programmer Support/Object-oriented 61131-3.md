CLASS FB_Actuator
VAR
    bEnable : BOOL := FALSE;
    sStatus : STRING := 'Idle';
END_VAR
METHOD Enable : BOOL
METHOD Disable : BOOL
END_CLASS

METHOD Enable : BOOL
    IF NOT bEnabled THEN
        // Perform enable logic
        bEnabled := TRUE;
        sStatus := 'Enabled';
        Enable := TRUE;
    ELSE
        Enable := FALSE;
    END_IF;
END_METHOD

CLASS FB_Valve EXTENDS FB_Actuator
VAR
    rPosition : REAL := 0.0; // 0% to 100%
END_VAR
METHOD SetPosition : BOOL
END_METHOD

CLASS FB_Actuator
VAR
    bEnabled : BOOL := FALSE;
    sStatus : STRING := 'Idle';
END_VAR

METHOD Enable : BOOL
    bEnabled := TRUE;
    sStatus := 'Enabled';
    Enable := TRUE;
END_METHOD

METHOD Disable : BOOL
    bEnabled := FALSE;
    sStatus := 'Disabled';
    Disable := TRUE;
END_METHOD
END_CLASS
