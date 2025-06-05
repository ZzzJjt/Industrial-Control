VAR
    Temp : REAL := 85.0;
    Overheat : BOOL;
END_VAR

IF Temp > 100.0 THEN
    Overheat := TRUE;
ELSE
    Overheat := FALSE;
END_IF;

VAR
    i : INT;
    Sum : INT := 0;
END_VAR

FOR i := 1 TO 10 DO
    Sum := Sum + i;
END_FOR;

VAR
    Start : BOOL;
    T1 : TON;
END_VAR

T1(IN := Start, PT := T#5s);
IF T1.Q THEN
    // Action after 5 seconds ON
END_IF;

VAR
    Mode : INT;
END_VAR

CASE Mode OF
    0: // Standby
    1: // Run
    2: // Error
END_CASE;

VAR
    Values : ARRAY[1..5] OF INT := [10, 20, 30, 40, 50];
    Total : INT := 0;
    i : INT;
END_VAR

FOR i := 1 TO 5 DO
    Total := Total + Values[i];
END_FOR;
