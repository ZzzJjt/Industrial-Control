FUNCTION_BLOCK FB_Actuator
VAR
    IsOn : BOOL := FALSE;
END_VAR

METHOD PUBLIC SwitchOn
    IsOn := TRUE;
END_METHOD

METHOD PUBLIC SwitchOff
    IsOn := FALSE;
END_METHOD

FUNCTION_BLOCK FB_Valve EXTENDS FB_Actuator
VAR
    Opening : REAL := 0.0; // 0–100% valve opening
END_VAR

METHOD PUBLIC SetOpening
VAR_INPUT
    Value : REAL;
END_VAR
    IF Value >= 0.0 AND Value <= 100.0 THEN
        Opening := Value;
    END_IF
END_METHOD
