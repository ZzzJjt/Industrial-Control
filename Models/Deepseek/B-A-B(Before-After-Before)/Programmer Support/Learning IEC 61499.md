PROGRAM PLC_PRG
VAR
    TempSensor: REAL := 0.0;        // Input: current temperature
    SetPoint: REAL := 70.0;         // Configuration parameter
    Heater_ON: BOOL := FALSE;       // Output: heater status
END_VAR

// Main logic (runs cyclically)
IF TempSensor < SetPoint THEN
    Heater_ON := TRUE;
ELSE
    Heater_ON := FALSE;
END_IF;

FUNCTION_BLOCK TempSensor_FB
VAR_IN_OUT
END_VAR
VAR_OUTPUT
    temperature: REAL := 0.0;
END_VAR
EVENT_OUTPUT
    DataReady: EVENT;
END_FUNCTION_BLOCK

FUNCTION_BLOCK Heater_FB
VAR_INPUT
    enable: BOOL := FALSE;
END_VAR
EVENT_INPUT
    Command: EVENT;
END_FUNCTION_BLOCK

